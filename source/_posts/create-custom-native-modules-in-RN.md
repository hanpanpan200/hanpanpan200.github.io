title: 自定义 React Native 模块
date: 2017-07-16 16:42:17
category: ReactNative
---

<img src="/images/create-custom-native-modules-in-RN/landing.jpeg" alt="Landing" width="80%"/>

虽然 React Native 目前已经提供了很多常用的组件，但是在实际项目中，我们总会发现需求往往是这些常用组件所不能够满足的。比如说：图像识别模块、指纹识别模块等。这种模块大部分已经有了比较成熟的SDK，我们此时需要做的就是将已有的轮子，封装到 React Native 中来使用。

经过一些尝试和探索后，我终于找到了一个比较容易上手的封装 React Native 插件方案，希望能得到更多的反馈来改善。

<!-- more -->

## 目录结构

React Native 支持 iOS， Android 和 Windows 三个平台。本文我们只考虑 iOS 和 Android。

RN 自定义模块的目录结构的基本组成为：

{% codeblock lang:javascript %}

-android
-ios
-index.js
-package.json
-README.md

{% endcodeblock %}

其中，
android：Android平台的原生代码
ios：iOS 平台的原生代码
index.js：自定义模块的入口（还有一种方式是定义 index.ios.js 和 index.android.js来分别作为对应平台的入口）
package.json： 自定义组件的npm包信息
README.md：相关文档说明

## 脚手架

在清楚了目录结构之后，我们便可以动手写代码了。不过你不必一一手动创建上述的每个文件，现有的脚手架可以帮我们快速搭建起这个架子，随后我们只需要补充进自己的逻辑代码即可。那么下面我们来看看这些脚手架。

### react-native new-library

原生 React Native 提供了一个命令：

{% codeblock lang:javascript %}

react-native new-library --name "YourLibName"

{% endcodeblock %}

需要注意的是：如果你在一个非 React Native 项目的目录下执行以上命令，会得到 error

> Command `new-library` unrecognized. Make sure that you have run `npm install` and that you are inside a react-native project.

也就是说，使用原生的 `react-native new-library` 命令时需要在已有的 React Native 项目中去执行，比如：

{% codeblock lang:javascript %}

react-native init AwesomeProject
cd AwesomeProject
react-native new-library --name "YourLibNameA"
react-native new-library --name "YourLibNameB"

{% endcodeblock %}

上述命令帮我们做了以下两件事情：
1. 在 `AwesomeProject/ios/` 目录下，创建一个 `Libraries/` 目录。
2. 把我们上面创建的`YourLibNameA` 和 `YourLibNameB` 放在 `Libraries/` 下面。

它的优势在于：
1. React Native 官方出品，会随着 React Native 一起升级，较稳定。

它的局限在于：
1. 需要建立在现有的 RN 项目之上，组件库代码和产品代码混在一起，不利于单独开发、发布和维护。
2. 只生成了一个 iOS 项目的 library，不支持 Android 。
3. 如需 link, 需要手动将相应的依赖添加到 iOS 项目中。

下面我们来介绍另外一种方式。

### create_react_native_library

首先需要安装：

{% codeblock lang:javascript %}

npm install -g react-native-create-library

{% endcodeblock %}

之后，你便可以在任意目录下执行：

{% codeblock lang:javascript %}

react-native-create-library AwesomeLib

{% endcodeblock %}

上述命令会创建一个独立的工程，文件的目录结构为：
{% codeblock lang:javascript %}

-android
-ios
-windows
-index.js
-package.json
-README.md

{% endcodeblock %}

当我们不指定 `--platforms` 参数时，会默认创建包含 `android`, `ios` 和 `windows` 三个平台的工程。

[create_react_native_library](https://github.com/frostney/react-native-create-library) 与 `react-native new-library` 相比，它的优势在于：

1. 不依赖于现有的 RN 项目，作为独立项目进行开发、发布和维护。
2. 兼容 iOS, Android 和 Windows平台，支持通过参数指定 `platforms` 和一些常用的参数比如： `android package id` 
3. 封装了 `react-native link` 的逻辑，执行 `react-native link` 即可完成对应平台的依赖设置。

它的局限在于：非官方出品，需关注是否与最新的 React Native 版本兼容。

目前 `create_react_native_library` 支持最新的`react-native@0.51`。

## 调试和测试

当我们用 create_react_native_library 创建了独立的自定义组件库之后，我们怎样去调试这个新开发的 npm 模块呢？

做法有很多：

1. 在 npm 上 publish 一个 beta 版本，然后在项目中反复：安装 npm beta package -> 微调 -> 安装
2. 在本地用相对路径安装，然后反复进行： 安装本地依赖 -> 微调 -> 安装
3. [yarn-link](https://yarnpkg.com/lang/zh-hans/docs/cli/link/)

通过 `yarn-link` 的方式来进行调试最为方便。下面举例说明：

> 已有项目 project，需要引用的自定义模块为项目 utility（package name 为 react-native-utility）。
目录结构如下：

{% codeblock lang:javascript %}

-source
  |-project
  |-utility

{% endcodeblock %}

调试时，需要先创建一个 symlink 链接到本地的 utility ， 然后再在 project 中，使用 link 了的包：

{% codeblock lang:javascript %}

cd source/utility
yarn link
cd ../project
yarn link react-native-utility

{% endcodeblock %}

注意：

**1. Link 与 Unlink**

在调试过程中改动了 utility 中的依赖后，常由于 cache 而导致依赖未及时更新而找不到而报错。 此时需要执行以下操作重新 link：

{% codeblock lang:javascript %}

cd source/utility
yarn unlink
yarn link
cd ../project
yarn unlink react-native-utility
watchman watch-del-all && rm -rf $TMPDIR/react-* && rm -rf node_modules/ && yarn install
yarn link react-native-utility

{% endcodeblock %}

**2. react-native link**

如自定义组件需要添加 native 项目的依赖，可通过 `react-native link react-native-utility` 来配置。
但是由于 react-native link 执行时需要从 package.json 中读取 dependency, 因此需要先将 react-native-utility 添加到 `package.json` 的 `dependencies` 之后再执行 react-native link react-native-utility。
又由于通过 `yarn link` 进行调试时不需要更新 `package.json`，因此在调试阶段，在执行了 `react-native link react-native-utility` 之后，再将 package.json 中的更新移除。

在完成了本地的开发和调试之后，我们就可以将这个自定义组件发布到 npm 了。

## 发布组件到 npm 

发布组件到 npm， 需要进行以下几步：

1. 注册 [npm](https://www.npmjs.com/) 。
2. 在项目目录下，执行 npm adduser， 随后根据提示，输入用户信息。显示 `Logged in as YourUserName on https://registry.npmjs.org/` 后，即可进行发布。
3. 执行 npm publish ，发布包到 npm 。

之后，我们就可以通过包名来安装 npm 依赖了：
{% codeblock lang:javascript %}

yarn add react-native-utility

{% endcodeblock %}
