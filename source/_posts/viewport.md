---
title: 移动端适配总结
---

如果下面这行代码经常见到，但又对其一知半解，那么这篇文章也许可以帮你解惑

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

## 一、基础知识梳理

### 1.1、像素

#### a.物理像素

物理像素也就是移动设备中所提到的分辨率，比如手机的分辨率为`750x1334`像素，其指的是屏幕的垂直方向上有`1334`个物理像素块，水平方向上有`750`个像素块。其分辨率越高，所展示的图像就越清晰。

#### b.设备独立像素`(device-independent pixels)`/CSS像素

设备的独立像素，不同于物理像素，更多指的是设计稿中的尺寸，比如设计稿中一个Img的宽高为`100x100`像素，这个就是独立像素，也对应于css像素，是虚拟化的。

那么，可能会有疑问，这手机分辨率有`750x1334`像素，但是屏幕大小只有`375x667`像素，这两者有什么关系呢？

这里就需要引入css像素，也就是设备独立像素。css像素中1px可以代表多个物理像素，在dpr为2的手机中，CSS `1px`可以对应2个物理像素，也就是相比于分辨率为`375x667`的手机来说，`750x1334`分辨率的手机会更加清晰细腻。

如下图所示：这是`dpr = 2`的情况，最外框为css的`1px`，但其却覆盖了4个物理像素（蓝色的小方块）

<img src="/images/viewport/1.png" width = "100" alt="1" align=center />

当用户缩小时：物理像素（蓝色小方块）是不会改变的，此时一个物理像素从原来只覆盖1/4的CSS像素变成了可覆盖大于7/10的CSS像素，相当于一个物理像素所能展示的CSS像素更多，所以用户所能看到的内容也更多，但图像变小

<img src="/images/viewport/2.png" width = "100" alt="1" align=center />

反之，当用户放大时：原来一个CSS像素仅需要4个物理像素便能展示，放大后一个CSS像素所需的物理像素可能要8-9个（大概），所以能展示的CSS像素变少，图像变大，用户所能看到的内容也就变少了

<img src="/images/viewport/3.png" width = "100" alt="1" align=center />

### 1.2、设备像素比dpr

`dpr=物理像素/独立像素`，通过js可以获取：`window.devicesPixelRatio`。

## 二、视口viewport

### 2.1、PC与移动端的视口viewport

#### 2.1.1、PC端视口

在`html`、`body`设置`width:100%;height:100%;`的时候，它并不是无效的。100%这种百分数应该是继承父元素而来的。那在这里是继承哪里的呢？

在PC浏览器中，有一个用来约束CSS布局视口的东西，又叫做初始包含块。这也就是所有宽高继承的由来。除去`margin`、`padding`，浏览器布局视口`layout viewport`和可视窗口`visual viewport`宽高是一致的，统称为视口`viewport`，同时也和浏览器本身的宽度一致。

举个例子：下图中，当前浏览器的宽度`1365px`，则相应的body宽度也为`1365px`，文字显示正常，阅读无障碍

通过`document.documentElement.clientWidth`可以获取布局视口。

<img src="/images/viewport/4.png" width = "800" alt="1" align=center />



#### 2.2、移动端

##### 2.2.1、布局视口

<img src="/images/viewport/5.png" width = "400" alt="1" align=center />


在移动端，默认的情况下，移动端浏览器厂商为了让用户在小屏幕下传统的PC网页也能够很好地展示，一般会把布局视口宽度设置地很大去存放页面，在`768px ~ 1024px`之间，最常见的宽度是`980px`。这个宽度可以通过`document.documentElement.clientWidth`得到。

##### 2.2.2、视觉视口

<img src="/images/viewport/6.png" width = "400" alt="1" align=center />


对于视觉视口来说，这个东西是呈现给用户的，它是用户看到网页区域内CSS像素的数量。（CSS像素的数量越多，说明展示的页面元素越多，看到的页面元素也就越小）

值得注意的是，在移动端缩放不会改变布局视口的宽度，当缩小的时候，屏幕覆盖的css像素变多，视觉视口变大，看到的字体图形字体变小，反之亦然。

而在PC端，缩放对应布局宽度和视觉窗口宽度都是联动的。而浏览器宽度本身是固定的，无论怎么缩放都不受影响。

##### 2.2.3、视口缺陷

<img src="/images/viewport/7.png" width = "400" alt="1" align=center />

从上图可以看到，用户手机屏幕所展示的内容（`visual viewport`）仅占了`layout viewport`的一部分，并不能全部展示出来。



通过`chrome`模拟出手机的屏幕

如上图所示，布局视口的宽度（这里`body`被默认设置成了`980px`）是要远远大于移动端浏览器的宽度(`360px`)的。大多数浏览器会默认铺开整个布局视口到整个屏幕中，导致页面中的字体非常小，阅读体验非常差。

试想一下，如果一个网页不对移动端进行适配，用户进行阅读的时候，由于字体太小，就不得不手动放大以便于阅读，但是放大的时候另外一个问题又出现了：

<img src="/images/viewport/8.png" width = "400" alt="1" align=center />

从上图中可以看到，虽然文字变大了，但是相应的，屏幕中所显示的信息也变少了，用户需要手动去滑动整个页面让目标信息展示出来。



##### 2.2.4、理想视口

所以后来苹果引入了理想视口的概念，它对设备来说是最理想的布局视口尺寸，用户进入页面时就不需要手动再去缩放页面。

所以便有了这段代码：

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

**`width=device-width`：** 把布局视口设置成为浏览器（屏幕）的宽度。

**`initial-scale=1` ：**初始缩放的比例是1，使用它的时候，同时也会将布局视口的尺寸设置为初始缩放比例后的尺寸。而缩放的尺寸就是基于屏幕的宽度来的，也就起到了和`width=device-width`同样的效果。

每个移动设备浏览器中都有一个理想的宽度，这个理想的宽度是指css中的宽度，跟设备的物理宽度没有关系，在css中，这个宽度就相当于100%的所代表的那个宽度（这个其实就是`devices-width`）。

我们可以用`meta`标签把`viewport`的宽度设为那个理想的宽度，如果不知道这个设备的理想宽度是多少，那么用`device-width`这个特殊值就行了，同时`initial-scale=1`也有把`viewport`的宽度设为理想宽度的作用。

为什么需要有理想的`viewport`呢？比如一个分辨率为`320x480`的手机理想`layout viewport`的宽度是`320px`，而另一个屏幕尺寸相同但分辨率为`640x960`的手机的理想`layout viewport`宽度也是为`320px`，那为什么分辨率大的这个手机的理想宽度要跟分辨率小的那个手机的理想宽度一样呢？

这是因为，只有这样才能保证同样的网站在不同分辨率的设备上看起来都是一样或差不多的。实际上，现在市面上虽然有那么多不同种类不同品牌不同分辨率的手机，但它们的理想`viewport`宽度归纳起来无非也就 `320、360、384、400`等几种，都是非常接近的，理想宽度的相近也就意味着我们针对某个设备的理想`viewport`而做出的网站，在其他设备上的表现也不会相差非常多甚至是表现一样的。



## 三、移动端适配方案

在移动端上，有了`layout viewport`的固定宽度之后，再进行移动端适配就不再是难事，简单总结了如下几种移动端适配的方法：

### 3.1、flex弹性盒模型实现的移动端适配

以淘宝移动端页面为例：https://main.m.taobao.com/index.html

通过将盒子设置成`flex`布局以及`width:100%`，根据需要来设置`justify-content`以及`align-items`来对齐居中等，实现在不同屏幕尺寸的移动端下的适配。

在需要适配的地方使用弹性盒模型以及百分比，在不需要适配的地方使用px，举个例子，底部tap就是适配的，而中间的导航，如”天猫新品，今日爆款“的图标块是原始`px`尺寸。

<img src="/images/viewport/9.png" width = "400" alt="1" align=center />



这种适配方案在手机端是OK的，但一旦转换到ipad端....，如下图所示，中间的导航栏由于`div`宽度过长，而其中的图标大小还是原始尺寸，导致盒子过大，图标无法填充满整个盒子，更别说是切换分页了（这个盒子中是有分页器的）

<img src="/images/viewport/10.png" width = "400" alt="1" align=center />

**这种适配方案的优缺点：**

**优点：**
实现简单便捷，`flex+百分比/flex+vw`灵活方便，无需引入其他库便可以实现大部分手机的屏幕适配
**缺点：**
针对于平板类的移动端，还需要另外做适配才能兼容，比较繁琐

### 3.2、rem+px2rem适配方案

> 在W3C官网上是这样描述rem的——“font size of the root element” 。

基本思路：

假如说我们的设计稿为`750px`，现在把设计稿分成`10`份，一份就为`75px`。

同样，获取到移动端屏幕的宽度，将其分成十份，并且将每份的大小设置成`1rem`，也就是设置`html`的`font-size`。设置`div`大小时，便可用`rem`进行设置，比如，设计稿中一个`Img`是`75 x 75`，代码中就可以设置这个`img`为`1rem`，即`img`的宽度是设计稿的`1/10`，对应的比例在手机中也就是屏幕宽度的`1/10`，也就是`1rem`。分成几份全由自己来决定，可以分成`10`份，也可以分成`20`份，等等。

这样可以实现适配，但是问题在于，每次写css的宽高时，都需要通过自己再进行计算转换成`rem`，比如如果有个`img`是`50x50`像素，它在设计稿中的比例是 `50/75 = 0.667`份，相当于在代码中我们需要写`0.667rem`，这其实是非常繁琐且低效的。

但是，现有工具可以帮我们解决这个问题：`postcss-px2rem`。我们在写代码的时候还是可以用`px`，设计稿中写了多少`px`我们代码中就写多少`px`，在代码编译阶段，`postcss-px2rem`就会将`px`编译成相应的`rem`，这样我们就无需手动去换算，这是一种比较成熟的移动端适配方案。（这里不深入探究`1px`的问题，解决`1px`问题需要配合`scale`以及`dpr`进行）



下面以`vue`项目且`750px`设计稿为例：

我们设定设计稿`html`的`fontSize = 75px`，则`1rem = 75px`(分成10份)，体现在`postCss-px2rem`中就是`remUnit: 75`。这时在代码中写`1px`，代码编译后会转换成` 1/75 = 0.01333rem`，而这个`0.01333rem`是相对于移动端屏幕的，

举个例子：如果移动端屏幕为`375px`，且我们对移动端网页的`html`设置了`fontSize = 375 / 10 = 37.5px`，则`0.013333rem = 37.5 * 0.013333 = 0.5px`，其比例和设计稿是一致的，所以显示在屏幕中也就是正常的。



设置`font-size`的代码如下（引入的方式按需而定，比如这段代码可以通过一个`script`标签内联在`app.html`）：

```js
const width = document.documentElement.clientWidth;
const html = document.getElementByTagName('html')[0];
html.style.fontSize = width / 10 + 'px';
```


配置`postCss-px2rem`：

```js
// postcss.config.js
module.exports = {
    plugins: {
        autoprefixer: {},
        'postcss-px2rem': {
            remUnit: 75
        }
    }
};
```



### 3.3、`vw+px2rem`适配方案

在移动端适配发展的初期，浏览器对`vw`的兼容性相比于`rem`是差很多的，但是随着浏览器的发展，慢慢地对`vw`的兼容相比于以前有了天翻地覆的变化

回想`rem`适配，若是将设计稿分成100份的时候，其实`1rem`也就对应了`1vw`，两者的原理是差不多的，`vw`的好处就体现在其不再需要引入`js`，直接通过css代码便可以设置，

代码如下：

<img src="/images/viewport/11.png" width = "600" alt="1" align=center />

其中，还是将设计稿分成了十份，如果设计稿为`1080px`，则一份为`108`像素，注意，这两个值是通过自己去设置的，并不是一定要这样设置，可以根据设计稿以及自己方便计算来设置；

设计稿的`html fontSize`设计成`1/10`的设计稿宽度`108`，所以`postCss-px2rem`中的`remUnit`就为`108`，相应的移动端页面中`1rem = 10vw`，相当于在`375`的屏幕中`1rem = 37.5px`。

同时，也可以利用`px2rem`工具，写代码时就可以直接利用`px`。



**<u>参考文章：</u>**

**viewport**

[A tale of two viewports — part one](https://www.quirksmode.org/mobile/viewports.html)

[A tale of two viewports — part t](https://www.quirksmode.org/mobile/viewports2.html)

[Meta viewport](https://www.quirksmode.org/mobile/metaviewport/)

[移动前端开发之viewport的深入理解](https://www.cnblogs.com/2050/p/3877280.html)

**移动端适配**

[使用Flexible实现手淘H5页面的终端适配](https://github.com/amfe/article/issues/17)

[再聊移动端页面的适配](https://juejin.im/entry/5a9d07ee6fb9a028c149f55b)















