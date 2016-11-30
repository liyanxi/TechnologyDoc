### React Native学习札记3
### React Native基础组件与布局样式学习

----------------

### 本节主要介绍以下组件

- View 容器组件（同[Html][0]中的`div`标签）
- Text 文本组件（同[Html][0]中的`p`标签）
- Image 图片组件StyleSheet- TextInput 输入组件
- ScrollView 滑动容器组件（同原生组件`ScrollView`滑动方向水平和垂直）
- ListView 滑动列表组件（同原生`ListView`）
- 布局样式 StyleSheet

### 1.View 容器组件
 - 定义
 
	`<View>`React Native组件，相当于html中的`<div>`标签，具有相同的特性，支持嵌套。`<View>`组件可以把固定的区间分割成独立的、不同的部分。用于严格的模块化布局组织工具，默认内部`child`垂直排列,可调整排列方向，这点与Android原生组件`LinearLayout`很像。
	
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


----------------


[0]: http://www.w3school.com.cn/html/index.asp

