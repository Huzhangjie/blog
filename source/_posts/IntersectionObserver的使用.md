---
title: IntersectionObserver的使用
date: 2021-08-31 10:35:48
subtitle: 利用IntersectionObserver实现懒加载
categories: 进阶
tags:
- javascript
- API
- 浏览器
cover: /images/b0fced9bf8864e88bb35b437b72f0c14.jpg
---

今天在研究替换felx-block的主题的代码块展示时，发现script代码中使用了[Intersection Observer API](https://developer.mozilla.org/zh-CN/docs/Web/API/Intersection_Observer_API)的API，便去学习了一下。
> IntersectionObserver接口 (从属于Intersection Observer API) 提供了一种异步观察目标元素与其祖先元素或顶级文档视窗(viewport)交叉状态的方法。

通俗一点说就是这个API能够监听到**目标元素**是否与**视窗**产生一个交叉区（是否滚动到视窗的位置）
![使用场景](20210831153114.png)


## 构造函数
`IntersectionObserver`是浏览器原生提供的构造函数
```javascript
const io = new IntersectionObserver(callback[, options]);
```
构造函数的返回值是一个观察器实例。实例的`observe`方法可以指定观察哪个 DOM 节点

```javascript
// 开始观察
io.observe(document.getElementById('example'));
// 停止观察
io.unobserve(element);
// 关闭观察器
io.disconnect();
```

`observe`的参数是一个 DOM 节点对象。如果要观察多个节点，就要多次调用这个方法。
```javascript
io.observe(elementA);
io.observe(elementB);
```

### 参数 callback
当元素可见比例超过指定阈值后，会调用一个`callback`，接受两个参数
`callback`一般会触发两次。一次是目标元素刚刚进入视口（开始可见），另一次是完全离开视口（开始不可见）。

```javascript
var io = new IntersectionObserver(
  (entries, observer) => {
    console.log(entries, observer);
  }
);
```
- **entries**: 一个`IntersectionObserverEntry`对象的数组
> ex: 如果同时有两个被观察的对象的可见性发生变化，entries数组就会有两个成员。
- **observer**: 被调用的`IntersectionObserver`实例


### IntersectionObserverEntry 对象
- **time**: 可见性发生变化的时间，是一个高精度时间戳，单位为毫秒
- **target**: 被观察的目标元素
- **rootBounds**: 返回根元素的 DOM矩形信息，[getBoundingClientRect()](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect) 的返回值
- **boundingClientRect**: 返回包含目标元素的矩形信息
- **intersectionRect**: 目标元素与视口（或根元素）的交叉区域的信息
- **intersectionRatio**: 返回 intersectionRect 与 boundingClientRect 的比例值
- **isIntersecting**: 返回目标元素是否与`root`元素相交

### 参数 options(可选)
observer实例的配置对象。具体配置如下

- **root**: 监听元素的祖先`Element`对象，root的边界即为`视口`。root默认值为`document`
- **rootMargin**: 相当于根元素的`margin`，用来影响计算交叉部分的矩形大小。默认值是`"0px 0px 0px 0px"`。
- **threshold**: 决定了什么时候触发回调函数。值为一个数组，每个成员都是一个门槛值。默认值为`[0]`，即交叉比例（intersectionRatio）达到0时触发回调函数
```javascript
// 表示当目标元素 0%、25%、50%、75%、100% 可见时，会触发回调函数
new IntersectionObserver(
  entries => {/* ... */}, 
  {
    threshold: [0, 0.25, 0.5, 0.75, 1]
  }
);
```


## 实现懒加载
图片懒加载，在图片元素滚动到视口时再请求图片资源

```html
<div class="list">
  <div class="item" background-image-lazy data-img="xxxx1.jpg"></div>
  <!-- n条数据 -->
  <div class="item" background-image-lazy data-img="xxxxn.jpg"></div>
</div>
```
```javascript
function handleLazyBG () {
  // 找出所有的 attr带 background-image-lazy的el
  const lazyBackgrounds = Array.from(document.querySelectorAll('[background-image-lazy]'))
  let lazyBackgroundsCount = lazyBackgrounds.length
  if (lazyBackgroundsCount > 0) {
    // 创建 observer实例
    let lazyBackgroundObserver = new IntersectionObserver(function(entries, observer) {
      entries.forEach(function({ isIntersecting, target }) {
        // isIntersecting 出现在视窗中，从data-set中取出 url
        if (isIntersecting) {
          let img = target.dataset.img
          if (img) {
            target.style.backgroundImage = `url(${img})`
          }
          // 取消订阅该元素
          lazyBackgroundObserver.unobserve(target)
          lazyBackgroundsCount --
        }
        // 关闭订阅
        if (lazyBackgroundsCount <= 0) {
          lazyBackgroundObserver.disconnect()
        }
      })
    })
    // 开启订阅每一个元素
    lazyBackgrounds.forEach(function(lazyBackground) {
      lazyBackgroundObserver.observe(lazyBackground)
    })
  }
}
```
点击查看[demo](https://codesandbox.io/s/thirsty-nightingale-8iktj?file=/index.html)
效果图：
![懒加载效果图](https://i.loli.net/2021/08/31/56eoHcd3tz4EVqS.gif)


## 注意点
IntersectionObserver API 是异步的，不随着目标元素的滚动同步触发。

规格写明，`IntersectionObserver`的实现，应该采用`requestIdleCallback()`，即只有线程空闲下来，才会执行观察器。这意味着，这个观察器的优先级非常低，只在其他任务执行完，浏览器有了空闲才会执行。

## 参考文献
- [Intersection Observer API - Web API 接口参考 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Intersection_Observer_API)
- [IntersectionObserver API 使用教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/11/intersectionobserver_api.html)