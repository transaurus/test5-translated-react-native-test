---
id: environment-setup
title: Get Started with React Native
hide_table_of_contents: true
---

import PlatformSupport from '@site/src/theme/PlatformSupport';
import BoxLink from '@site/src/theme/BoxLink';

**React Native 讓熟悉 React 的開發者能夠打造原生應用程式。** 同時，原生開發者也能透過 React Native 實現跨平台功能一致性，只需編寫一次共用的功能程式碼。

我們認為體驗 React Native 的最佳方式是透過**框架**——這是一個包含所有必要 API 的工具箱，能協助您建構符合生產環境標準的應用程式。

您也可以不使用框架來開發 React Native 應用，但我們發現多數開發者會受益於採用如 [Expo](https://expo.dev) 這類 React Native 框架。Expo 提供基於檔案的路由系統、高品質的通用函式庫，以及無需直接管理原生檔案就能修改原生程式碼的外掛機制。

<details>
<summary>Can I use React Native without a Framework?</summary>

Yes. You can use React Native without a Framework. **However, if you’re building a new app with React Native, we recommend using a Framework.**

In short, you’ll be able to spend time writing your app instead of writing an entire Framework yourself in addition to your app.

The React Native community has spent years refining approaches to navigation, accessing native APIs, dealing with native dependencies, and more. Most apps need these core features. A React Native Framework provides them from the start of your app.

Without a Framework, you’ll either have to write your own solutions to implement core features, or you’ll have to piece together a collection of pre-existing libraries to create a skeleton of a Framework. This takes real work, both when starting your app, then later when maintaining it.

If your app has unusual constraints that are not served well by a Framework, or you prefer to solve these problems yourself, you can make a React Native app without a Framework using Android Studio, Xcode. If you’re interested in this path, learn how to [set up your environment](set-up-your-environment) and how to [get started without a framework](getting-started-without-a-framework).

</details>

## 使用 Expo 建立新 React Native 專案

<PlatformSupport platforms={['android', 'ios', 'tv', 'web']} />

Expo 是企業級 React Native 框架，提供簡化開發流程的工具組，包含檔案式路由系統、標準化原生模組庫等進階功能。

Expo 框架為免費開源專案，在 [GitHub](https://github.com/expo) 與 [Discord](https://chat.expo.dev) 擁有活躍社群。Expo 團隊與 Meta 的 React Native 核心團隊密切合作，確保 Expo SDK 能第一時間支援最新 React Native 功能。

Expo 團隊另提供 Expo Application Services (EAS) 選配服務，在應用程式開發流程的每個階段為 Expo 框架提供補充工具。

請在終端機執行以下指令建立新 Expo 專案：

```shell
npx create-expo-app@latest
```

專案建立完成後，請參閱 Expo 入門指南的其他章節開始開發您的應用程式。

<BoxLink href="https://docs.expo.dev/get-started/set-up-your-environment">Continue with Expo</BoxLink>