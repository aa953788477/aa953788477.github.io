---
title: react-native 给android端设置启动图
date: 2016-07-22 22:45:33
tags:
- 移动端
- react-native
---

公司的 [环保头条](http://www.hbtoutiao.cn/) 安卓端，全部使用 `react-native` , 相对之前的webapp 程序, 它的动画性能更好，更加贴近于原生的体验。 app功能开发基本完成后，发现一个很严重的问题，在app启动的时候会有很明显的白屏现象，不同的机型不同（cpu好的白屏时间短），大概1s到2s的时间。

为了解决这个问题，找了很多的资料，参考了 [ReactNative安卓首屏白屏优化](http://reactnative.cn/post/754)， 也并没有解决掉启动慢点问题 ， 原因是全react-native的应用，内存换效率的做法并不起作用。 
于是我决定曲线就够，在app启动时先展示一张背景图片，当启动后隐藏。

<!--more-->

首先在 `android/app/src/main/res/drawable-hdpi` 里加上一个图片作为程序启动图 (名称随意，例如：`splash.png`) 

然后修改 `android/app/src/main/res/values/styles.xml`

```xml
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
    <!-- 这里将刚刚那张图片设置为背景图片， splash对应图片名称 -->
    <item name="android:windowBackground">@drawable/splash</item>
</style>
```

最后修改我们的 `react-native` 顶层的容器，将它覆盖整个窗口

```javascript
<View style={styles.container}></View>

const styles = StyleSheet.create({
	container: {
		flex: 1,
		backgroundColor: '#FFF' //背景色根据自己的需求来
	}
})
```

ok, 到这一步， 启动应用看看，会发现已经没有白屏了，会有一张覆盖全屏的启动图等待。

在 *安卓5.0* 以上会有沉浸式状态栏闪烁的问题，原因是，react-native默认是不设置状态栏的，所以我们需要在 `MainActivity.java` 中增加如下代码：

```java
// 这里重写 createRootView 方法保证在耗时操作前完成状态栏设置
@Override
protected ReactRootView createRootView() {
     if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
        Window window = getWindow();
        // 设置为沉浸式状态栏
        window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
    }
    return super.createRootView();
}
```

到此，就全部完成啦，妈妈再也不用担心我的 app 启动白屏啦！


