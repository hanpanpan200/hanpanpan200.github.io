title: 谈谈依赖的版本控制配置
date: 2016-08-19 11:08:53
category: ReactNative
tags: 依赖版本控制
---


项目中引用第三方包你一般会在什么时候更新呢？是一旦用了某个版本就再也不会更新？还是一有新版本就马上更新？还是定期检查release note，根据自己的需要更新？

在一个React Native项目中，考虑到React Native升级的很频繁，于是团队决定，我们会固定一个React Native版本，每过一个阶段之后，由团队中一个人去完成升级工作。等这个人本地的升级完成，测试通过后，将升级后的代码上传，其他人再基于最新升级后的代码做相关的Feature。

这个故事就发生在一次React Native的升级过程中。

<!-- more -->

## 修改依赖的版本控制配置

由于Jest现在需要react native 至少0.30.0的支持了，目前ReactNative的最新稳定版本在0.31.0。
于是昨天就将React Native升级到了 0.31.0。

> react-native@0.26.3 => react-native@0.31.0
react@15.0.2 => react@15.2.1

升级完之后，肉测了项目里所有为数不多的功能之后(等把Jest测试加上，就终于不用每次改点代码就心惊肉跳了)，今天早上merge了，自信满满地提醒同事们可以升级了。

20分钟之后，反应来了，Berry说他本地升级完之后出现了很多这样子的warning:

> Warning: You are manually calling a React.PropTypes validation function for the 'bottom' prop on 'StyleSheet.fullScreen'. This is deprecated...

咦？怎么会酱紫？我本地怎么都没有？google 之

后来查到了这个 [issue](https://github.com/facebook/react-native/issues/9093)

这是react@15.3.0的bug，竟然出现在了Berry的机器上。。辣么。。

请Berry查了他本地的react版本，确认一下凶手。

 `npm ls react`

 结果就是 `react@15.3.0`

 瞄一眼`package.json`

```json
 "react": "^15.2.1",
 "react-native": "^0.31.0",
```
对！凶手只有一个！他就是！ 我！。。

`^15.2.1` 意思是 `15.2.1 <= version < 16.0.0`，回想升级的过程

按照[Official docs](https://facebook.github.io/react-native/docs/upgrading.html) 执行的
```
npm install --save react-native@0.31
```

升级完之后，默认更新了 `package.json`， 都怪自己粗心没有留意版本。

最后方案是：把版本号固定，日后升级由某一个人单独做升级，并且解决了升级过程中出现的问题之后，所有人再一起upgrade，这样既可以达到升级目的，也可以划分明确的关注点，升级的人只做好升级，做功能的人专心做功能不用担心由于跑一遍 `npm install` 就跑出来莫名其妙的warning 和 error。

```json
 "react": "15.2.1",
 "react-native": "0.31.0",
```
以此类推，修改类似 `package.json` 或者 `gemfile` 这种跟版本有关的配置时，一定要谨慎，特别是变化比较快的库，更是更加谨慎！
