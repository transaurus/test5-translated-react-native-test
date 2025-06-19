---
title: Meet Doctor, a new React Native command
author: Lucas Bento
authorTitle: React Native Community
authorURL: 'https://twitter.com/lbentosilva'
authorImageURL: 'https://avatars3.githubusercontent.com/u/6207220?s=460&v=4'
authorTwitter: lbentosilva
tags: [announcement]
---

在 React Native 社群 6 位貢獻者提交超過 20 個拉取請求後，我們興奮地推出 `react-native doctor` 指令——這項新功能將協助您進行開發環境的初始化設定、疑難排解，並自動修復錯誤。`doctor` 指令的設計靈感主要來自 [Expo](https://expo.io/) 和 [Homebrew](https://brew.sh/) 的同名指令，並融入些許 [Jest](https://jestjs.io/) 的介面風格。

<!--truncate-->

實際運作畫面如下：

<p style={{textAlign: 'center'}}>
  <video width={700} controls="controls" autoPlay style={{borderRadius: 5}}>
    <source type="video/mp4" src="/img/homepage/DoctorCommand.mp4" />
  </video>
</p>

## 運作原理

The `doctor` command currently supports most of the software and libraries that React Native relies on, such as CocoaPods, Xcode and Android SDK. With `doctor` we'll find issues with your development environment and give you the option to automatically fix them. If `doctor` is not able to fix an issue, it will display a message and a helpful link explaining how to fix it manually as the following:

<p style={{textAlign: 'center'}}>
  <img width={700} src="/img/DoctorManualInstallationMessage.png" alt="Doctor command with a link to help on Android SDK's installation" title="Doctor command with a link to help on Android SDK's installation" />
</p>

## 立即試用

The `doctor` command is available as a part of React Native 0.62. However, you can try it without upgrading yet:

```sh
npx @react-native-community/cli doctor
```

## 目前支援的檢查項目

`doctor` 當前支援以下檢查：

- Node.js (>= 8.3)
- yarn (>= 1.10)
- npm (>= 4)
- Watchman (>= 4)，用於開發模式下監控檔案系統變更

針對 Android 環境的特殊檢查：

- Android SDK (>= 26)，Android 的軟體運行環境
- Android NDK (>= 19)，Android 原生開發工具包
- `ANDROID_HOME`，Android SDK 設定所需的環境變數

針對 iOS 環境的檢查：

- Xcode (>= 10)，用於開發、建置與發布 iOS 應用的整合開發環境
- CocoaPods，iOS 應用程式庫依賴管理工具
- ios-deploy (選用)，CLI 內部用於在實體 iOS 裝置安裝應用的函式庫

## 致謝

特別感謝 React Native 社群的貢獻者，尤其是 [@thymikee](https://github.com/thymikee)、[@thib92](https://github.com/thib92)、[@jmeistrich](https://github.com/jmeistrich)、[@tido64](https://github.com/tido64) 和 [@rickhanlonii](https://github.com/rickhanlonii)。