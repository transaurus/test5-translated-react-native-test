---
id: set-up-your-environment
title: Set Up Your Environment
hide_table_of_contents: true
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import constants from '@site/core/TabsConstants';

import GuideLinuxAndroid from './\_getting-started-linux-android.md';
import GuideMacOSAndroid from './\_getting-started-macos-android.md';
import GuideWindowsAndroid from './\_getting-started-windows-android.md';
import GuideMacOSIOS from './\_getting-started-macos-ios.md';

本指南將教您如何設定開發環境，以便透過 Android Studio 和 Xcode 運行專案。這將使您能夠使用 Android 模擬器和 iOS 模擬器進行開發、在本機建置應用程式等。

:::note
This guide requires Android Studio or Xcode. If you already have one of these programs installed, you should be able to get up and running within a few minutes. If they are not installed, you should expect to spend about an hour installing and configuring them.

<details>
<summary>Is setting up my environment required?</summary>

Setting up your environment is not required if you're using a [Framework](/architecture/glossary#react-native-framework). With a React Native Framework, you don't need to setup Android Studio or XCode as a Framework will take care of building the native app for you.

If you have constraints that prevent you from using a Framework, or you'd like to write your own Framework, then setting up your local environment is a requirement. After your environment is set up, learn how to [get started without a framework](getting-started-without-a-framework).

</details>
:::

#### 開發作業系統

<Tabs groupId="os" queryString defaultValue={constants.defaultOs} values={constants.oses} className="pill-tabs">
<TabItem value="macos">

#### Target OS

<Tabs groupId="platform" queryString defaultValue={constants.defaultPlatform} values={constants.platforms} className="pill-tabs">
<TabItem value="android">

[//]: # 'macOS, Android'

<GuideMacOSAndroid/>

</TabItem>
<TabItem value="ios">

[//]: # 'macOS, iOS'

<GuideMacOSIOS/>

</TabItem>
</Tabs>

</TabItem>
<TabItem value="windows">

#### Target OS

<Tabs groupId="platform" queryString defaultValue={constants.defaultPlatform} values={constants.platforms} className="pill-tabs">
<TabItem value="android">

[//]: # 'Windows, Android'

<GuideWindowsAndroid/>

</TabItem>
<TabItem value="ios">

[//]: # 'Windows, iOS'

## Unsupported

> A Mac is required to build projects with native code for iOS. You can use [Expo Go](https://expo.dev/go) from [Expo](environment-setup#start-a-new-react-native-project-with-expo) to develop your app on your iOS device.

</TabItem>
</Tabs>

</TabItem>
<TabItem value="linux">

#### Target OS

<Tabs groupId="platform" queryString defaultValue={constants.defaultPlatform} values={constants.platforms} className="pill-tabs">
<TabItem value="android">

[//]: # 'Linux, Android'

<GuideLinuxAndroid/>

</TabItem>
<TabItem value="ios">

[//]: # 'Linux, iOS'

## Unsupported

> A Mac is required to build projects with native code for iOS. You can use [Expo Go](https://expo.dev/go) from [Expo](environment-setup#start-a-new-react-native-project-with-expo) to develop your app on your iOS device.

</TabItem>
</Tabs>

</TabItem>
</Tabs>