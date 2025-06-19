---
title: The GAAD Pledge - One Year Later
authors: [alexmarlette]
tags: [announcement]
---

自Facebook作出[GAAD承諾](https://diamond.la/GAADPledge/)要使React Native具備無障礙功能已過一年，此專案的進展已超出我們的預期。我們興奮地宣布這項專案將持續至2021年，並想向各位報告截至目前為止的進展。在去年對React Native的無障礙缺口進行全面分析後，我們便開始著手填補這些缺口。

我們最初有90個待解決的缺口分析議題，從2021年3月專案在GitHub上啟動至今：

- 社群已關閉11個議題

- React Native團隊評估並關閉了19個議題

- 合併了9個拉取請求

- 合併了1個針對React Native文件的拉取請求

我們要感謝React Native社群在過去一年中為實現更具無障礙性的React Native所做出的重大貢獻。每一位貢獻者的努力都在推動React Native無障礙功能的改進中發揮了作用。

<!--truncate-->

## 修復項目

通過9個拉取請求，我們已在多個組件中修復了兩類問題，並為API新增了一項功能。

- 已在七個組件中解決了「禁用狀態」問題

- 在兩個組件中解決了「選中狀態」問題

- 為React Native API新增了查詢AccessibilityManager.getRecommendedTimeoutMillis()的功能

### 禁用狀態宣告與禁用功能

在缺口分析中發現最普遍的問題之一是某些組件不會宣告或禁用功能。現在有七個組件會宣告其禁用狀態或禁用點擊功能。

宣告禁用時機

- `Button` - [#31001](https://github.com/facebook/react-native/pull/31001)

- `Images` - [#31252](https://github.com/facebook/react-native/pull/31252)

- `ImageBackground` - [#31252](https://github.com/facebook/react-native/pull/31252)

當組件具有禁用屬性時，會禁用點擊功能

- `Button` - [#31001](https://github.com/facebook/react-native/pull/31001)

- `Text` - [React Native Team commit](https://github.com/facebook/react-native/commit/33ff4445dcf858cd5e6ba899163fd2a76774b641)

- `Pressable` - [React Native Team commit](https://github.com/facebook/react-native/commit/1c7d9c8046099eab8db4a460bedc0b2c07ed06df)

- `TouchableHighlight` - [#31135](https://github.com/facebook/react-native/pull/31135)

- `TouchableOpacity` - [#31108](https://github.com/facebook/react-native/pull/31108)

- `TouchableNativeFeedback` - [#31224](https://github.com/facebook/react-native/pull/31224)

- `TouchableWithoutFeedback` - [#31297](https://github.com/facebook/react-native/pull/31297)

### 選中狀態宣告

有些組件在獲得焦點時不會宣告其選中狀態。現在當組件獲得焦點且AccessibilityState設為選中，或組件變更為選中狀態時，此行為已獲得修正。

宣告選中時機

- `Button` - [#31001](https://github.com/facebook/react-native/pull/31001)

- `TextInput` - [#31144](https://github.com/facebook/react-native/pull/31144)

### 無障礙逾時設定

先前在Android上無法查詢無障礙逾時設定。此修復新增了查詢`AccessibilityManager.getRecommendedTimeoutMillis()`的功能，可查詢UI元素自動關閉或自動進展前的「採取行動時間」。

## 文件新增內容

React Native 的文件必須更新以反映每個新增或變更的 API。[React Native 文件的新增內容](https://reactnative.dev/docs/next/accessibilityinfo#getrecommendedtimeoutmillis-android)涵蓋了 `getRecommendedTimeoutMillis()` 在 AccessibilityInfo 中的新增。

## 社群參與

我們要感謝以下所有提交並合併 pull request 的貢獻者，以及審查和評論問題的人。

### 已合併的 Pull Requests

- [@huzaifaaak](https://twitter.com/huzaifaaak) 關閉了 3 個問題：
  - [為按鈕無障礙功能新增 talkback 支援：disabled prop #31001](https://github.com/facebook/react-native/pull/31001)
  - [無障礙/按鈕測試 #31189](https://github.com/facebook/react-native/pull/31189)
- [@natural_clar](https://twitter.com/natural_clar) 關閉了 1 個問題：
  - [功能：當 `TouchableHighlight` 被禁用時設定無障礙狀態為禁用 #31135](https://github.com/facebook/react-native/pull/31135)
- [fabriziobertoglio1987](https://github.com/fabriziobertoglio1987) 關閉了 2 個問題：
  - [[Android] 當 `TextInput` 元件被選中時未宣告選中狀態 #31144](https://github.com/facebook/react-native/pull/31144)
  - [無障礙修復：圖片未宣告「禁用」 #31252](https://github.com/facebook/react-native/pull/31252)
- [@kyamashiro73](https://twitter.com/kyamashiro73) 關閉了 1 個問題：
  - [為 `TouchableNativeFeedback` 無障礙功能新增 talkback 支援：disabled prop #31224](https://github.com/facebook/react-native/pull/31224)
- [@grgr-dkrk](https://twitter.com/dkrk0901) 關閉了 1 個問題並新增至 React Native 文件：
  - [在 AccessibilityInfo 中新增 `getRecommendedTimeoutMillis` #31063](https://github.com/facebook/react-native/pull/31063)
  - [功能：在 accessibilityInfo 中新增 `getRecommendedTimeoutMillis` 章節 #2581](https://github.com/facebook/react-native-website/pull/2581)
- [@crloscuesta](https://twitter.com/crloscuesta) 關閉了 1 個問題：
  - [當 `TouchableWithoutFeedback` 被禁用時禁用無障礙狀態 #31297](https://github.com/facebook/react-native/pull/31297)
- [@chakrihacker](https://twitter.com/chakrihacker) 關閉了 1 個問題：
  - [當無障礙禁用設定時禁用 `TouchableOpacity` #31108](https://github.com/facebook/react-native/pull/31108)

感謝以其他方式貢獻時間的社群成員！

[Simek](https://github.com/Simek)、[saurabhkacholiya](https://github.com/saurabhkacholiya)、[meehawk](https://github.com/meehawk)、[intergalacticspacehighway](https://github.com/intergalacticspacehighway)、[chrisglein](https://github.com/chrisglein)、[jychiao](https://github.com/jychiao) 和 [Waltari10](https://github.com/Waltari10)

## 參與進來！

我們已經取得了很大進展，但尚未完成。我們需要您的支援來達成目標。Facebook 的 React Native 團隊承諾支援處理無障礙問題的貢獻者。他們將繼續回應無障礙問題的評論並審查 pull request。React Native 團隊也在處理一些最棘手的無障礙問題，包括將 accessibilityRoles 正確翻譯成其他語言以及為特定元件指定錯誤文字。

加入我們一起解決剩下的問題。[改進 React Native 無障礙功能的專案看板](https://github.com/facebook/react-native/projects/15)上仍有未解決的無障礙問題。[已選中/未選中狀態](https://github.com/facebook/react-native/issues/30843)、[集合的進入/退出](https://github.com/facebook/react-native/issues/30861) 和 [集合中的位置](https://github.com/facebook/react-native/issues/30977) 等問題是現有和新貢獻者為 React Native 無障礙功能做出貢獻的絕佳機會。

### 了解更多

閱讀關於如何在 [Facebook 技術部落格](https://tech.fb.com/react-native-accessibility/) 上進行差距分析，或關於在 [React Native 部落格](https://reactnative.dev/blog/2021/03/08/GAAD-React-Native-Accessibility) 上發布 GitHub 問題的資訊。