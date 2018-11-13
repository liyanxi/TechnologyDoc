>大家都知道开发中应用程序的性能是非常重要的，但是这也是优化提升的难点，本章针对 Android性能实践——从减少APK的大小开始，提升用户的体验。
>
>原文地址 https://developer.android.com/topic/performance/reduce-apk-size

用户经常会避免下载看起来太大的应用程序，尤其是在设备连接到2G和3G网络或付费网络的应用市场内部。这篇文章讲述如何减少您的应用程序APK的大小，以便使更多的用户下载您的应用程序。

## 理解APK文件结构

在讨论如何降低您的应用程序的大小之前，理解一个应用程序的APK的文件结构它是非常有帮助的。一个APK文件包含一个ZIP归档，主要包含您的应用程序的所有文件。这些文件包括Java类文件，资源文件，和包含编译的文件资源。

一个APK包含如下目录：

* `META_INF/`：包含`CERT.SF`和`CERT.RSK` 签名文件，以及`MANIFEST.MF`清单文件。
* `assets/`：包含应用程序的资产,应用程序可以使用一个`AssetManager`对象检索。
* `res/`：包含不能被编译进`resources.arsc`文件的资源。
* `lib/`：包含编译后的代码,是特定于处理器的软件层。这个目录包含为每个平台类型分配的子目录，像`armeabi`,`armeabi-v7a`,`arm64-v8a`,`x86`,`x86_64`,和`mips`。

APK还包含以下文件。其中,只有`AndroidManifest.xml`是强制性的。

* `resources.arsc`：包含编译的资源。这个文件包含来自文件夹`res/values/`下所有配置的XML内容。包装工具提取这个XML内容，将它编译成二进制形式并且归档这些内容。这个内容包括语言字符串和样式以及路径下不存在于`resources.arsc`文件的内容，比如布局文件和图片。
* `classes.dex`：包含DEX文件中编译的类可以被 Dalvik/ART 虚拟机理解的格式。
* `AndroidManifest.xml`：包含Android的核心清单文件。这个文件列出了应用的名称,版本,访问权限,以及引用的库文件的应用程序。文件使用Android的二进制XML格式。

## 减少资源的数量和大小

APK的大小影响应用程序的加载速度，对内存的使用，和它对电量的消耗。一个使您的APK较小的简单方式是减少它包含资源的数量和大小。特别是，您可以移除应用程序不再使用的资源，和您可以使用可拉伸的[Drawable](https://developer.android.com/reference/android/graphics/drawable/Drawable.html)图像文件。本节讨论这些方法以及其他几个方法都是通过减少您的应用程序使用的资源来减少您的APK总体规模。

### 移除不使用的资源

Android Studio 内部包含一个静态代码分析器`lint`工具，使用它可以检索资源文件夹`res/`，找出您的代码不引用的资源。当`lint`工具在您的工程中发现一个潜在的未被使用的资源，它会打印一条消息就像下面的例子。

```
res/layout/preferences.xml: Warning: The resource R.layout.preferences appears
    to be unused [UnusedResources]
```

*⚠️：`lint`工具不能扫描`assets/`目录，通过反射引用assets,或您连接库文件到您的应用。同时,它不会删除资源;只提醒你他们的存在*

您的代码依赖的库文件或许包含不被使用的资源。如果在您的`build.gradle`构建文件中启用[shrinkResources](https://developer.android.com/studio/build/shrink-code.html),它表示Gradle会自动删除这些资源。

```
android {
    // Other settings

    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

为了使用`shrinkResources`,你必须启用代码压缩。在构建过程中，首先[ProGuard](https://developer.android.com/studio/build/shrink-code.html)删除未被使用的代码但是闲置未被使用的资源。然后Gradle移除未被使用的资源。

关于ProGuard和其它Android Studio提供的途径，帮助您减少APK大小的更多的信息，可以参照[Shrink Your Code and Resources](https://developer.android.com/studio/build/shrink-code.html)。

在Android Gradle插件0.7和更高版本上，您可以声明您的应用程序支持的配置。Gradle通过使用`resConfig`和`resConfigs`类别以及默认配置`defaultConfig`选项，去构建系统。然后构建系统可以防止不受支持的配置的资源,出现在APK文件中,从而减少APK文件的大小。关于此功能的更多信息，可以查看[ Remove unused alternative resources](https://developer.android.com/studio/build/shrink-code.html#unused-alt-resources)

### 最小化来自于外部库资源的使用

当我们开发一个Android应用程序时，您通常使用外部库来提高应用程序的可用性和多功能性。例如，您可以引用[Android Support Library](https://developer.android.com/topic/libraries/support-library/index.html)来改善应用程序在旧设备上的用户体验，或者您可以在您的应用程序内部使用[Google Play Services](https://developers.google.com/android/guides/overview)来检索文本，对其进行自动翻译。

如果一个库被设计用于一个服务器或桌面。它可能包括许多您的应用程序不需要的对象和方法。为了做到只包括您需要的部分，如果库的证书允许您可以编辑该库的文件。您还可以使用一个替代的方法，移动指定的功能模块到您的应用程序。

*⚠️：ProGuard可以清除一些不必要的代码导入库，但是不能移除一个库庞大的内部依赖关系。*

### 仅仅支持特定密度

Android支持一组非常大的设备，包括各种各样的屏幕密度。在Android 4.4（API级别19）和更高版本，该框架支持各种密度:`ldpi`，`mdpi`，`tvdpi`，`hdpi`，`xhdpi`，`xxhdpi`，`xxxhdpi`。即使Android支持所有的这些密度，您也不需要为每个密度导出相应的资源。

如果您知道只有一小部分用户的设备拥有特定的密度,这些密度是否需要包括到您的应用程序中那是值得考虑的。对应于一个特定的屏幕密度，如果您不包括相应的资源,Android会根据其它屏幕密度，找寻与之相近的现有资源。

如果您的应用程序只需要按比例缩小的图像，在`drawable-nodpi/`文件下放置唯一图像即可，这样您可以节省更多的空间。我们建议每一个应用程序包括至少一个`xxhdpi`图像变体。

关于屏幕密度的更多信息，可以参见[Screen Sizes and Densities](https://developer.android.com/about/dashboards/index.html#Screens)

### 使用drawable对象

一些图像不要求一个静态的图片资源；框架可以在运行时动态画出图像。[Drawable](https://developer.android.com/reference/android/graphics/drawable/Drawable.html)对象（`<shape>`在XML内）在您的APK内部可以占用少量的空间。另外，XML`Drawable`对象产生的单色图像符合材料设计（material design）的指导方针。

### 减少资源
 
对于一个图像的变化，您可以包括一个单独的资源,如着色、阴影或旋转版本相同的图像。不管怎样,我们建议您重用相同的一组资源,在运行时根据需要自定义它们。

Android 提供了一些工具来改变资源的颜色，在Android5.0（API21）和更高的版本上可以使用`android:tint和tintMode`属性。对于较低的平台版本，可以使用[ColorFilter](https://developer.android.com/reference/android/graphics/ColorFilter.html)类。

对于一种资源经过旋转可以得到另一种资源，您可以只导入一个。下面的代码片段提供了一个示例：将一个“thumb up”绕轴旋转180度转变成“thumb down”：

```
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/ic_thumb_up"
    android:pivotX="50%"
    android:pivotY="50%"
    android:fromDegrees="180" />
```

### 图像通过代码渲染

您还可以通过程序渲染您的图片，进而减少您的APK大小。程序渲染腾出空间，因为您不需要存储图片到你的APK文件中。

### Crunch PNG 文件

在构建过程中,`aapt`工具可以使用无损压缩方式优化放置在`res/drawable/`目录下的图像资源，进而降低图片空间占用。例如，the aapt tool can convert a true-color PNG that does not require more than 256 colors to an 8-bit PNG with a color palette（这句不好翻译）.这样转换之后，相同质量的图像会占用更小的内存。

请注意，`aapt`拥有以下限制：

* `aapt`不能压缩`asset/`目录下的PNG文件。
* `aapt`工具优化图像文件需要使用256或更少的颜色。
* `aapt`工具可能多次优化已经压缩过的PNG文件。为了避免这个您可以在Gradle文件中使用`cruncherEnabled`标志禁用对PNG文件的处理。

```
aaptOptions {
    cruncherEnabled = false
}
```
 
### 压缩 PNG 和 JPEG 文件

您可以在不丢失图像质量的情况下使用工具减少 PNG 文件的大小，像 [pngcrush](http://pmt.sourceforge.net/pngcrush/), [pngquant](https://pngquant.org/), or [zopflipng](https://github.com/google/zopfli)。所有这些工具可以减少PNG文件的大小,同时保留感知的图像质量。

`pngcrush`工具是特别有效的：这个工具遍历PNG过滤器和zlib(Deflate)参数,使用过滤器和参数的组合压缩图像。然后选择压缩效果最好的配置输出（*It then chooses the configuration that yields the smallest compressed output*）。

为了压缩 JPEG 文件，你可以使用工具比如： [packJPG](http://www.elektronik.htw-aalen.de/packjpg/) and [guetzli](https://github.com/google/guetzli).

### 使用 WebP 格式文件

针对Android 3.2（API级别13）和更高版本，您还可以使用WebP图像格式文件，而不是使用 PNG 和 JPEG 文件。WebP格式提供了有损压缩(如JPEG)以及透明度(如PNG)，相较于JPEG或PNG它可以提供更好的压缩。

使用Android Studio，您可以转换存在的 BMP，JPG，PNG 或 静态的GIF图像为 WebP格式。更多的信息，请参考 [Create WebP Images Using Android Studio](https://developer.android.com/studio/write/convert-webp.html)。

*⚠️：谷歌市场接受APK的启动图标只能是PNG格式。*

### 使用矢量图

您可以使用矢量图形创建分辨率无关图标和其他可伸缩的媒体。使用这些图形可以大大减少你的APK大小。在Android中，矢量图表示为VectorDrawable对象。[VectorDrawable](https://developer.android.com/reference/android/graphics/drawable/VectorDrawable.html)对象,一个100字节的文件可以用于呈现屏幕大小的图像。

然而，系统需要大量的时间来呈现每个**VectorDrawable**对象，且图像越大呈现到屏幕上所需要的时间越长。因此，考虑只有当显示小图像时使用这些矢量图形。

更多的关于**VectorDrawable**对象如何工作的信息，可以参考[Working with Drawables](https://developer.android.com/training/material/drawables.html)。

### 使用矢量图形动画图像

不能使用 [AnimationDrawable](https://developer.android.com/reference/android/graphics/drawable/AnimationDrawable.html)对象去创建帧动画，因为这样做对于每一帧动画都需要包括一个单独的**bitmap**文件,这大大增加APK的大小。

您应该使用[AnimatedVectorDrawableCompat](https://developer.android.com/reference/android/support/graphics/drawable/AnimatedVectorDrawableCompat.html)去创建[矢量动画](https://developer.android.com/training/material/animations.html#AnimVector)。

## 减少 native 和 Java 代码

这里有几种方法可以用来减少在您的应用程序中 Java 和 native 代码库的大小。

### 移除不必要生成的代码

确定了解由工具自动生成的任何代码的足迹（*Make sure to understand the footprint of any code which is automatically generated*）。例如，许多协议缓冲工具生成过多的方法和类,它可以使您的应用程序大小提升两倍或三倍。

### 避免使用枚举

一个枚举可以使您的应用程序的`classes.dex`文件增加约 1.0 到 1.4 KB大小。对于复杂系统或共享库这些添加可以迅速增大此文件。如果可能的话，可以考虑使用`@IntDef`注解和[ProGuard](https://developer.android.com/studio/build/shrink-code.html)去剥离枚举，取出将它们转换为整数。这种类型的转换保存所有的枚举类型是安全可靠的。

### 减少本地二进制文件的大小

如果您的应用程序使用本地代码和Android NDK,你也可以减少你的应用程序的发布版本的大小，以优化你的代码。两个有用的技术去除调试符号而不是提取本地库。

#### 移除调试符号

使用调试符号的意义，在于您的应用程序在开发过程中仍然需要调试。使用`arm-eabi-strip`工具（Android NDK 提供），从本地库删除不必要的调试符号。
之后,您可以编译发布构建。

#### 避免提取本地库

在构建应用程序的发布版本时,如果您在你的应用程序的清单文件中，元素标签[<application>](https://developer.android.com/guide/topics/manifest/application-element.html)下设置属性如下`android:extractNativeLibs="false"`,则APK包将不会压缩`.so`文件。禁用这个标志可以在应用的安装过程中，阻止[PackageManager](https://developer.android.com/reference/android/content/pm/PackageManager.html)从您的APK拷贝`.so`文件到文件系统，这样做的好处是让您更新您的应用程序更小。

## 维护多个精简的 APKS

APK可能包含一些用户下载但从未使用的内容,如区域或语言信息。为用户创建一个最小的下载,你可以将您的应用程序分割成几个apk,分化的因素如屏幕大小或GPU硬件的支持。

当用户下载您的应用程序,他们的设备基于设备的功能和设置接收到正确的APK。这种方式,设备没有接收到资产设备没有的功能。例如,如果用户拥有一个`hdpi`设备,他们不需要相对于更高密度的设备包括的`xxxhdpi`资源。

更多相关信息，请参考 [Configure APK Splits](https://developer.android.com/studio/build/configure-apk-splits.html) 和 [Maintaining Multiple APKs](https://developer.android.com/training/multiple-apks/index.html)。