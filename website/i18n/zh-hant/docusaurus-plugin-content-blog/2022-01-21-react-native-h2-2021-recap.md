---
title: React Native - H2 2021 Recap
authors: [cortinico]
tags: [announcement]
---

儘管我們仍對 [React Native 0.67 版本的發佈](/blog/2022/01/19/version-067) 感到興奮，但我們想花點時間**慶祝**社群在過去半年取得的成就，並分享我們對 React Native 未來的**展望**。

<!--truncate-->

具體來說，2021 年下半年對我們和社群來說都是 [令人振奮的半年](/blog/2021/08/19/h2-2021#pushing-the-technology-forward)，我們有機會對開源生態系統投入更多。我們翻新了一些既有流程，並從零開始建立新的流程，這些將幫助您、我們和社群享受**更優質**的 React Native 體驗。

## 程式庫健康狀態

In H2 2021, we invested in tackling some of the _OSS debt_ that our repository built up over the years. Specifically, most of our focus was around **pull requests**. We built an internal process to make sure all the new pull requests are addressed in a timely manner.

雖然這並非完整清單，但我們想特別強調貢獻者提交的一些**具影響力**的 PR：

- **Accessibility**
  - [#31630](https://github.com/facebook/react-native/pull/31630) `Added Support for Entrance/exit from collection by Flatlist` by [@anaskhraza](https://github.com/anaskhraza)
- **Crash**
  - [#29452](https://github.com/facebook/react-native/pull/29452) `Fix - TextInput Drawable to avoid Null Pointer Exception RuntimeError` by [@fabriziobertoglio1987](https://github.com/fabriziobertoglio1987)
- **Display**
  - [#31777](https://github.com/facebook/react-native/pull/31777) `fix: TouchableNativeFeedback ripple starts on previous touch location` by [@intergalacticspacehighway](https://github.com/intergalacticspacehighway)
  - [#31789](https://github.com/facebook/react-native/pull/31789) `Fix support for blobs larger than 64 KB on Android` by [@tomekzaw](https://github.com/tomekzaw)
  - [#31007](https://github.com/facebook/react-native/pull/31007) `Fix selectionColor doesn't style Android TextInput selection handles` by [@fabriziobertoglio1987](https://github.com/fabriziobertoglio1987)
  - [#32398](https://github.com/facebook/react-native/pull/32398) `Fix Android border positioning regression` by [@oblador](https://github.com/oblador)
  - [#29099](https://github.com/facebook/react-native/pull/29099) `[Android] Allows to set individual (left,top,right,bottom) dotted/dashed` by [@fabriziobertoglio1987](https://github.com/fabriziobertoglio1987)
  - [#29117](https://github.com/facebook/react-native/pull/29117) `[Android] Fix font weight numeric values` by [@fabriziobertoglio1987](https://github.com/fabriziobertoglio1987)
- **Interaction**
  - [#28995](https://github.com/facebook/react-native/pull/28995) `[Android] Fix TextInput Cursor jumping to the right when placeholder null` by [@fabriziobertoglio1987](https://github.com/fabriziobertoglio1987)
  - [#28952](https://github.com/facebook/react-native/pull/28952) `[Android] Fix non selectable Text in FlatList` by [@fabriziobertoglio1987](https://github.com/fabriziobertoglio1987)
  - [#29046](https://github.com/facebook/react-native/pull/29046) `[Android] onKeyPress event not fired with numeric keys` by [@fabriziobertoglio1987](https://github.com/fabriziobertoglio1987)
  - [#31500](https://github.com/facebook/react-native/pull/31500) `fix#29319 - ios dismiss modal` by [@intergalacticspacehighway](https://github.com/intergalacticspacehighway)
  - [#32179](https://github.com/facebook/react-native/pull/32179) `Fix: multiline textinput start "jerking" when trying to move cursor.` by [@xiankuncheng](https://github.com/xiankuncheng)
  - [#29039](https://github.com/facebook/react-native/pull/29039) `Fix to make taps on views outside parent bounds work on Android` by [@hsource](https://github.com/hsource)
- **Performance**
  - [#31764](https://github.com/facebook/react-native/pull/31764) `Optimize font handling on iOS` by [@Adlai-Holler](https://github.com/Adlai-Holler)
  - [#32536](https://github.com/facebook/react-native/pull/32536) `Don't reconstruct app component on split-screen` by [@Somena1](https://github.com/Somena1)
- **Testing**
  - [#31401](https://github.com/facebook/react-native/pull/31401) `Add unit tests for VirtualizedList render quirks` by [@NickGerleman](https://github.com/NickGerleman)

部分 PR 解決了同時影響 Meta 和整個開源社區的問題，從相關已關閉議題的反應數量即可看出其重要性。

還有許多其他我們想特別提及的 PR，我們要再次**感謝**所有花時間幫助我們解決錯誤並改進 React Native 的人們。

## 社群參與

在半年開始時，我們設定了一個目標，要**更頻繁地與社群溝通**並建立持續此行為的流程。以下是我們在 2021 年下半年的一些互動：

<!--alex ignore gross-->

- 我們有機會參與 [React Native EU](https://www.react-native.eu/)，由 [Joshua Gross](https://twitter.com/joshuaisgross) 帶來一場演講 - [將 Fabric 渲染器帶入「Facebook」應用](https://www.youtube.com/watch?v=xKOkILSLs0Q&t=3987s)
- 我們在 Reddit 上舉辦了一場 [「問我們任何事」（AUA）](https://www.reddit.com/r/reactnative/comments/pzdo1r/react_native_team_aua_thursday_oct_14_9am_pt/)，收到了超過 100 個問題！AUA 對我們來說是一個了解社群參與度的好機會，對你們來說也是一個提出任何問題的機會。如果你還沒看過，一定要去看看答案，因為其中一些非常有見地。
- 我們分享了我們的 [多平台願景](https://reactnative.dev/blog/2021/08/26/many-platform-vision)，提供了 [Android 12 和 iOS 15](https://reactnative.dev/blog/2021/09/01/preparing-your-app-for-iOS-15-and-android-12) 的注意事項指南，以及 [Hermes 成為 React Native 預設 JS 引擎](https://reactnative.dev/blog/2021/10/26/toward-hermes-being-the-default) 的進展和願景！
- 我們的 [Kevin Gozali](https://twitter.com/fkgozali) 出現在 [React Native Radio 播客的一集](https://reactnativeradio.com/episodes/rnr-222-the-new-architecture-with-kevin-gozali-from-the-rn-core-team) 中，談論了新架構。
- 在 [ReactConf 2021](https://conf.reactjs.org/) 上，[Rick Hanlon](https://twitter.com/rickhanlonii) 分享了 React 和 React Native 的統一多平台願景。此外，[Eric Rozell](https://twitter.com/EricRozell) 和 [Steven Moyes](https://twitter.com/moyessa) 展示了 React Native Desktop 在支持 Meta 和 Microsoft 應用方面的驚人進展，並展示了多平台願景的實踐。

除了在 2021 年下半年分享更多更新外，我們也比以往任何時候都**更依賴**我們的社群。我們依賴貢獻者在試用新架構材料的早期草案時提供的關鍵反饋。同時，我們在調試關鍵版本問題和改進方面也得到了社群專家的極大支持。

我們的社群為 React Native 帶來了豐富的知識，我們需要繼續培育它。

## 新架構的推出與版本發布

2022 年將是**開源新架構**的一年。

我們一直在努力交付推出新架構到應用和函式庫所需的基礎設施。我們邀請了一些合作夥伴和核心貢獻者/函式庫維護者參與，以完善我們對新架構的支持，並獲得早期反饋。

我們現在正在準備在網站上發布一份新指南：[新架構入門](https://github.com/facebook/react-native-website/pull/2879)。這將是 2022 年我們將發布的一系列材料的起點，這些材料將幫助你遷移/開始你的新架構項目。

Moreover, we would like to stress the [importance of **giving feedback**](https://github.com/facebook/react-native-website/pull/2879) on the New Architecture material. We’re still in the process of finalizing the last details and your input will help everyone adopt the new architecture more seamlessly.

**版本發布**在新架構的推廣過程中扮演著關鍵角色。我們在上半年的目標是確保任何阻礙發布的問題不會停滯不前。我們通過[明確並改進流程與職責](https://github.com/facebook/react-native/wiki/Releases)來提高問責制。現在，我們的發布協調工作會在一個[專屬的討論庫](https://github.com/reactwg/react-native-releases/discussions)中進行，並提供了[更清晰的發布問題報告機制](https://github.com/facebook/react-native/issues/new?assignees=&labels=Needs%3A+Triage+%3Amag%3A%2CType%3A+Upgrade+Issue&template=upgrade-regression-form.yml)。

在2022年上半年，我們將繼續迭代發布職責以支持新架構的推廣。如果您想協助測試發布候選版本或[參與改進工作](https://github.com/facebook/react-native/projects/18)，歡迎[加入討論](https://github.com/reactwg/react-native-releases/discussions/categories/improvements)！

## 邁向移動端與更遠的未來

從[ReactConf的演講陣容](https://conf.reactjs.org/)可以看出，React Native不僅僅局限於Android和iOS。

早在2021年，我們就分享了我們的[多平台願景](https://reactnative.dev/blog/2021/08/26/many-platform-vision)，並在桌面端和VR領域成功推廣了React Native。

我們期待將**平台特定的模式**整合到React Native的體驗中。

最後，我們要再次感謝社區在2021年下半年的大力支持。看到貢獻者們在GitHub上齊心協力、互相幫助，修復錯誤、分享知識，並協助我們將React Native帶給數百萬用戶，這總是令人驚嘆。

敬請期待，並展望**更加精彩的2022年** 🎉！