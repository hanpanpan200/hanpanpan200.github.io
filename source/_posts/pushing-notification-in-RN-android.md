title: 基于 react-native-fcm 的消息推送 - Android（二）
date: 2017-11-21 21:14:21
category: ReactNative
tags: PushNotification
---

<img src="/images/pushing-notification-in-RN/android-landing.jpeg" alt="landing" />

在上篇文章中，我们提到了 [怎样在 iOS 中集成 react-natiave-fcm](http://hanpanpan200.github.io/2017/11/20/pushing-notification-in-RN-iOS)，今天我们主要讨论 Android 中的配置和注意事项。

<!-- more -->

## Firebase 中 Android App 的配置

在 Android 平台中配置 Pushing Notification 远比 Apple 简单，并不需要在 developer 账号中做特殊的配置。
想要调用 FCM 服务向 Android 设备推送消息，我们需要在 Firebase 中配置对应的 Android App。步骤如下：

1. 在 [Firebase Console](https://console.firebase.google.com/u/0/) 中创建项目（如项目已创建，可省略此步）。
2. 进入 1 创建的项目，然后点击 `Add Firebase to your Android app`。
3. 填写 `android package name`（务必与你 Android 工程中的 applicationId 保持一致，查找时，在 `android/app/build.gradle` 文件中搜索 `applicationId`）。
4. 点击 REGISTER APP 按钮。
5. 在当前页面中，点击按钮 `Donwload google-services.json`，将其下载到本地，并根据说明，将此文件拖动至 `android/app`。
6. 在接下来的页面中，点击下一步，直至完成。

完成上述步骤之后，可以到 [https://console.firebase.google.com/u/0/project/{your-app-id}/notification]() 的 dashboard 中尝试手动发送推送消息来检验配置是否正确。

通过这一步，我们完成了 Firebase 中 Android 项目的配置，FCM 已经可以将消息推送到 Android 设备了。

## Android 工程配置

在上一步中，我们已经可以在 Firebase console 中手动推送消息了，那么当消息发送出来之后，Android App 怎样才能接收推送的消息呢？我们还需要以下几步：

- 我们需要根据 [react-native-fcm](https://github.com/evollu/react-native-fcm#android-configuration) 的文档，完成 Android 项目中的配置。

- 根据 [上篇文章](http://hanpanpan200.github.io/2017/11/20/pushing-notification-in-RN-iOS) ，在组件中注册 FCM 服务。
- 在 Firebase Console 中手动推送消息。

你以为就这样 happy ending 了吗？
事实是：你完成了 Android 中的配置，欢欣雀跃地从 Firebase Console 手动推送了一条消息。但是消息来了之后，你发现有什么不对劲。。嗯，为什么通知栏里 notification 的图标变成了白色小方块？

## 白色小方块的 notification icon 问题

原因是因为 Android 5.0 之后，google 统一了 notification icon 的风格，icon 的颜色只能用透明色和白色，如果你的 icon 中有彩色，那么系统就会认为此图标为一个无效的图标，并用一个白色小方块替代你的 notification icon。

解决这个问题的 [workaround](https://stackoverflow.com/questions/28387602/notification-bar-icon-turns-white-in-android-5-lollipop/28387744#28387744) 有很多，思路有：

- 根据 `targetSdk` 动态设置 icon 的颜色（需要在 Android 原生项目中，用 Java 处理）
- 在 `build.gradle` 中设置 `targetSdkVersion` 为 20

除非项目有特殊需求必须使用彩色的 notification icon，不然我比较推荐的做法是遵循 google 的设计规范，替换一个只有白色和透明色的 notification icon。祝你好运，希望你不会是有特殊需求的那一个。

至此，我们的 Android 设备也可以成功的接收到 Firebase Console 推送来的消息啦！

## 小结：

到目前为止，我们的消息都是从 Firebase Console 手动推送的，那么问题来了，怎样把 FCM 集成到 App 的后端服务器呢？这个过程中会包含 FCM、App Backend 还有 App 三方的通讯了，应该怎样去集成呢？

针对这些问题，我将会的下篇文章中为大家做介绍，敬请期待。

