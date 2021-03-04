title: Using Icons in React Native
date: 2017-07-12 16:01:38
category: ReactNative
tags: SVG
---

<img src="/images/icons-in-rn/landing.jpg" alt="Landing" />

在 React Native 开发中，我们常常会有下面这样的需求：

1. 需要在 App 中显示图标
2. 图标需要兼容不同设备的分辨率（包括 Android 和 iOS 两个平台）

<!-- more -->

为了满足以上两个需求，我们不妨先思考，我们到底需要解决什么样的问题？

1. 显示图标的时候，这个图标应该既可以是一个图片文件，也可以是一个字体图标
2. 兼容不同分辨率这一点实际想要实现的是：图片缩放后不会失真

于是我们可以由以上分析得出两种方案：

方案一：使用图片文件来展示图标，并且在不同分辨率的设备上缩放后不失真
方案二：使用字体图标来展示，并且在不同分辨率的设备上缩放后不失真（缩放不失真正好是字体图标的特性）

那么，下面我们一起来看一下，以上两种方案在 React Native 中是怎样做的。

## 图片文件

### 位图图标

和原生 iOS 和 Android 开发类似，项目中加载图片时一般会采用`加载静态资源文件`的方案。为了适配不同分辨率的设备，一般的步骤为：

1. 设计出图标后，导出位图，此处以 `png` 为例
2. 同一份设计，分别生成多个不同分辨率的图片文件，给文件命名时后缀加上 `@2x` 和 `@3x`（最终在加载图标时，系统会根据当前设备的分辨率自动加载其对应分辨率的图片文件）

比如说，home 图标最终会在本地存三个文件：

`home.png`, `home@2x.png` 和 `home@3x.png`

React Native 的 [Image](http://facebook.github.io/react-native/releases/0.46/docs/images.html#static-image-resources) 组件提供了对这种用法的支持，只需要将资源文件都放入 `img` 文件夹下，然后使用时将 Image 组件的 `source`属性设置为对应的路径即可：

{% codeblock lang:javascript %}

<Image source={require('./img/home.png')} />

{% endcodeblock %}

注意：

此处 `require('./img/home.png')` 中，`'./img/home.png'` 的值只能是常量。
不能够动态拼接图片的路径，比如下面的用法是错误的：

{% codeblock lang:javascript %}

const imagePath = './img/home.png'
<Image source={imagePath} />

{% endcodeblock %}

这种方案可以解决问题，但是会有以下两个缺点：

1. App 中存储的静态资源文件增多，使得 App 打包后变大 (通俗的说-费流量 😅 )
2. UX( UI 设计师) 的工作量变大，本来只需要做一个图标，现在需要做三个了

### Alternatives

还有一个替代方案是在 React Native 中直接加载 SVG 图片来做图标。

但是需要注意的是，原生 React Native 的 Image 组件目前是不支持 SVG 的，以下代码是不工作的。

{% codeblock lang:javascript %}

<Image source={require('./img/home.svg')} />

{% endcodeblock %}

目前有一个第三方的库 [react-native-svg-uri](https://github.com/matc4/react-native-svg-uri) 提供了对渲染 SVG 图片的支持，但这个库在 Android 中有一些 KnownBugs，我们现在先不展开讨论这种方案了，大家感兴趣的话可以自行了解。

## 字体图标

[react-native-vector-icons](https://github.com/oblador/react-native-vector-icons) 提供了10个字图库的支持，其中包括：

* [`Entypo`](http://entypo.com) by Daniel Bruce (**411** icons) 
* [`EvilIcons`](http://evil-icons.io) by Alexander Madyankin & Roman Shamin (v1.8.0, **70** icons) 
* [`FontAwesome`](http://fortawesome.github.io/Font-Awesome/icons/) by Dave Gandy (v4.7.0, **675** icons) 
* [`Foundation`](http://zurb.com/playground/foundation-icon-fonts-3) by ZURB, Inc. (v3.0, **283** icons)
* [`Ionicons`](http://ionicframework.com/docs/v2/ionicons/) by Ben Sperry (v3.0.0, **859** icons)
* [`MaterialIcons`](https://www.google.com/design/icons/) by Google, Inc. (v3.0.1, **932** icons)
* [`MaterialCommunityIcons`](https://materialdesignicons.com/) by MaterialDesignIcons.com (v1.9.33, **1932** icons)
* [`Octicons`](http://octicons.github.com) by Github, Inc. (v5.0.1, **176** icons)
* [`Zocial`](http://zocial.smcllns.com/) by Sam Collins (v1.0, **100** icons)
* [`SimpleLineIcons`](http://simplelineicons.com/) by Sabbir & Contributors (v2.4.1, **189** icons)

### 常用图标

如果图标不需要特殊定制的话，引用这些库中的字体图标就可以了。比如：

{% codeblock lang:javascript %}

import Icon from 'react-native-vector-icons/FontAwesome'

const myIcon = (<Icon name="rocket" size={30} color="#900" />)

{% endcodeblock %}

### 自定义图标

很多项目中都需要使用自定义的图标，当设计好了图标之后，我们需要把图标转成字体，然后集成到 React Native 项目中。

`react-native-vector-icons` 是支持[自定义图标](https://github.com/oblador/react-native-vector-icons#custom-fonts)的，比较方便的方式是根据 [Fontello](http://fontello.com/) （或者 [Icomoon](https://icomoon.io/app)） 的配置来生成自定义的图标库。步骤为：

1. 将设计好的图标以 SVG 格式导出
2. 将 SVG 图片导入[Fontello](http://fontello.com/)（或 [Icomoon](https://icomoon.io/app)）
3. 导出字体和相关配置
4. 手动在iOS 和 Android 项目中配置字体资源
5. 在 React Native 中引用自定义字体图标

比较详细的步骤可以查看[博客](https://blog.bam.tech/developper-news/add-custom-icons-to-your-react-native-application)

这个方案可以解决`位图图标`方案中的问题。但是同样也不是很完美，因为每次更新图标时，需要重新生成字体文件和 `Fontello`（或者 `Icomoon`）的配置，并将配置和字体文件更新到项目中。

## 小结

本文主要介绍了两种在 React Native 中使用图标的两种方案，并列举了其优缺点。
不过最后还是那句话：没有最好的，只有最适合的。
在具体项目和需求面前，希望你可以结合实际情况找到最适合自己的方案。