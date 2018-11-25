---
title: react-native 开始走向mobile开发
date: 2016-07-17 09:38:24
tags:
- 移动端
- react-native
---

从去年开发接触移动端web开发，做了许多的webapp程序，嵌入原生程序中的表现都不太好，老是会卡顿。也尝试过许多的hybird解决方案， `phoneGap` 和 `H5+` 这两者都是以 `webview` 为核心的开发。由于andorid系统的原因，变现都不太好（IOS上还是很流畅的）。

去年下半年开始关注 [react-native](http://reactnative.cn/), 一直也没有机会正式拿来开发, 最近公司要开发android版的 [环保头条](http://www.hbtoutiao.cn/), 决定使用react-native来开发。体验之后性能的确是比webview的方式好太多。 所以决定记录一下react-native 应用开发的一些知识点
<!--more-->

## 一、配置开发环境
这个环境配置，官方文档给的很全我就不多做赘述了，按照 [开始使用React Native
](http://reactnative.cn/docs/0.28/getting-started.htm) 文档配置就可以了。

## 二、应用结构

使用命令 `react-native init` 生成项目之后会有一个这样的文件结构

![14691964106736](/images/14691964106736.jpg)

* android/ios  存放的是双平台的代码，如果不需要加载原生模块，这两个文件夹中的文件就可以不用管
* node_modules node模块
* src          这个是自己建的目录，用于存放项目源码
* index.[ios/android].js 启动时会根据平台运行不同的js

剩下的就都是配置文件

src 下是项目结构

* actions/store/reducers   是redux相关的
* configs/contants/images  配置文件和图片
* serices                  请求的相关的js
* components               组件
* layouts                  页面

## 三、用JSX语法构建页面ui
构建成功之后，打开 `index.android.js` , 会发现render方法下这样的代码

```javascript
 <View style={styles.container} >
	 <Nav title="关于" router={this.props.router}></Nav>
    <View style={styles.wrapper}>
       <View style={styles.main}>
	       <Image style={styles.logo} source={require('../images/logo.png')}/>
	       <Text style={styles.title}>环保头条</Text>
	       <Text style={styles.subTitle}>您关心的就是头条</Text>
	       <Text style={styles.version}>当前版本:   V1.0 (Build:42)</Text>
       </View>
       <View style={styles.footer}>
          <Text style={styles.agreement}>用户协议</Text>
       	<Text style={styles.copyRight}>技术支持：深圳博安达信息技术股份有限公司</Text>
       </View>
    </View>
</View>
```

react 是基于虚拟dom的设计， 所谓虚拟dom，就是讲所有的 在view和逻辑之间，抽象出的对象，最后将这些对象解析成页面，不限web 或者原生，只是给出不同的实现就可以了。 如果不用jsx, react的语法写起来就是这样的

```javascript
React.createElement('View', {
	style: {}
}, React.createElement('View'))
```

这么一看还是jsx方便，虽然网上有很多抨击jsx 的言论， 但是jsx 对于虚拟DOM 来说仍然是最好的一种方式， Vue2.0也用jsx了，虽然也保留了template的方式，但是尤大大也说了很多情况还是要使用jsx的。

最后，jsx 只是一种便于在js里写虚拟dom对象的语法而已，并没有什么特别的，还是很容易上手的。

[jsx写法看这里](http://reactjs.cn/react/docs/jsx-in-depth.html)

## 四、stylesheet 样式管理
用了react-native就是不能像web开发一样愉快的写样式了， react-native 也提供了自己的样式定义的方法,
使用 flexbox 布局，也支持 absolute觉得定位，基本和 css中一直

```javascript
const styles = StyleSheet.create({
    container: {
        backgroundColor: 'white',
        width: width,
        height: height,
        position: 'relative'
    },
    wrapper: {
        flex: 1,
        paddingTop: 50,
        paddingBottom: 80,
        alignItems: 'center',
        justifyContent: 'space-between'
    }
 })
 
<View style={styles.container}/>
```

关于flexbox布局，阮一峰老师写过一篇详细的教程 [Flex 布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

react-native也并不是什么属性都支持，具体到每一个组件，例如 : 对于字体相关的样式 fontSize， color,这些只有Text 支持，而Text组件并不支持width和height这些组件。

## 五、redux管理你的app

react是通过状态的变化来改变页面的展示，每一个组件都有内部状态，当用户点击页面或者滑动的时候都会触发事件，对应的改变状态，更新页面展示。

如果没有状态管理的话，每一个组件就是相对独立，组件之前有交互变得很难, 例如：子组件中的一个操作，需要改变父组件的状态，这种场景控制起来很麻烦。

redux就是问了解决这种问题而生的框架，整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 `store` 中。 页面中的组件发生事件，就产生一个 `action` , `action`将参数转给 `reducer` ， `reducer` 进过逻辑处理返回新的 `state` redux响应 state变化，来更新页面。

一旦使用 `redux` 就尽量不要在组件内部创建 `state` , 一个组件的标准定义应该如下:

1. props 参数，根据不同的参数展示不同页面
2. action 组件中响应用户操作产生action,经redux修改状态 传入新的 props
3. 响应props改变，更新页面。


## 六、网络请求 fetch

> React Native的目标之一是创造一个实验场，让我们可以实验不同的技术和一些疯狂的想法。因为浏览器环境还不够灵活，我们除了自己实现整个技术栈以外别无选择。其中有些部分我们没有刻意修改，而且希望尽可能地忠实于浏览器的API。网络API就是一个典型的例子。

使用方法

```javascript
fetch('https://mywebsite.com/endpoint/').then(function(res){
	return res.json()
})
```

fetch 是一个基于异步处理 `Promise` 的设计，更加的方便好用，现在浏览器环境也支持了（仅谷歌）, 第二的参数是可选的，用于配置一些请求参数，例如下面这样发送一个 *post* 请求

```javascript
fetch('https://mywebsite.com/endpoint/', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
  },
  body: 'key1=value1&key2=value2'
})

```

更多 fetch 和 Promise 讲解看这里就好啦：

1. [JavaScript Fetch API](http://www.tuicool.com/articles/QZBJ7zJ)
2. [Promise对象](http://es6.ruanyifeng.com/#docs/promise)

另外 react-native 还支持 XMLHttpRequest 和 WebSocket, [网络](http://reactnative.cn/docs/0.28/network.html#content)

## 七、Navigator 不同场景（页面）间进行切换

之前做webapp的时候最难搞的就是不同场景间的切换，虽说都有router和场景动画，但是很难分清楚，前进和返回的不同动画， 也没有手势返回，就算模拟出来了这两者，性能也不一定好。

在 `react-native` 中提供 Navigator 组件， 用来控制不同页面间的切换, 基本使用方式如下

```javascript
<Navigator
    initialRoute={{name: 'My First Scene', index: 0}}
    renderScene={(route, navigator) =>
      <MySceneComponent
        name={route.name}
        onForward={() => {
          var nextIndex = route.index + 1;
          navigator.push({
            name: 'Scene ' + nextIndex,
            index: nextIndex,
          });
        }}
        onBack={() => {
          if (route.index > 0) {
            navigator.pop();
          }
        }}
      />
    }
  />
```

在实际使用中，我们可以配合自定义的router对象，封装的更加完美一些

```javascript
class Router {
	constructor(navigator) {
		this.navigator = navigator
		this.exitflag = false
		if (Platform.OS === 'android') {
			BackAndroid.addEventListener('hardwareBackPress', ()=> {
				const routesList = this.navigator.getCurrentRoutes()
				const currentRoute = routesList[routesList.length - 1]
				if (currentRoute.name !== 'main') {
					navigator.pop()
					return true
				} else if (!this.exitflag) {
					this.exitflag = true
					ToastAndroid.show('再按一次退出！', ToastAndroid.SHORT)
					setTimeout(() => {
						this.exitflag = false
					}, 3000)
					return true
				}
				return false
			})
		}
	}


	push(props = {}, route) {
		this.exitflag = false
		let routesList = this.navigator.getCurrentRoutes()
		let nextIndex = routesList[routesList.length - 1].index + 1
		route.props = props
		route.index = nextIndex
		route.sceneConfig = route.sceneConfig ? route.sceneConfig : 		CustomSceneConfigs.customPushFromRight
		route.id = _.uniqueId()
		route.component = connectComponent(route.component)
		this.navigator.push(route)
	}


	pop() {
		this.navigator.pop()
	}

	toNews(props) {
		this.push(props, {
			component: News,
			name: 'news'
		})
	}

	toAbout(props) {
		this.push(props, {
			component: About,
			name: 'about'
		})
	}
}
```

```javascript
// 不同的场景 创建不同的element
renderScene({ component, name, props, id, index }, navigator) {
		this.router = this.router || new Router(navigator)
		if (component) {
			return React.createElement(component, {
				...props,
				ref: view => this[index] = view,
				router: this.router,
				route: {
					name,
					id,
					index
				}
			})
		}
}
render() {
	return (
			<View style={styles.container}>
	            <StatusBar
	                backgroundColor={config.theme_color}
	                barStyle="light-content"
	            />
				<Navigator
					ref={view => this.navigator=view}
					initialRoute={initialRoute}
					configureScene={this.configureScene.bind(this)}
					renderScene={this.renderScene.bind(this)}/>
				<Utils/>
			</View>
		)
}
```

## 八、AsyncStorage 数据存储

解决了场景切换，数据缓存也是之前webapp开发的痛点， 主要的原因是localstorage的容量不一致，而且用户也可以直接清除浏览器缓存。

AsyncStorage是一个简单的、异步的、持久化的Key-Value存储系统，它对于App来说是全局性的。它用来代替LocalStorage， 使用方式非常简单：

```javascript
mport React, {
	AsyncStorage
} from 'react-native'


export async function setItem (key, value) {
	if (value == null) return Promise.reject('StorageService Error: value should not be null or undefined')
	return await AsyncStorage.setItem(key, JSON.stringify(value))
}


export function getItem (key) {
	return AsyncStorage.getItem(key)
		.then(function (value) {
			return JSON.parse(value)
		})
}

```

* async await 是es7的新特性，用于定义一个异步的函数

## 九、Animated 动画处理

> 动画是现代用户体验中非常重要的一个部分，`Animated` 库就是用来创造流畅、强大、并且易于构建和维护的动画。
最简单的工作流程就是创建一个 `Animated.Value` ，把它绑定到组件的一个或多个样式属性上。然后可以通过动画驱动它，譬如 `Animated.timing` ，或者通过 `Animated.event` 把它关联到一个手势上，譬如拖动或者滑动操作。除了样式，`Animated.value` 还可以绑定到props上，并且一样可以被插值。这里有一个简单的例子，一个容器视图会在加载的时候淡入显示

```javascript
class FadeInView extends React.Component {
   constructor(props) {
     super(props);
     this.state = {
       fadeAnim: new Animated.Value(0), // init opacity 0
     };
   }
   componentDidMount() {
     Animated.timing(          // Uses easing functions
       this.state.fadeAnim,    // The value to drive
       {toValue: 1},           // Configuration
     ).start();                // Don't forget start!
   }
   render() {
     return (
       <Animated.View          // Special animatable View
         style={{opacity: this.state.fadeAnim}}> // Binds
         {this.props.children}
       </Animated.View>
     );
   }
 }
```

以上的代码和介绍来源于 [官方文档](http://reactnative.cn/docs/0.28/animated.html#content) ， 动画目前还不是特别熟悉，但是使用之后还是非常流畅的。


## 最后

总的来说，react-native 开发出来的app 还是非常流畅的，现场它的生态也非常好，一些ui库和原生模块很快就能找到。上个月阿里也发布了原生方案 `weex` ，就目前来说生态还是不太好，不能直接在项目上使用。 我们需要开发双端一致的app时， react-native就成了不二之选。






