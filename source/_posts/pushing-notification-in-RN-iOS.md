title: 基于 react-native-fcm 的消息推送 - iOS（一）
date: 2017-11-20 21:14:21
category: ReactNative
tags: PushNotification
---

<img src="/images/pushing-notification-in-RN/landing.jpg" alt="landing" />

消息推送在 App 中很常用，那么在 React Native 中怎样来集成消息推送呢？Google 推出的 [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/?hl=zh-cn) 是用来做消息推送的，不过在中国国内使用时需要安装 Google Service 。

阅读本文时，你需要知道以下几个概念：
1. [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/?hl=zh-cn)：Firebase 的消息推送服务，是 [GCM](https://developers.google.com/cloud-messaging/) 的 升级版，是 Android 消息推送的新方案，并且其在背后集成了 Apple 的 APNs 服务, 因此开发者可以使用它提供的接口向 iOS 和 Android 设备推送消息。
2. [APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1): Apple 的 APNs 服务是消息推送功能的核心，开发者可以使用它提供的一系列接口向 iOS App 推送消息。

[react-native-fcm](https://github.com/evollu/react-native-fcm) 是 [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/?hl=zh-cn) 的 React Native 插件，将其集成到 React Native 项目中时，需要完成 iOS 和 Android 项目中的消息推送配置后，才可成功向对应的平台成功推送消息。本文属于 [基于 react-native-fcm 的消息推送](http://hanpanpan200.github.io/tags/PushNotification/) 系列文档的第一篇，主要介绍基于 react-native-fcm 的 iOS 消息推送配置。

<!-- more -->

## Apple 配置
FCM 需要调用 APNs 的 API 从而向 iOS App 推送服务。那么 FCM 与 APNs 是怎样沟通的呢？APNs 提供了[两种连接](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW1)方式：Provider Authentication Tokens 和 APNs Provider Certificates。

下面对这两种方式做下简单对比。

### APNs Provider Certificates

配置方式：
- 进入[Certificates, Identifiers & Profiles](https://developer.apple.com/account/ios/certificate/) 页面。
- 选中 AppID -> 选中对应的 AppID -> Edit -> Push notifications
<img src="/images/pushing-notification-in-RN/certificate.png" alt="certificate" />
- Create Certificate...(development or production)
<img src="/images/pushing-notification-in-RN/create-certificate.png" alt="create-certificate" />
- Add certificate to App ID


### Provider Authentication Tokens

配置方式：
- 进入[Certificates, Identifiers & Profiles](https://developer.apple.com/account/ios/certificate/) 页面。
- 选中 Keys
- Create new key -> 输入 name 并选中 APNs -> 下一步
<img src="/images/pushing-notification-in-RN/create-key.png" alt="create-key" />
- 下载 Key file 并将其保存在安全的地方（注意，此文件只提供一次下载机会，请妥善保存）

### APNs Provider Certificates 与 Provider Authentication Tokens 的对比

**APNs Provider Certificates 的方式：**

1. 一经配置，有效期为一年，过期之后，需重新配置。
2. 每个 App 独立配置，可为一个 App 配置不同环境下的 certificate，例如 development, production 等。
3. 在 Apple 中配置好后，需将 certificate 导出，分别上传至 Firebase 供推送消息使用。
4. 在 Apple 中配置好后，需将 certificate 上传至 [CI/CD](https://en.wikipedia.org/wiki/Continuous_integration) 服务器供打包使用。

**Provider Authentication Tokens 的方式：**
>Use the Apple Push Notification service for your notification requests. One key is used for all of your apps.

1. APNs auth key 一经配置，可供当前开发者账号下所有的App使用。
2. 在 Apple 中配置好之后，只需上传至 Firebase 即可使用，在 [CI/CD](https://en.wikipedia.org/wiki/Continuous_integration) 服务器中，不需额外配置。

鉴于 `APNs auth key` 的方式较为简单，且是目前 Firebase 所推荐的做法。因此我们项目中也采取的此种方式。

通过这一步，我们拿到了 Firebase 和 APNs 安全通信的 `APNs Authentication key` 。

## Firebase 中 iOS App 的配置

1. 在 [Firebase Console](https://console.firebase.google.com/u/0/) 中创建项目。
2. 进入 1 创建的项目，然后点击 `Add Firebase to your iOS app`。
3. 填写 `iOS bundle ID`（务必与你 iOS 工程中的 bundle id 保持一致）。
4. 点击 REGISTER APP 按钮。
5. 在当前页面中，点击按钮 `Donwload GoogleService-Info.plist`，将其下载到本地，并根据说明，将此文件拖动至Xcode的 iOS 项目中的指定位置。
6. 在接下来的页面中，点击下一步，直至完成。
7. 在当前项目中，选中你刚创建的 App -> Settings
<img src="/images/pushing-notification-in-RN/settings.png" alt="settings" />
1. 选中 CLOUD MESSAGING 后，进入对应的 iOS App， 上传你的 **APNs Authentication Key**
<img src="/images/pushing-notification-in-RN/auth-key.png" alt="auth-key" />
至此，你已经在 Firebase 中完成了 iOS 项目的配置，可以到 [https://console.firebase.google.com/u/0/project/{your-app-id}/notification]() 的 dashboard 中尝试手动发送推送消息来检验配置是否正确。

通过这一步，我们完成了 Firebase 中 iOS 项目的配置，FCM 已经可以成功与 APNs 通信了。

## iOS 工程配置

在上一步中，我们已经可以在 Firebase console 中手动推送消息了，那么当消息发送出来之后，App 为什么还没有接受到它呢？不要着急，我们还需要在 iOS 项目中做些配置，它们主要分为以下两点：

1. 依赖安装和系统事件处理
根据 [react-native-fcm配置说明](https://github.com/evollu/react-native-fcm#ios-configuration) ，我们需要在 iOS 项目中用 cocoapods 安装 Firebase 依赖，随后在 `AppDelegate.m` 和 `AppDelegate.h` 的系统事件处理中，修改相应的事件处理逻辑。
2. Capabilities设置
每当用户在设备上打开 App 时，系统都会自动在你的 App 和 APNs 之间建立连接，此时还需要在项目中进行以下配置，你的设备才可以成功收到推送消息了。你需要在 iOS 项目 -> Capabilities 中进行以下设置：
- 开启 Push notifications （会自动生成 Entitlements 文件）
- 开启 Keychain sharing
- 在 Background mode 中，开启 Background fetch 和 remote notifications

此刻，Firebase 已经可以成功推送消息了，而你的 App 也已经具备了接收消息的能力。

## 在组件中注册 FCM 服务

此时，FCM 服务可以通过调用 APNs 推送消息。而我们的 App 也具备了接收消息的能力。那么怎样才能从 Firebase Console 中推送一条消息给当前的 iOS App 呢？换言之，怎样将当前设备注册到 FCM 服务呢？是的，我们还需要一个注册设备的过程。

react-native-fcm 封装了 FCM 接收和推送消息处理的逻辑，并暴露了一些 JS 方法供 React Native 调用，其中包括：注册 FCM 服务，接收推送消息后的事件处理等等。例如，我们经常将 FCM 服务的初始化放在整个 App 入口处 component 的 `componentDidMount` 回调中。

```
class App extends Component {
  componentDidMount() {
    FCM.requestPermissions().then(()=>console.log('granted')).catch(()=>console.log('notification permission rejected'));
    
    FCM.getFCMToken().then(token => {
      console.log(token)
    });
  }
}
```
在组件 `componentDidMount` 通过调动 `FCM.getFCMToken()` 可以完成 FCM 服务的注册。

## 在 Firebase Console 中手动推送消息

FCM 支持向安装了 App 的所有设备广播消息、向指定设备推送消息 和 向注册了某一 Topic 的 App 发送消息。在此我们主要介绍怎样在Firebase Console 手动广播消息和向指定设备推送消息。

### 广播消息
注册成功之后，你可以在 [Firebase Console](https://console.firebase.google.com/u/0/) 中广播消息到 iOS 平台的所有 App。在手动发送消息时，需要：
- 选中 Target
- 选中 User segment
- 选中你想广播消息的平台

<img src="/images/pushing-notification-in-RN/broadcast.png" alt="broadcase message" />

### 向指定设备推送消息

除了广播消息之外，我们还可以通过 fcm token 定向给某一个设备推送消息。在手动发送消息时，需要：
- 选中 Target
- 选中 Single device
- 输入你在 `FCM.getFCMToken()` 成功后获取的 token

<img src="/images/pushing-notification-in-RN/send-message-to-device.png" alt="send message to device" />

此时 我们的 iOS App 就可以收到提醒了！

那么 Android 平台呢？它需要哪些特殊配置？我将在下一篇文章中为大家详细道来。