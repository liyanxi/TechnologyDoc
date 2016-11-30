### React Native学习札记2
### Props（属性）与State（状态）

----------------

>我们通常使用两种数据来控制一个组件：props和state。props是在父组件中指定(传递到子控件)，而且一经指定，在被指定组件的生命周期中则不再改变。 对于需要改变(通常指用户交互反馈)的数据，我们需要使用state重新渲染(render)组件(即实现局部刷新)。——[React Native 中文网](http://reactnative.cn/docs/0.38/state.html#content)

下面就让我们分别对 props 和 state 做相应的介绍和使用：

### 1.Props(属性)
- 定义：大多数组件在定义时就可以通过各种自定义参数来实现组件的定制。对外提供的这些参数就称为`props` （属性）。

- 以常见的基础组件Image为例，在创建一个图片时，可以传入一个名为source的prop来指定要显示的图片的地址，以及使用名为style（同css样式style属性）的prop来控制其尺寸。

```
/**
 * 图片尺寸
 * Created by liyanxi on 2016/11/28.
 */

import React, {Component} from 'react';
import {StyleSheet, View, Image} from 'react-native';

export default class ImageDimension extends Component {
    render() {
        let pic = {
            uri: "https://upload.wikimedia.org/wikipedia/commons/d/de/Bananavarieties.jpg"
        }
        return (
            <View style={styles.container}>
                <Image source={pic} style={styles.imgStyle}></Image>
            </View>
        );
    }
}
const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',//vertical
        alignItems: 'center',   //horizontal
        backgroundColor: '#F5FCFF',
    },
    imgStyle: {
        width: 193,
        height: 110
    }
});
```

`注` 上面`return`返回语句为[JSX](http://reactjs.cn/react/docs/jsx-in-depth.html)语法，在`render`函数中组件以`<`开始，表达式或变量以`{`开始；此处`source`为Image加载图片资源链接参数不可更改，`{}` 括号内为js变量或表达式，需要执行后取值，因此我们可以把任意合法的JavaScript表达式通过括号嵌入到JSX语句中。

- 自定义的组件也可以使用`props`。通过在不同的场景下使用不同的属性定制，从而提高自定义组件的复用度。只需在`render`函数中引用`this.props`--property-->`this.props.property`即可。下面看`自定义`组件和实际调用`render`函数代码：

```
class Greeting extends Component {
  render() {
    return (
      <Text>Hello {this.props.name}!</Text>
    );
  }
}
class LotsOfGreetings extends Component {
  render() {
    return (
      <View style={{alignItems: 'center'}}>
        <Greeting name='Rexxar' />
        <Greeting name='Jaina' />
        <Greeting name='Valeera' />
      </View>
    );
  }
}
```
`注` 上面代码没什么可说的，主要注意一点：入参名称（name）要与自定义组件内部调用名称（name）一致，负责会报错（入参或出参找不到或未定义），比较好的写法就是在自定义组件`Greeting`中默认初始化`props`即：`this.props = {name:""}`，涉及到`props state`初始化的不同（ES5与ES6的 [区别 ]("http://bbs.reactnative.cn/topic/15/react-react-native-%E7%9A%84es5-es6%E5%86%99%E6%B3%95%E5%AF%B9%E7%85%A7%E8%A1%A8")）此处不做详解，后面会单独说明。

### 2.State(状态）
- 前面已经描述过，作为用户交互设计产生的反馈通常都是通过`state`来进行数据的改变、界面的刷新。
与`props`一样都是作为控制组件的，唯一的区别是否需要及时改变。引用方式也比较简单同`props`即：`this.state.property`,比`props`多的一个方法就是在于改变`setState`,在需要更新数据的地方调用此方法设置值就行（实际相当于重新调用`render`函数）。具体使用可参考如下代码示例：

```
/**
 * react native 输入组件调试
 * Created by liyanxi on 2016/11/28.
 */
import React, {Component} from 'react';//ES6语法
import {StyleSheet, View, TextInput, Text} from 'react-native';

export default class TextInputComponent extends Component {//ES6语法
    /**
     * ES6 构造函数中：state 或 props 初始化
     * ES5 getDefaultProps、getInitialState
     */
    constructor(props) {
        super(props);
        this.state = {inputText: ""};
    }
    /**
     * 唯一根布局
     * @returns {XML}
     */
    render() {
        return (
            <View style={styles.container}>
                <TextInput
                    style={styles.input}
                    placeholder={"Type here to translate!"}
                    onChangeText={(text)=> this.setState({inputText: text})}>
                </TextInput>
                <Text style={styles.showText}>
                    {this.state.inputText.split(" ").map((word)=> word && 'Q').join(" ")}
                </Text>
            </View>
        );
    }
}
const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 10
    },
    input: {
        fontSize: 20,
        borderWidth: 1,
        borderColor: 'skyblue',
        // padding: 5 系统默认padding简直是个坑,可根据实际自身做调整
    },
    showText: {
        fontSize: 32
    }
});
```
`注` 此处代码中标注的`ES5`与`ES6`的区别此处不做详述（后续单独讲解），需要学习的可参考 <a href="http://bbs.reactnative.cn/topic/15/react-react-native-%E7%9A%84es5-es6%E5%86%99%E6%B3%95%E5%AF%B9%E7%85%A7%E8%A1%A8" target="_blank" style="text-decoration:none;">React/React Native ES5 ES6写法对照表</a>
### 3.完整代码见Github
- <a href="https://github.com/liyanxi/AwesomeProject" target="_blank" style="text-decoration:none;"> AwesomeProject Demo实例 </a>

------------------

### 学习网站
- [React Native 中文网](http://reactnative.cn/)
- [React.js官网](http://reactjs.cn/react/docs/displaying-data.html)






