---
title: Introducing Create React Native App
author: Adam Perry
authorTitle: Software Engineer at Expo
authorURL: 'https://github.com/dikaiosune'
authorImageURL: 'https://avatars2.githubusercontent.com/u/6812281'
authorTwitter: dika10sune
tags: [engineering]
youtubeVideoId: 9baaVjGdBqs
---

今天我們宣佈推出 [Create React Native App](https://github.com/react-community/create-react-native-app)：這款全新工具能大幅簡化 React Native 專案的初始設定！其設計靈感源自 [Create React App](https://github.com/facebookincubator/create-react-app)，是由 [Facebook](https://code.facebook.com) 與 [Expo](https://expo.io)（前身為 Exponent）共同合作的成果。

許多開發者在安裝與配置 React Native 現有的原生建置依賴項時遇到困難，尤其是 Android 環境。使用 Create React Native App 時，您無需使用 Xcode 或 Android Studio，且能在 Linux 或 Windows 系統上為 iOS 裝置進行開發。這是透過 Expo 應用實現的，該應用能直接載入並執行純 JavaScript 寫成的 CRNA 專案，無需編譯任何原生程式碼。

嘗試建立新專案（若已安裝 yarn 請替換為相應指令）：

```sh
$ npm i -g create-react-native-app
$ create-react-native-app my-project
$ cd my-project
$ npm start
```

此指令將啟動 React Native 打包工具並顯示 QR 碼。透過 [Expo 應用](https://expo.io)掃描即可載入您的 JavaScript 程式碼。終端機將轉發所有 `console.log` 呼叫。您可使用任何標準 React Native API 及 [Expo SDK](https://docs.expo.dev/versions/latest/) 功能。

## 原生程式碼怎麼辦？

許多 React Native 專案需編譯 Java 或 Objective-C/Swift 的依賴項。Expo 應用已內建相機、影片、通訊錄等 API，並捆綁了熱門函式庫如 [Airbnb 的 react-native-maps](https://docs.expo.dev/versions/latest/sdk/map-view/) 或 [Facebook 身份驗證](https://docs.expo.dev/versions/latest/sdk/facebook/)。但若您需要的原生依賴項未包含在 Expo 中，則可能需要自行配置建置環境。如同 Create React App，CRNA 也支援「彈出」(eject) 功能。

執行 `npm run eject` 可獲得與 `react-native init` 生成相似的專案結構。此時您將需要 Xcode 和/或 Android Studio，如同直接使用 `react-native init` 時的情況，透過 `react-native link` 添加函式庫將正常運作，且您能完全掌控原生程式碼的編譯流程。

## 有疑問或意見回饋？

Create React Native App 現已穩定至可供常規使用，我們非常期待聽到您的使用體驗！您可透過 [Twitter](https://twitter.com/dika10sune) 聯繫我，或在 [GitHub 儲存庫](https://github.com/react-community/create-react-native-app) 提交問題。我們非常歡迎 Pull Request！