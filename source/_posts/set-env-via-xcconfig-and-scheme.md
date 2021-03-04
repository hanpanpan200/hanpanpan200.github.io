title: 用 Xcode Configuration 和 Scheme 配置项目环境
date: 2019-02-28 14:49:33
category: iOS
---

<img src="/images/set-env-via-xcconfig-and-scheme/logo.svg" alt="Landing" />

想象一个场景，我们正在开发一款支付系统，这个支付系统同时支持有Web版和原生的iOS APP版本。
这个支付系统有三个环境：
- dev: 调用支付的开发环境接口，并不会真的扣钱。
- qa: 调用支付的测试环境接口，假的测试账户中的余额会发生变化。
- production: 调用产品环境中的支付接口，账户余额和钱会真实发生变化。

对于 Web 版的系统，我可以在浏览器中打开三个窗口，然后依次输入不同环境下的域名： `www.dev.pay-example.com`，`www.qa.pay-example.com`，`www.prod.pay-example.com` 来分别进行测试。

对于 iOS APP 呢？我们想要的其实是类似的，打开一台手机，我们可以同时安装三个APP：`PayExampleDev`，`PayExampleQA`，`PayExample` 然后可以切换APP来使用它们。

那么怎样通过一个 iOS 工程打包出三个不同的 APP 呢？方法很多，本文我们就来介绍通过 `Xcode Configuration` 和 `Scheme` 的配置方式来实现。

<!-- more -->

## 添加 [Xcode Configuration](https://help.apple.com/xcode/mac/10.1/index.html?localePath=en.lproj#/dev745c5c974)
- 新建 Group，命名为：Configuration
- 在 Configuration 中新建不同环境下的 Configuration Settings file

具体步骤如下图：
<img src="/images/set-env-via-xcconfig-and-scheme/1.gif" />

## 配置项目的 Configuration
- 在不同的 Configuration file 中，配置需要的键值对，用于在项目中引用，具体配置，请参考 [Demo源码](https://github.com/hanpanpan200/PayExample/blob/master/PayExample/Configuration/dev.xcconfig)
- Project -> Info -> Configurations
- 将 Configurations 配置为 Dev、QA、Prod，并配置对应的 Configuration File

具体步骤如下图：
<img src="/images/set-env-via-xcconfig-and-scheme/2.gif" />

## 创建  scheme
- 创建 scheme (PayExampleDev、PayExampleQA、PayExampleProd)，供不同环境使用
- `Edit Scheme -> Step -> Info -> Build Configuration`，配置对应的 Build Configuration 为 `Dev、QA、Prod`

<img src="/images/set-env-via-xcconfig-and-scheme/3.gif" />

## 在项目配置中引用 xcconfig 中定义的变量
- `PayExample target -> General -> Display Name`
- 配置 Display Name 为 `$(PE_BUNDLE_NAME)`

具体步骤如下图：
<img src="/images/set-env-via-xcconfig-and-scheme/4.gif" />

在某一个 Scheme 上执行某操作时，在此 Scheme 的此步骤上定义的 BuildConfig 就会生效。

比如:
用于打包 QA 环境的scheme `PayExampleQA`，其 Build 配置的 `Build Configuration` 为 `QA`
`QA` 在 `Configuration` 中配置的 `config` 文件为：`qa.xcconfig` 
项目中，APP 的 Display Name 的值为：`$(PE_BUNDLE_NAME)`
`qa.xcconfig` 中 `PE_BUNDLE_NAME` 的值为 `PayExampleQA`

如此 PayExampleQA scheme 在打包后，显示的APP名称为 `PayExampleQA`

## Setup bundle identifier

iOS 中是通过 `bundle identifier` 来标识不同的应用的。也就是说，想要通过一份代码同时编译出三个应用安装到设备中，我们需要在 `dev.xcconfig`, `qa.xcconfig`, `prod.xcconfig` 中分别定义 `BUNDLE_IDENTIFIER` ，并在项目配置中引用它。

当我们直接在配置中定义 `PE_BUNDLE_ID` ，并在项目配置中引用它时(Xcode10.1 (10B61))，会发现配置无效（如下图）

<img src="/images/set-env-via-xcconfig-and-scheme/1.png" />

当查看 `info.plist` 时，可以发现 `Bundle identifier` 的 value 是 `$(PRODUCT_BUNDLE_IDENTIFIER)`。 我们可以在 `Project -> Build Settings -> Product Bundle Identifier` 中配置它的值为 `$(PE_BUNDLE_ID)`。具体操作如下图：

<img src="/images/set-env-via-xcconfig-and-scheme/5.gif" />

## 最终效果

完成上述配置以后，我们可以在不同 Scheme 下执行 run 命令将 APP 安装到模拟器中，查看效果：
<img src="/images/set-env-via-xcconfig-and-scheme/6.gif" />

## What's More

如果你需要在 Jenkins 或者 Travis 中搭建 pipeline，此配置同样适用。

当我们用脚本来打包 APP 时，会执行如下命令：
```
xcodebuild archive -workspace 项目名称.xcworkspace 
                       -scheme scheme名称 
                       -archivePath archive包存储路径 
                       CODE_SIGN_IDENTITY=证书 
                       PROVISIONING_PROFILE=描述文件UUID
```

通过指定不同的 scheme 即可管理不同环境下的项目配置。

构建 pipeline 将会在后续文章中详细介绍，敬请期待！