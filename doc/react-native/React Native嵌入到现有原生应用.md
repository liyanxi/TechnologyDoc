## React Native嵌入到现有原生应用(Android)

>如果你正准备从头开始制作一个新的应用，那么React Native会是个非常好的选择。但如果你只想给现有的原生应用中添加一两个视图或是业务流程，React Native也同样不在话下。只需简单几步，你就可以给原有应用加上新的基于React Native的特性、画面和视图等。

### 1.核心概念
这里把React Native组件植入到Android应用中有如下几个主要步骤：

1. 首先要了解你要植入的React Native组件。
2. 在Android项目根目录中使用npm来安装`react-native` ，这样同时会创建一个`node_modules/`的目录。
3. 创建js文件，编写React Native组件的js代码。
4. 在`build.gradle`文件中添加`com.facebook.react:react-native:+`，以及一个指向`node_nodules/`目录中的`react-native`预编译库的`maven`路径。
5. 创建一个React Native专属的`Activity`(可混合使用)，在其中再创建`ReactRootView`。
6. 生成对应`bundle`文件，启动React Native的Packager服务，运行应用。
7. 接下来可自由发挥，添加更多的React Native组件。
8. 在真机上运行、调试（这里需要先更新`bundle`文件）。
9. 打包。

### 2.开发环境准备
此处可参考我写的另一篇文章[React Native环境搭建][0]，(已有环境的，此处可选择跳过).

### 3.添加JS代码管理
* 在项目的根目录运行以下命令：

```
$ npm init
$ npm install --save react react-native
$ curl -o .flowconfig https://raw.githubusercontent.com/facebook/react-native/master/.flowconfig
```
此时可在Android项目根目录下看到如下文件/文件夹：`node_modules/`、`package.json`

* 打开`package.json`文件，在`scripts`属性下添加如下字段：

`"start": "node node_modules/react-native/local-cli/cli.js start"`

* 在根目录下新建js文件夹(存放React Native相关组件)，index.android.js(RN启动文件)。
主要启动代码如下：

```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 * @flow
 */
import {AppRegistry} from 'react-native';
import AwesomeProject from './js/AwesomeProject'
//注册（只要一遍）
AppRegistry.registerComponent('AwesomeProject', () => AwesomeProject);

```

### 4.添加App开发准备操作
* 在你app的`build.gradle`文件内添加`React Native`相关依赖：

`compile 'com.facebook.react:react-native:+' // From node_modules.`

* 在你工程`build.gradle`文件内添加本地`React Native`maven库：

```
allprojects {
    repositories {
        jcenter()
        maven {
            // All of React Native (JS, Android binaries) is installed from npm
            url "$rootDir/node_modules/react-native/android"
        }
    }
}
```
<font color='red'>注意事项:这里的url路径要写对，否则将出现如下错误：
“Failed to resolve: com.facebook.react:react-native:0.x.x" errors after running Gradle sync in Android Studio.
</font> 

* `AndroidManifest.xml`清单文件配置
 - 如果你需要访问React Native的DevSettingsActivity，则需要注册
`<activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />`

### 5.准备工作搞定：接下来添加原生代码

* 纯React Native页面主要代码（底部有完整代码地址）

```
package com.itingchunyu.reactnative.view;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.appcompat.BuildConfig;
import android.view.KeyEvent;

import com.facebook.react.ReactInstanceManager;
import com.facebook.react.ReactRootView;
import com.facebook.react.common.LifecycleState;
import com.facebook.react.modules.core.DefaultHardwareBackBtnHandler;
import com.facebook.react.shell.MainReactPackage;

/**
 * Created by liyanxi on 2016/12/13.
 * Copyright (c) itingchunyu@163.com. All rights reserved.
 */

public class MyReactActivity extends AppCompatActivity implements DefaultHardwareBackBtnHandler {

    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mReactRootView = new ReactRootView(this);
        mReactInstanceManager = ReactInstanceManager.builder()
                .setApplication(getApplication())
                .setBundleAssetName("index.android.bundle") //可远程地址
                .setJSMainModuleName("index.android")//根目录下index.android.js文件
                .addPackage(new MainReactPackage())//如果为true，则会启用诸如JS重新加载和调试之类的开发人员选项.反之打包
                .setUseDeveloperSupport(true)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();
                //'AwesomeProject'==>index.android.js 页面内注册名称，可根据自己随意调整
        mReactRootView.startReactApplication(mReactInstanceManager, "AwesomeProject", null);
        setContentView(mReactRootView);
    }

    @Override
    protected void onPause() {
        super.onPause();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostPause(this);
        }
    }

    @Override
    protected void onResume() {
        super.onResume();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostResume(this, this);
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostDestroy(this);
        }
    }

    @Override
    public boolean onKeyUp(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_MENU && mReactInstanceManager != null) {
            mReactInstanceManager.showDevOptionsDialog();
            return true;
        }
        return super.onKeyUp(keyCode, event);
    }

    @Override
    public void invokeDefaultOnBackPressed() {
        super.onBackPressed();
    }
}
```

* 原生和RN混合页面主要代码（底部有完整代码地址）：

```
package com.itingchunyu.reactnative.view;

import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.appcompat.BuildConfig;
import android.widget.LinearLayout;

import com.facebook.react.ReactInstanceManager;
import com.facebook.react.ReactRootView;
import com.facebook.react.common.LifecycleState;
import com.facebook.react.shell.MainReactPackage;
import com.itingchunyu.reactnative.R;

/**
 * Created by liyanxi on 2016/12/13.
 * Copyright (c) itingchunyu@163.com. All rights reserved.
 */

public class HybridActivity extends AppCompatActivity {

    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;

    private LinearLayout layoutReactContainer;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_second);

        mReactRootView = new ReactRootView(this);
        mReactInstanceManager = ReactInstanceManager.builder()
                .setApplication(getApplication())
                .setBundleAssetName("index.android.bundle")
                .setJSMainModuleName("index.android")
                .addPackage(new MainReactPackage())
                //如果为true，则会启用诸如JS重新加载和调试之类的开发人员选项.反之打包
                .setUseDeveloperSupport(true)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();
        mReactRootView.startReactApplication(mReactInstanceManager, "AwesomeProject", null);

        layoutReactContainer= (LinearLayout) findViewById(R.id.layout_react);

        layoutReactContainer.addView(mReactRootView);
    }
}
```

### 6.配置权限保证开发过程中红屏错误能正确展示

>如果您的应用程式指定的Android API等级为23或更高，请确定您已为开发版本启用了重叠权限。您可以使用“设置”进行检查。 canDrawOverlays（this）;.这在开发版本中是必需的，因为反应本地开发错误必须显示在所有其他窗口之上。由于在API级别23中引入了新的权限系统，用户需要批准它。这可以通过将下面的代码添加到onCreate（）方法中的Activity文件来实现。 OVERLAY_PERMISSION_REQ_CODE是类的一个字段，它负责将结果传递回Activity。

```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    if (!Settings.canDrawOverlays(this)) {
        Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
                                   Uri.parse("package:" + getPackageName()));
        startActivityForResult(intent, OVERLAY_PERMISSION_REQ_CODE);
    }
}
```
最后，必须覆盖onActivityResult() 方法（如下面的代码所示）来处理一致性UX的权限接受或拒绝的情况。

```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == OVERLAY_PERMISSION_REQ_CODE) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (!Settings.canDrawOverlays(this)) {
                // SYSTEM_ALERT_WINDOW permission not granted...
            }
        }
    }
}
```

### 7.应用飞起来

* 启动服务器`React packager`

>$npm start(配合package.json文件`start`)

或者

>$react-native start

![启动服务](http://img.blog.csdn.net/20170110192315891?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2luODE2NzIzNDU5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
  
* 启动App应用(看效果)

![app效果](http://img.blog.csdn.net/20170110193117207?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2luODE2NzIzNDU5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 调试过程踩过的坑

 启动ReactNative相关Activity页面崩溃
![app崩溃](http://img.blog.csdn.net/20170110192017765?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2luODE2NzIzNDU5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 解决方案看代码截图：
 ![崩溃解决](http://img.blog.csdn.net/20170110192201390?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2luODE2NzIzNDU5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 <font color='red'> 此处红框内需要注意开发调试过程需要设置为true动态加载，false涉及到静态bundle文件，下面打包会单独讲。</font>

### 8.在Android Studio中打包apk
>你也可以使用Android Studio来打包！You can use Android Studio to create your release builds too! It’s as easy as creating release builds of your previously-existing native Android app. There’s just one additional step, which you’ll have to do before every release build. You need to execute the following to create a React Native bundle, which’ll be included with your native Android app:

```
react-native bundle --platform android --dev false --entry-file ./index.android.js --bundle-output ./app/src/main/assets/index.android.bundle --assets-dest ./app/src/main/res/
```
<font color='red'>上面主要意思是说在打包前不可避免的操作如下:</font>
> 1.在根目录下执行上述命令，生成`React Native Activity页面bundle文件`，此处注意路径不要写错（这里是在` ./app/src/main/assets/`目录下生成对应`index.android.bundle`文件）
> 
> 2.远程bundle文件（用于动态更新，这里暂不详解）

#### 9.到此，相信你的应用应该跑起来了。期间如果遇到问题可给我留言，或发邮件（itingchunyu@163.com）咨询我，感谢你的浏览；

### 10.完整代码地址
[ReactNativeDemo][1]

---------------------------

[0]: http://blog.csdn.net/win816723459/article/details/53395038
[1]: https://github.com/liyanxi/ReactNativeDemo
