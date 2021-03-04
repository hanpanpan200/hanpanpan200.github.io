title: React Native 在 Android 中下载文件并分享
date: 2017-02-08 13:28:10
category: ReactNative
tags: 开源项目
---

<img src="/images/react-native-android-share/landing.jpg" alt="Landing" width="80%"/>

有这样一个应用场景：

在一个 Android 的 App 中，我们将一个文件下载到 App 中以后，通过点击一个分享按钮，将这个文件分享到本机已经安装了的其他 App 中。

这个场景涉及到`下载`和`分享`两个步骤。

<!-- more -->

使用 React Native 开发原生的 Android App 时，相信有很多开发者跟我一样并不太熟悉原生 Android App 开发。所以我们要去调查同样的 feature 在原生 Android App 中需要怎样处理，然后用 React Native 的方式去实现它。

那么在 Android 中，App 一般会将下载的文件存放在什么路径呢？

下面我们就来解答这个问题。

## 把从App中下载的文件存在哪里？

首先第一个问题：在 Android 中，App 一般会将下载的文件存放在什么路径？

为了解决这个问题，首先需要了解 Android Storage 。 Android 中有 Internal Storage 与 External Storage 。

从这篇[博客](http://blog.imallen.wang/blog/2015/09/24/internal-vs-external-storage/)中我们了解到：

>Internal storage 和 private external storage 对于 app 来说都是 private 的，即其他 app 无权访问。

>App的 Internal storage 位于 `/data/data/packagename`，而 External storage 则位于 `/storagge/emulated/0/`下，其中 public external storage 的位置一般在`/storage/emulated/0/dirname`下，private external storage 则位于`/storage/emulated/0/Android/data/`下。

它们的使用场景如下：

>Internal storage 用来保存不愿被其他 app 访问而且占用空间较小的信息，卸载时会被删除，比如数据库，设置信息，cache 等。

>Private external storage 用来保存不愿被其他 app 访问而且占用空间较大的信息，卸载时会被删除，比如下载的图片或者多媒体信息，cache 等。

> Public external storage 则用来保存可被其他 app 访问的信息，如歌曲等。


于是我们可以知道，在 App 中下载一个文件，且可以在其他 App 中分享（数据被其他 App 访问），我们需要将文件存放在 `Public external storage` 中。而且我们可以通过方法

```
getExternalStorageDirectory()
```
来获取 `Public external storage` 的根目录。

那么，我们应该需要在 `Public external storage` 的根目录下为这个 App 创建一个单独的文件夹来存储数据吧？我们可以参考一下其他 App 是怎样做的，来验证这个想法。

打开 Android 设备的文件系统的主页，可以看到两个重要的目录: `Main Storage` 和 `System`。

<img src="/images/react-native-android-share/home.jpg" alt="File-System-Home" width="40%"/>

然后打开 `Main Storage` 文件夹可以看到这里显示了手机上很多 App 的目录。比如说，当我们打开 `netease` 就可以看到我们在`网易云音乐`中下载的一些音乐或者MV文件。
于是我们可以知道，我们当前需要的做的，就是在 Public external storage 的根目录下，创建一个专属于自己App的文件夹，用来存放下载的文件。

<img src="/images/react-native-android-share/MainStorage.jpg" alt="Main-Storage" width="40%"/>

## 用React Native的方式来实现在Android中下载并分享

在 React Native 中，我们可以用 [react-native-fetch-blob](https://github.com/wkh237/react-native-fetch-blob)来处理下载，然后用 [react-native-share](https://github.com/EstebanFuentealba/react-native-share)来处理分享。

Example代码地址： [RN-android-download-share](https://github.com/hanpanpan200/RN-android-download-share)。

Example的运行结果：

1、 下载文件
<img src="/images/react-native-android-share/download.gif" alt="download" width="40%"/>

下载成功后，打开文件系统，在 `Main Storage` 目录下查看：此时已经创建了文件夹 `RNAndroidDownloadShare`，并且已将文件`Dropbox.pdf`保存在这个文件夹内。

<img src="/images/react-native-android-share/download-result1.jpg" alt="download-result1" width="40%"/>
<img src="/images/react-native-android-share/download-result2.jpg" alt="download-result2" width="40%"/>

2、 通过Email分享文件

<img src="/images/react-native-android-share/share.jpg" alt="share" width="40%"/>
<img src="/images/react-native-android-share/share-via-email.jpg" alt="share-via-email" width="40%"/>

## Tips

如果不将文件放在 public external storage的文件目录内的话，即使文件下载后已经存在了设备中，在Android中仍然是无法通过写代码的方式将它分享出去的。有兴趣的同学可以亲自试试这种情况下的效果。
