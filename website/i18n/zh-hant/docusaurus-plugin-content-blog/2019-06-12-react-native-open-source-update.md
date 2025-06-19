---
title: React Native Open Source Update June 2019
authors: [cpojer]
tags: [announcement]
---

## 程式碼與社群健康狀況

過去六個月內，共有超過550位貢獻者向React Native提交了2800次提交。來自社群的400位貢獻者建立了超過[1,150個拉取請求](https://github.com/facebook/react-native/pulls?page=24&q=is%3Apr+closed%3A%3E2018-12-01&utf8=%E2%9C%93)，其中[820個拉取請求](https://github.com/facebook/react-native/pulls?utf8=%E2%9C%93&q=is%3Apr+closed%3A%3E2018-12-01+label%3A%22Merged%22+)被合併。

儘管我們透過Lean Core計劃將網站、CLI和許多模組從React Native中拆分出來，過去六個月內每日平均拉取請求數量已從三個增加至約六個。目前平均未處理的拉取請求數量低於25個，我們通常會在數小時或數天內回覆建議與審查意見。

### 具意義的社群貢獻

我們想特別強調以下幾項近期我們認為出色的貢獻：

- **Accessibility:** React Native 0.60 will ship with many improvements to accessibility APIs both on Android and iOS. All of the new features are directly using APIs provided by the underlying platform so they’ll integrate with native assistance technologies both on Android and iOS. We’d like to thank [Marc Mulcahy](https://github.com/marcmulcahy), [Alan Kenyon](https://github.com/facebook/react-native/pull/24746), [Estevão Lucas](https://github.com/elucaswork), [Sam Mathias Weggersen](https://github.com/sweggersen) and [Janic Duplessis](https://twitter.com/janicduplessis) for their contributions:
  - [Additional Accessibility Roles + States](https://github.com/facebook/react-native/pull/24095) and a [new Accessibility States API](https://github.com/facebook/react-native/pull/24608). Added a number of missing accessibility roles for various components and a new API for better web support in the future.
  - [AccessibilityInfo.announceForAccessibility](https://github.com/facebook/react-native/pull/24746). Added support for Android, previously iOS-only.
  - [Extended Accessibility Actions Support](https://github.com/facebook/react-native/pull/24695). Added callbacks to deal with accessibility around user-defined actions.
  - [Support for iOS Accessibility flags](https://github.com/facebook/react-native/pull/23913) and [support for "reduce motion"](https://github.com/facebook/react-native/pull/23839).
  - [Android keyboard accessibility improvements](https://github.com/facebook/react-native/pull/24359). Added a `clickable` prop and an `onClick` callback for invoking actions via keyboard navigation _(note: this will soon be renamed to `focusable`)._
  - [Use CALayers to draw text](https://github.com/facebook/react-native/pull/24387). Fixed an issue that made scaled-up text disappear on iOS.
- **New App Screen:** The community came up with a [design for the new app screen](https://github.com/react-native-community/discussions-and-proposals/issues/122) that is implemented in 0.60. This screen is what most people see when they are first using React Native. It now links first time users to the documentation and the look fits with our upcoming website redesign 🌟. Huge thanks to [Orta](https://twitter.com/orta), [Adam Shurson](https://www.linkedin.com/in/ashurson/), [Glauber Castro](https://github.com/glauberfc), [Karan Singh](https://github.com/karanpratapsingh), [Eli Perkins](https://twitter.com/_eliperkins), [Lucas Bento](https://twitter.com/lbentosilva) and [Eric Lewis](https://twitter.com/ericlewis) for all their work and collaboration!
  - Check out the new app screen on the “_[React Native Show](https://www.youtube.com/watch?v=ImlAqMZxveg)_“ video series.
- **TurboModule Types:** The new [TurboModules system](https://github.com/react-native-community/discussions-and-proposals/issues/40) requires [types for all native modules](https://github.com/facebook/react-native/issues/24875) to guarantee type safe operations in native. In just over two weeks, the community sent ~40 Pull Requests to complete this work for flow typed native modules. Aside from the people already mentioned above, we’d like to thank [Michał Chudziak](https://twitter.com/michalchudziak), [Michał Pierzchała](https://twitter.com/thymikee), [Wojtek Szafraniec](https://github.com/wojteg1337), and [Jean Regisser](https://github.com/jeanregisser) and everyone else who sent one or more Pull Requests.
- **Haste:** Since 2015 React Native used the [“haste” module system](https://github.com/reactjs/reactjs.org/commit/0629e3e2289ed54fac854472aec9a5f6c8318c98#diff-c42b758729cb89976b3a8fd51d1227fa) that allows importing modules just via a global id instead of a relative path which is convenient but not well supported by many tools. [James Ide](https://twitter.com/JI) proposed removing haste, similar to how React removed haste many years ago. He planned all the work through an [umbrella task](https://github.com/facebook/react-native/issues/24316) and he sent 18 Pull Requests to make it happen! Check out [his Twitter thread](https://twitter.com/JI/status/1136369775083319296) to learn more.
- **Android Fragments:** [John Shelley](https://github.com/jpshelley)‘s proposal to make React Native work via [Android Fragments](https://github.com/facebook/react-native/pull/12199) was merged and will be available in 0.61. [Read more about Android Fragments here](https://developer.android.com/guide/components/fragments).

### Lean Core計劃

The primary motivation of [Lean Core](https://github.com/react-native-community/discussions-and-proposals/issues/6) has been to split modules out of React Native into separate repositories so they can receive better maintenance. In just a six months repositories like [WebView](https://github.com/react-native-community/react-native-webview), [NetInfo](https://github.com/react-native-community/react-native-netinfo), [AsyncStorage](https://github.com/react-native-community/react-native-async-storage), the [website](https://github.com/facebook/react-native-website) and the [CLI](https://github.com/react-native-community/cli) received more than 800 Pull Requests combined. Besides better maintenance, these projects can also be independently released more often than React Native itself.

我們也藉此機會從React Native移除了過時的polyfill和舊版元件。過去為了在舊版JavaScriptCore（JSC）中支援像[`Map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)和[`Set`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)等語言特性，polyfill是必要的。現在React Native已搭載新版JSC，這些polyfill已被移除。

這項工作仍在進行中，還有許多內容需要從原生端和JavaScript端拆分或移除，但已有早期跡象顯示我們成功逆轉了表面積和應用體積增長的趨勢：以JavaScript套件為例，約一年前的0.54版中React Native的JavaScript套件大小為530kb，在短短六個月內到0.57版時增長至607kb（+77kb）。現在我們看到主分支上的套件大小減少了28kb至579kb，差異超過100kb！

隨著Lean Core計劃第一階段告一段落，我們將更審慎地評估新增至React Native的API，並持續探索讓React Native更小、更快的方法，同時尋找讓社群能接管各種元件的方式。

## 使用者回饋

六個月前我們詢問社群「[您不喜歡React Native的哪些地方？](https://github.com/react-native-community/discussions-and-proposals/issues/64)」，這讓我們對人們面臨的問題有了很好的了解。我們[幾個月前回覆了該貼文](https://github.com/react-native-community/discussions-and-proposals/issues/104)，現在是時候總結針對主要問題所取得的進展：

- **升級體驗：** React Native 社群齊心協力改善了多項升級體驗：包括[自動連結](https://github.com/react-native-community/cli/blob/master/docs/autolinking.md)、透過 [rn-diff-purge](https://github.com/react-native-community/rn-diff-purge) 提供更完善的升級指令，以及即將推出的升級輔助網站。我們也會透過每項主要版本的部落格文章，確保清楚傳達重大變更與新功能。這些改進將大幅簡化未來 0.60 版本之後的升級流程。
- **支援與不確定性：** 許多人對 Pull Request 缺乏進展感到挫折，並對 Facebook 投資 React Native 的承諾存疑。如上述數據所示，我們有信心迎接更多 Pull Request，並熱切期待您的提案與貢獻！
- **效能：** React Native 0.59 搭載了速度大幅提升的新版 JavaScriptCore (JSC)。此外，我們也持續優化預設啟用 [inline-requires](/docs/performance#ram-bundles-inline-requires) 的流程，未來幾個月將有更多更新。
- **文件：** 我們已啟動全面[翻新與重寫 React Native 文件](https://github.com/facebook/react-native-website/issues/929)的計畫，誠摯歡迎您的參與貢獻！
- **Xcode 警告：** 我們已[清除所有現有警告](https://github.com/facebook/react-native/issues/22609)，並致力避免新增警告。
- **熱重載：** React 團隊正在開發[新版熱重載系統](https://twitter.com/dan_abramov/status/1126948870137753605)，即將整合至 React Native。

目前仍有未盡完善之處：

- **除錯功能：** 我們修復了許多日常會遇到的惱人錯誤，但進展未如預期。我們理解 React Native 的除錯體驗有待加強，後續將優先改善此項目。
- **Metro 符號連結：** 尚未能提供簡易解決方案，但 React Native 使用者已[分享多種替代方案](https://github.com/facebook/metro/issues/1)可供參考。

Given the large amount of changes in the past six months, we'd like to ask you the same question again. If you are using the latest version of React Native and you have things you'd like to give feedback on, please comment on our new edition of [“What do you dislike about React Native?”](https://github.com/react-native-community/discussions-and-proposals/issues/134)

## 持續整合

Facebook 會先將所有 Pull Request 和內部變更合併至內部儲存庫，再同步回 GitHub。由於 Facebook 基礎架構與常見 CI 服務不同，並非所有開源測試都會在內部執行，導致同步至 GitHub 的提交經常使開源測試失敗，需耗費大量時間修復。

[Héctor Ramos](https://twitter.com/hectorramos) from the React Native team spent the past two months improving React Native's continuous integration systems both at Facebook and on GitHub. Most of the open source tests are now run before changes are committed to React Native at Facebook which will keep CI stable on GitHub when commits are being synchronized.

## 下一步

請關注我們關於 React Native 未來的演講！未來幾個月，Facebook React Native 團隊成員將在 [Chain React](https://infinite.red/ChainReactConf) 和 [React Native EU](https://react-native.eu/) 發表演說。同時請密切期待即將發布的 0.60 版本——_這將會非常精彩_ ✨