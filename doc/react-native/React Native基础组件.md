### React Native基础组件学习

----------------

### 本节主要介绍以下组件

- View 容器组件（同[Html][0]中的`div`标签）
- Text 文本组件（同[Html][0]中的`p`标签）
- Image 图片组件
- TextInput 输入组件
- ScrollView 滑动容器组件（同原生组件`ScrollView`滑动方向水平和垂直） 

### 1.View 容器组件
 - 定义
 
	`<View>`React Native组件，相当于html中的`<div>`标签，具有相同的特性，支持嵌套。`<View>`组件可以把固定的区间分割成独立的、不同的部分。用于严格的模块化布局组织工具，默认内部`child`垂直排列,可调整排列方向，这点与Android原生组件`LinearLayout`很像。
	
 - 主要属性	（详细请参考[View][1]）
   - pointerEvents enum('box-none', 'none', 'box-only', 'auto')
   
 ```
 用于控制当前视图是否可以作为触控事件的目标。
* auto: 视图可以作为触控事件的目标
* none: 视图不能作为触控事件的目标
* box-none: 视图自身不能作为触控事件的目标，但其子视图可以。
* box-only：视图自身可以作为触控事件的目标，但其子视图不能。
 ```
	- 样式
	
	```
	Flexbox
	
	backfaceVisibility enum('visible', 'hidden')
	
	backgroundColor string
	
	borderColor string
	
	borderTopColor string
	
	borderRightColor string
	
	borderBottomColor string
	
	borderLeftColor string
	
	borderRadius number
	
	borderTopLeftRadius number
	
	borderTopRightRadius number
	
	borderBottomLeftRadius number
	
	borderBottomRightRadius number
	
	borderStyle enum('solid', 'dotted', 'dashed')
	
	borderWidth number
	
	borderTopWidth number
	
	borderRightWidth number
	
	borderBottomWidth number
	
	borderLeftWidth number
	
	opacity number
	
	overflow enum('visible', 'hidden')
	
	android elevation number
	(限Android)使用Android原生的 elevation API来设置视图的高度（elevation）。这样可以为视图添加一个投影，并且会影响视图层叠的顺序。此属性仅支持Android5.0及以上版本。
	```
	
	
 - 用法
	
	`View`用法与`LinearLayout`一致，唯一的区别在于自身可定制的属性不同,下面结合代码介绍，见注释：
	
```
/**
 * Created by liyanxi on 2016/11/28.
 */
import React, { Component } from 'react';
import {
    StyleSheet,
    Text,
    View
} from 'react-native';
export default class WelcomDemo extends Component {
    render() {
        return (
        	  //JSX语法
            <View style={styles.container}>
                <Text style={styles.welcome}>
                    Welcome to React Native!
                </Text>
                <Text style={styles.instructions}>
                    To get started, edit index.android.js
                </Text>
                <Text style={styles.instructions}>
                    Double tap R on your keyboard to reload,{'\n'}
                    Shake or press menu button for dev menu
                </Text>
            </View>
        );
    }
}
//样式
const styles = StyleSheet.create({
    container: {	//默认垂直排列 flexDirection: 'column';
        flex: 1,
        justifyContent: 'center',	//内部child 主轴方向居中
        alignItems: 'center',		//内部child 次轴（垂直主轴方向：此处为列）方向居中
        backgroundColor: '#F5FCFF',//背景颜色
    },
    welcome: {
        fontSize: 20,
        textAlign: 'center',
        margin: 10,
    },
    instructions: {
        textAlign: 'center',
        color: '#333333',
        marginBottom: 5,
    },
});
```
注：完整代码见底部

### 2.Text 文本组件
> 一个用于显示文本的React组件，并且它也支持嵌套、样式，以及触摸处理。

* 自身属性
 - `numberOfLines ` number 用来当文本过长的时候裁剪文本。包括折叠产生的换行在内，总的行数不会超过这个属性的限制。
 - `onLongPress` function 当文本长按后调用此方法
 -  `onPress` function 当文本点击后调用此方法
 -  `selectable` function 决定用户是否可以长按选择文本，以便复制和粘贴。
 
* 样式style
> [View#style...][1]

	```
	color string
	
	fontFamily string
	
	fontSize number
	
	fontStyle enum('normal', 'italic')
	
	fontWeight enum("normal", 'bold', '100', '200', '300', '400', '500', '600', '700', '800', '900')
	
	指定字体的粗细。大多数字体都支持'normal'和'bold'值。并非所有字体都支持所有的数字值。如果某个值不支持，则会自动选择最接近的值。
	
	letterSpacing number
	
	lineHeight number
	
	textAlign enum("auto", 'left', 'right', 'center', 'justify')
	
	指定文本的对齐方式。其中'justify'值仅iOS支持。 
	```
* <font color='red'>注意事项</font>

 1. 嵌套文本（扁平格式，可继承父类的样式: <font color='red'>仅限于文本标签的子树</font>）
 2. 作为容器（导致子文本组件`Text`元素在布局上不同于其它组件：在Text内部的元素不再使用flexbox布局，而是采用文本布局。这意味着<Text>内部的元素不再是一个个矩形，而可能会在行末进行折叠。）
 
 ```
 <Text>
  <Text>First part and </Text>
  <Text>second part</Text>
</Text>
// Text container: all the text flows as if it was one
// |First part |
// |and second |
// |part       |
<View>
  <Text>First part and </Text>
  <Text>second part</Text>
</View>
// View container: each text is its own block
// |First part |
// |and        |
// |second part|
 ```
 
### 3.Image 图片组件
>一个用于显示多种不同类型图片的React组件，包括网络图片、静态资源、临时的本地图片、以及本地磁盘上的图片（如相册）等。详细用法参阅[图片文档][2]。

```
render() {
  return (
    <View>
      <Image
        style={styles.icon}
        source={require('./icon.png')}
      />
      <Image
        style={styles.logo}
        source={{uri: 'http://facebook.github.io/react/img/logo_og.png'}}
      />
    </View>
  );
}
```
 - 主要属性
 
  * resizeMode enum('cover','contain','stretch'); 决定当组件尺寸和图片尺寸不成比例的时候如何调整图片的大小
  
	<font color='red'>cover</font>: 在保持图片宽高比的前提下缩放图片，直到宽度和高度都大于等于容器视图的尺寸（如果容器有padding内衬的话，则相应减去）。译注：这样图片完全覆盖甚至超出容器，容器中不留任何空白。
	
	<font color='red'>contain</font>: 在保持图片宽高比的前提下缩放图片，直到宽度和高度都小于等于容器视图的尺寸（如果容器有padding内衬的话，则相应减去）。译注：这样图片完全被包裹在容器中，容器中可能留有空白
	
	<font color='red'>stretch</font>: 拉伸图片且不维持宽高比，直到宽高都刚好填满容器。
	
	* source {uri: string}, number 

	uri是一个表示图片的资源标识的字符串，它可以是一个http地址或是一个本地文件路径（使用require(相对路径)来引用）。	
   	 
 - 样式此处不作详解，大同小异（同flexbox、shadow、transforms） 

### 4. TextInput 输入组件

>TextInput是一个允许用户在应用中通过键盘输入文本的基本组件。本组件的属性提供了多种特性的配置，譬如自动完成、自动大小写、占位文字，以及多种不同的键盘类型（如纯数字键盘）等等。

最简单的用法就是丢一个<font color='red'>TextInput</font>到应用里，然后订阅它的<font color='red'>onChangeText</font>事件来读取用户的输入。它还有一些其它的事件，譬如<font color='red'>onSubmitEditing</font>和<font color='red'>onFocus</font>。一个简单官网的例子如下：

```
import React, { Component } from 'react';
import { AppRegistry, TextInput } from 'react-native';

class UselessTextInput extends Component {
  constructor(props) {
    super(props);
    this.state = { text: 'Useless Placeholder' };
  }
  render() {
    return (
      <TextInput
        style={{height: 40, borderColor: 'gray', borderWidth: 1}}
        onChangeText={(text) => this.setState({text})}
        value={this.state.text}
      />
    );
  }
}
// App registration and rendering
AppRegistry.registerComponent('AwesomeProject', () => UselessTextInput);
```

<font color='red'>注意1:</font> 有些属性仅在`multiline`为true或者为false的时候有效。此外，当`multiline=false`时，为元素的某一个边添加边框样式（例如：`borderBottomColor`，`borderLeftWidth`等）将不会生效。为了能够实现效果你可以使用一个`View`来包裹`TextInput`.

<font color='red'>注意2:</font>`TextInput`有一个默认的底部边框，不可以改变。避免这种情况的解决方案是要么不显式地设置高度,这种情况系统将照顾边界显示在正确的位置,或则设置底部border边框颜色透明即`underlineColorAndroid `

另一方面：可以通过设置app内`Activity`的`windowSoftInputMode`属性（`adjustResize`等）调整软键盘的位置以及界面的展示具体参考[Activity属性设置][3];

* 属性列表介绍

| 属性名 | 类型 | 可选值（解释） | 注意 |
| :-------------: | ----- | :------------- | :-----: |
| autoCapitalize  | enum | characters: 所有的字符。words: 每个单词的第一个字符。sentences: 每句话的第一个字符（默认）。none: 不自动切换任何字符为大写。 | - |
| autoCorrect  | bool | 拼写自动修正（默认true,关闭拼写自动修正） | - |
| autoFocus  | bool | 如果为true,在componentDidMount后会获得焦点。默认值fase | - |
| defaultValue   | string | 文本框中初始值 | 保持属性和状态同步的时候可使用defaultValue来代替 |
| editable   | bool | false:文本框不可编辑，默认为true | 依此可切换可编辑状态 |
| keyboardType  | enum  | 决定弹出何种软键盘，兼容所有平台（default,numeric,email-address） | - |
| maxLength   | number | 限制文本框可输入字符数 | 配合keyboradType属性使用 |
| multiline   | bool | true：可输入多行文本，默认为fales | 设置true时，与软件盘换行冲突 |
| onBlur   | function | 文本框失去焦点时监听 | 用于格式校验 |
| onChange   | function | 文本变化监听 | - |
| onChangeText    | function | 同onChange，改变后的文本作为参数传入 | - |
| onEndEditing    | function | 文本输入结束监听 | - |
| onFocus    | function | 文本框获取焦点时监听 | - |
| onLayout    | function | 布局改变或重新绘制时调用，参数为{x, y, width, height}。 | - |
| onSubmitEditing  | function   | 此回调函数当软键盘的确定/提交按钮被按下的时候调用此函数。 | 如果multiline={true}，此属性不可用。 |
| placeholder    | string | 没有输入文本输入时，展示文本 | - |
| placeholderTextColor   | string   | 对应placeholder展示文本颜色 | - |
| secureTextEntry    | bool  | 如果为true，文本框会遮住之前输入的文字，这样类似密码之类的敏感文字可以更加安全。默认值为false。 | - |
| selectTextOnFocus   | bool   | 如果为true，当获得焦点的时候，所有的文字都会被选中。 | - |
| selectionColor     | string | 设置输入框高亮时的颜色（在iOS上还包括光标） | - |
| value      | string | 文本框中的文字内容。 | TextInput是一个受约束的(Controlled)的组件，意味着如果提供了value属性，原生值会被强制与value属性保持一致。 | 


### 5. ScrollView 组件
>记住ScrollView必须有一个确定的高度才能正常工作，因为它实际上所做的就是将一系列不确定高度的子组件装进一个确定高度的容器（通过滚动操作）。要给一个ScrollView确定一个高度的话，要么直接给它设置高度（不建议），要么<font color='red'>确定所有的父容器的高度都是有界的。在视图栈的任意一个位置忘记使用{flex:1}都会导致错误</font>，你可以使用元素查看器来查找问题的原因。ScrollView内部的其他响应者尚无法阻止ScrollView本身成为响应者。

* 主要属性列表介绍

| 属性名 | 类型 | 可选值（解释） | 注意 |
| :-------------: | ----- | :------------- | :-----: |
| horizontal  | bool | true:所有的子视图会在水平方向上排成一行，而不是默认的垂直方向排列  | - |
| keyboardDismissMode  | enum | 用户拖拽视图时是否隐藏软件盘，none（默认值），拖拽时不隐藏软键盘。on-drag 当拖拽开始的时候隐藏软键盘。interactive 软键盘伴随拖拽操作同步地消失，并且如果往上滑动会恢复键盘。安卓设备上不支持这个选项，会表现的和none一样。  | - |
| keyboardShouldPersistTaps  | bool | 当此属性为false的时候，在软键盘激活之后，点击焦点文本输入框以外的地方，键盘就会隐藏。如果为true，滚动视图不会响应点击操作，并且键盘不会自动消失。默认值为false。  | - |
| onContentSizeChange  | bool | 此函数会在ScrollView内部可滚动内容的视图发生变化时调用。调用参数为内容视图的宽和高: (contentWidth, contentHeight)此方法是通过绑定在内容容器上的onLayout来实现的。  | - |
| onScroll  | function | 滚动监听：在滚动的过程中，每帧最多调用一次此回调函数。  | 调用的频率可以用scrollEventThrottle属性来控制。 |
| refreshControl  | element | 指定[RefreshControl][4]组件，用于为ScrollView提供下拉刷新功能。  | - |
| removeClippedSubviews  | bool | （实验特性）：当此属性为true时，屏幕之外的子视图（子视图的overflow样式需要设为hidden）会被移除。这个可以提升大列表的滚动性能。默认值为true。| - |
| showsHorizontalScrollIndicator  | bool | 当此属性为true的时候，显示一个水平方向的滚动条。  | - |
| showsVerticalScrollIndicator  | bool | 当此属性为true的时候，显示一个垂直方向的滚动条。  | - |


----------------


[0]: http://www.w3school.com.cn/html/index.asp
[1]: http://reactnative.cn/docs/0.39/view.html#content
[2]: http://reactnative.cn/docs/0.39/images.html
[3]: https://developer.android.com/guide/topics/manifest/activity-element.html
[4]: http://reactnative.cn/docs/0.39/refreshcontrol.html

