---
title: Introducing Button, Faster Installs with Yarn, and a Public Roadmap
authors: [hectorramos]
tags: [announcement]
---

我們收到許多反饋表示，由於 React Native 的開發進展非常快速，有時難以掌握最新動態。為了讓大家更清楚目前的工作重點，我們現在發佈了 [React Native 的路線圖](https://github.com/facebook/react-native/wiki/Roadmap)。整體而言，這些工作可分為三大優先事項：

- **核心函式庫**：為最常用的元件和 API 擴充更多功能。
- **穩定性**：改善基礎架構以減少錯誤並提升程式碼品質。
- **開發者體驗**：幫助 React Native 開發者更高效地工作。

如果您有希望加入路線圖的功能建議，請到 [Canny](https://react-native.canny.io/feature-requests) 提出新功能或討論現有提案。

## React Native 的新變化

[Version 0.37 of React Native](https://github.com/facebook/react-native/releases/tag/v0.37.0), released today, introduces a new core component to make it really easy to add a touchable Button to any app. We're also introducing support for the new [Yarn](https://yarnpkg.com/) package manager, which should speed up the whole process of updating your app's dependencies.

## 全新 Button 元件登場

我們今天推出了一個基礎的 `<Button />` 元件，在各平台都能完美呈現。這解決了我們最常收到的反饋之一：React Native 是少數沒有內建按鈕元件的行動開發工具包。

![Android 與 iOS 上的簡易按鈕](/blog/assets/button-android-ios.png)

```
<Button
  onPress={onPressMe}
  title="Press Me"
  accessibilityLabel="Learn more about this Simple Button"
/>
```

有經驗的 React Native 開發者都知道如何製作按鈕：在 iOS 上使用 TouchableOpacity 實現預設外觀，在 Android 上使用 TouchableNativeFeedback 實現漣漪效果，再套用一些樣式。自訂按鈕的建置或安裝並不特別困難，但我們的目標是讓 React Native 極易上手。透過在核心中加入基礎按鈕元件，新手開發者能在第一天就打造出令人驚豔的作品，而不必花時間調整按鈕格式或學習 Touchable 的細微差異。

Button 元件旨在各平台都能完美運作並呈現原生外觀，因此不會支援自訂按鈕的所有花俏功能。它是一個絕佳的起點，但並非用來取代您現有的所有按鈕。欲瞭解更多，請查閱 [新版 Button 文件](/docs/button)，內含可執行的範例！

## Speed up `react-native init` using Yarn

您現在可以使用 JavaScript 的新套件管理工具 [Yarn](https://yarnpkg.com/) 來大幅加速 `react-native init` 的流程。要體驗加速效果，請先 [安裝 yarn](https://yarnpkg.com/en/docs/install) 並將 `react-native-cli` 升級至 1.2.0 版：

```sh
$ npm install -g react-native-cli
```

現在建立新應用程式時，您應該會看到「使用 yarn」的提示：

![使用 yarn](/blog/assets/yarn-rncli.png)

在簡單的本地測試中，`react-native init` 在良好網路環境下僅需 **約 1 分鐘** 即可完成（使用 npm 3.10.8 時約需 3 分鐘）。安裝 yarn 是選用但強烈建議的步驟。

## 感謝您！

我們要感謝所有為此版本貢獻心力的開發者。完整 [發佈說明](https://github.com/facebook/react-native/releases/tag/v0.37.0) 已發佈於 GitHub。在修復了超過二十個錯誤並新增多項功能後，React Native 因您的付出而持續進步。