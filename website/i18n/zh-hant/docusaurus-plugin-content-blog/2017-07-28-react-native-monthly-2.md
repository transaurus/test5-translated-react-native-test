---
title: 'React Native Monthly #2'
author: Tomislav Tenodi
authorTitle: Product Manager at Shoutem
authorURL: 'https://github.com/tenodi'
authorImageURL: 'https://pbs.twimg.com/profile_images/877237660225609729/bKFDwfAq.jpg'
authorTwitter: TomislavTenodi
tags: [engineering]
---

React Native 每月會議持續進行中！本次會議我們邀請到 [Infinite Red](https://infinite.red/) 團隊參與，他們是 [Chain React 研討會](https://infinite.red/ChainReactConf) 的主辦方。由於多數與會者都在 Chain React 發表過演講，我們將會議延後一週舉行。研討會的演講影片已[上線](https://www.youtube.com/playlist?list=PLFHvL21g9bk3RxJ1Ut5nR_uTZFVOxu522)，歡迎大家觀看。現在來看看各團隊的進展吧。

## 與會團隊

第二次會議共有 9 個團隊參與：

- [Airbnb](https://github.com/airbnb)
- [Callstack](https://github.com/callstack-io)
- [Expo](https://github.com/expo)
- [Facebook](https://github.com/facebook)
- [GeekyAnts](https://github.com/GeekyAnts)
- [Infinite Red](https://github.com/infinitered)
- [Microsoft](https://github.com/microsoft)
- [Shoutem](https://github.com/shoutem)
- [Wix](https://github.com/wix)

## 會議記錄

以下是各團隊的分享內容：

### Airbnb

- 歡迎查看 [Airbnb 代碼庫](https://github.com/airbnb) 中的 React Native 相關專案。

### Callstack

- [Mike Grabowski](https://github.com/grabbou) has been managing React Native's monthly releases as always, including a few betas that were pushed out. In particular, working on getting a v0.43.5 build published to npm since it unblocks Windows users!
- Slow but consistent work is happening on [Haul](https://github.com/callstack-io/haul). There is a pull request that adds HMR, and other improvements have shipped. Recently got a few industry leaders to adopt it. Possibly planning to start a full-time paid work in that area.
- [Michał Pierzchała](https://twitter.com/thymikee) from the [Jest](https://github.com/facebook/jest) team has joined us at Callstack this month. He will help maintain [Haul](https://github.com/callstack-io/haul) and possibly work on [Metro Bundler](https://github.com/facebook/metro) and [Jest](https://github.com/facebook/jest).
- [Satyajit Sahoo](https://twitter.com/satya164) is now with us, yay!
- Got a bunch of cool stuff coming up from our OSS department. In particular, working on bringing Material Palette API to React Native. Planning to finally release our native iOS kit which is aimed to provide 1:1 look & feel of native components.

### Expo

- Recently launched [Native Directory](https://native.directory) to help with discoverability and evaluation of libraries in React Native ecosystem. The problem: lots of libraries, hard to test, need to manually apply heuristics and not immediately obvious which ones are just the best ones that you should use. It's also hard to know if something is compatible with CRNA/Expo. So Native Directory tries to solve these problems. Check it out and [add your library](https://github.com/react-community/native-directory) to it. The list of libraries is in [here](https://github.com/react-community/native-directory/blob/master/react-native-libraries.json). This is just our first pass of it, and we want this to be owned and run by the community, not just Expo folks. So please pitch in if you think this is valuable and want to make it better!
- Added initial support for installing npm packages in [Snack](https://snack.expo.io/) with Expo SDK 19. Let us know if you run into any issues with it, we are still working through some bugs. Along with Native Directory, this should make it easy to test libraries that have only JS dependencies, or dependencies included in [Expo SDK](https://github.com/expo/expo-sdk). Try it out:
  - [react-native-modal](https://snack.expo.io/ByBCD_2r-)
  - [react-native-animatable](https://snack.expo.io/SJfJguhrW)
  - [react-native-calendars](https://snack.expo.io/HkoXUdhr-)
- [Released Expo SDK19](https://blog.expo.io/expo-sdk-v19-0-0-is-now-available-821a62b58d3d) with a bunch of improvements across the board, and we're now using the [updated Android JSC](https://github.com/SoftwareMansion/jsc-android-buildscripts).
- Working on a guide in docs with [Alexander Kotliarskyi](https://github.com/frantic) with a list of tips on how to improve the user experience of your app. Please join in and add to the list or help write some of it!
  - Issue: [#14979](https://github.com/facebook/react-native/issues/14979)
  - Initial pull request: [#14993](https://github.com/facebook/react-native/pull/14993)
- Continuing to work on: audio/video, camera, gestures (with Software Mansion, `react-native-gesture-handler`), GL camera integration and hoping to land some of these for the first time in SDK20 (August), and significant improvements to others by then as well. We're just getting started on building infrastructure into the Expo client for background work (geolocation, audio, handling notifications, etc.).
- [Adam Miskiewicz](https://twitter.com/skevy) has made some nice progress on imitating the transitions from [UINavigationController](https://developer.apple.com/documentation/uikit/uinavigationcontroller) in [react-navigation](https://github.com/react-community/react-navigation). Check out an earlier version of it in [his tweet](https://twitter.com/skevy/status/884932473070735361) - release coming with it soon. Also check out `MaskedViewIOS` which he [upstreamed](https://github.com/facebook/react-native/commit/8ea6cea39a3db6171dd74838a6eea4631cf42bba). If you have the skills and desire to implement `MaskedView` for Android that would be awesome!

### Facebook

- Facebook 內部正在探索如何將原生的 [ComponentKit](https://componentkit.org/) 和 [Litho](https://fblitho.com/) 元件嵌入到 React Native 中。
- 非常歡迎對 React Native 做出貢獻！如果您想知道如何貢獻，["如何貢獻"指南](https://github.com/facebook/react-native-website/blob/master/CONTRIBUTING.md) 描述了我們的開發流程，並列出了發送第一個 pull request 的步驟。還有其他不需要編寫代碼的貢獻方式，例如處理問題或更新文檔。
  - 在撰寫本文時，React Native 有 **635** 個 [未解決的問題](https://github.com/facebook/react-native/issues) 和 **249** 個 [未合併的 pull requests](https://github.com/facebook/react-native/pulls)。這對我們的維護者來說是一個巨大的負擔，而且當問題在內部修復時，很難確保相關任務得到更新。
  - 我們不確定處理這個問題的最佳方法是什麼，同時又能讓社區滿意。一些（但不是全部！）選項包括關閉過時的問題、授予更多人管理問題的權限，以及自動關閉未遵循問題模板的問題。我們寫了一份「維護者期望」指南來設定期望並避免意外。如果您有想法可以讓這個過程對維護者更好，同時確保提出問題和 pull request 的人感到被傾聽和重視，請告訴我們！

### GeekyAnts

- 我們在 Chain React 上展示了與 React Native 文件配合使用的設計師工具。許多與會者報名了等待名單。
- 我們也在研究其他跨平台解決方案，如 [Google Flutter](https://flutter.io/)（即將進行重大比較）、[Kotlin Native](https://github.com/JetBrains/kotlin-native) 和 [Apache Weex](https://weex.incubator.apache.org/)，以了解架構差異以及我們可以從中學到什麼來提升 React Native 的整體性能。
- 我們已將大多數應用切換到 [react-navigation](https://github.com/react-community/react-navigation)，這提高了整體性能。
- 同時，我們宣布了 [NativeBase Market](https://market.nativebase.io/) —— 一個為開發者打造的 React Native 元件和應用市場。

### Infinite Red

- 我們想介紹 [Reactotron](https://github.com/infinitered/reactotron)。請查看 [介紹視頻](https://www.youtube.com/watch?v=tPBRfxswDjA)。我們將很快添加更多功能！
- 舉辦了 Chain React 會議。非常成功，感謝大家的參與！[視頻現已上線！](https://www.youtube.com/playlist?list=PLFHvL21g9bk3RxJ1Ut5nR_uTZFVOxu522)

### Microsoft

- [CodePush](https://github.com/Microsoft/code-push) 現已整合到 [Mobile Center](https://mobile.azure.com/)。現有用戶的工作流程不會有任何變化。
  - 一些用戶報告了重複應用的問題 —— 他們已經在 Mobile Center 上有一個應用。我們正在解決這些問題，但如果您有兩個應用，請告訴我們，我們可以為您合併它們。
- Mobile Center 現在支持 CodePush 的推送通知。我們還展示了如何結合通知和 CodePush 進行應用的 A/B 測試 —— 這是 React Native 架構獨有的功能。
- [VS Code](https://github.com/Microsoft/vscode) 有一個已知的 React Native 調試問題 —— 幾天後發布的下一個擴展版本將修復這個問題。
- 由於 Microsoft 內部還有許多其他團隊也在使用 React Native，我們將努力在下一次會議中讓所有團隊有更好的代表性。

### Shoutem

- 已完成在 [Shoutem](https://shoutem.github.io/) 上簡化 React Native 開發流程的工作。現在開發 Shoutem 應用時可使用所有標準的 `react-native` 指令。
- 我們投入大量精力研究 React Native 效能分析的最佳實踐。現有[效能文件](/docs/performance)多已過時，我們將盡快向官方文件提交 pull request，或至少將研究結論發表於部落格文章。
- 正將導航方案切換至 [react-navigation](https://github.com/react-community/react-navigation)，後續將提供使用反饋。
- 在工具包中發布了[新版 HTML 組件](https://github.com/shoutem/ui/tree/develop/html)，可將原始 HTML 轉換為 React Native 組件樹。

### Wix

- We started working on a pull request to [Metro Bundler](https://github.com/facebook/metro) with [react-native-repackager](https://github.com/wix/react-native-repackager) capabilities. We updated react-native-repackager to support RN 44 (which we use in production). We are using it for our mocking infrastructure for [detox](https://github.com/wix/detox).
- We have been covering the Wix app in detox tests for the last three weeks. It's an amazing learning experience of how to reduce manual QA in an app of this scale (over 40 engineers). We have resolved several issues with detox as a result, a new version was just published. I am happy to report that we are living up to the "zero flakiness policy" and the tests are passing consistently so far.
- Detox for Android is moving forward nicely. We are getting significant help from the community. We are expecting an initial version in about two weeks.
- [DetoxInstruments](https://github.com/wix/detoxinstruments), our performance testing tool, is getting a little bigger than we originally intended. We are now planning to turn it into a standalone tool that will not be tightly coupled to detox. It will allow investigating the performance of iOS apps in general. It will also be integrated with detox so we can run automated tests on performance metrics.

## 下次會議

下次會議訂於 2017 年 8 月 16 日舉行。由於這僅是第二次會議，我們想了解這些紀錄對 React Native 社群的幫助程度。若有改進會議產出的建議，歡迎透過 [Twitter](https://twitter.com/TomislavTenodi) 聯繫我。