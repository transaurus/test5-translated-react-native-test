---
title: Toward Hermes being the Default
authors: [huxpro]
tags: [announcement]
---

Since [we announced Hermes in 2019](https://engineering.fb.com/2019/07/12/android/hermes/), it has been increasingly gaining adoption in the community. The team at [Expo](https://expo.dev/), who maintain a popular meta-framework for React Native apps, recently [announced experimental](https://blog.expo.dev/expo-sdk-42-579aee2348b6) [support](https://blog.expo.dev/expo-sdk-43-beta-is-now-available-47dc54a8d29f) for Hermes after being [one of the most requested features of Expo](https://expo.canny.io/feature-requests/p/enabling-hermes). The team at [Realm](https://realm.io/), a popular mobile database, also recently shipped its [alpha support](https://github.com/realm/realm-js/issues/3940) for Hermes. In this post, we want to highlight some of the most exciting progress we've made over the past two years to push Hermes towards being _the best_ JavaScript engine for React Native. Looking forward, we are confident that with these improvements and more to come, we can make Hermes the default JavaScript engine for React Native across all platforms.

<!--truncate-->

## 為React Native優化

Hermes的定義性特色在於它如何事先執行編譯工作，這意味著啟用Hermes的React Native應用程式會附帶預先編譯的最佳化位元組碼，而非純JavaScript原始碼。這大幅減少了使用者啟動產品所需的工作量。來自Facebook和社群應用程式的測量數據顯示，啟用Hermes通常能將產品的TTI（或[互動時間](https://web.dev/interactive/)）指標減少近一半。

話雖如此，我們一直在改進Hermes的許多其他方面，讓它作為專為React Native打造的JavaScript引擎變得更好。

### 為Fabric打造新的垃圾回收器

With the upcoming [Fabric](https://github.com/react-native-community/discussions-and-proposals/issues/4) renderer in the new React Native architecture, it will be possible to synchronously call JavaScript on the UI thread. However, this means if the JavaScript thread takes too long to execute, it can cause noticeable UI frame drops and block user inputs. The [concurrent rendering](https://reactjs.org/blog/2021/06/08/the-plan-for-react-18.html) enabled by React [Fiber](https://reactjs.org/docs/faq-internals.html#what-is-react-fiber) will avoid scheduling long JavaScript tasks by splitting rendering work into chunks. However, there is another common source of latency from the JavaScript thread — when the JavaScript engine has to “stop the world” to perform a garbage collection (GC).

Hermes中先前的預設垃圾回收器[GenGC](https://hermesengine.dev/docs/gengc/)是一個單執行緒的分代垃圾回收器。新世代使用典型的半空間複製策略，而舊世代則使用標記-壓縮策略，使其非常擅長積極將記憶體返回給作業系統。由於其單執行緒特性，GenGC的缺點是會導致長時間的GC暫停。在像Facebook for Android這樣複雜的應用程式上，我們觀察到平均暫停時間為200毫秒，或在p99時為1.4秒。考慮到Facebook for Android龐大且多樣的使用者群，我們甚至見過長達7秒的暫停。

In order to mitigate this, we implemented a brand new _mostly concurrent_ GC named [Hades](https://hermesengine.dev/docs/hades). Hades collects its young generation exactly the same as GenGC, but it manages its old generation with a snapshot-at-the-beginning style mark-sweep collector. which can significantly reduce GC pause time by performing most of its work in a background thread without blocking the engine’s main thread from executing JavaScript code. **Our statistics show that Hades only pauses for 48ms at p99.9 on 64-bit devices (34x faster than GenGC!)** and around 88ms at p99.9 on 32-bit devices (where it operates as a single-threaded _incremental_ GC). These pause time improvements can come at the cost of overall throughput, due to the need for more expensive write barriers, slower freelist based allocation (as opposed to a bump pointer allocator), and increased heap fragmentation. We think those are the right trade-offs, and we were able to achieve overall lower memory consumption via coalescing and additional memory optimizations that we’ll talk about.

### 針對性能痛點出擊

應用程式的啟動時間對許多應用的成功至關重要，我們持續為 React Native 推進這一邊界。對於 Hermes 中實現的任何新 JavaScript 功能，我們都會仔細監控它們對生產性能的影響，確保不會導致指標倒退。在 Facebook，我們目前正在實驗 [Metro 中專為 Hermes 設計的 Babel 轉換配置](https://github.com/facebook/react-native/blob/main/packages/react-native-babel-preset/src/configs/main.js#L41)，用 Hermes 的原生 ESNext 實現取代十幾個 Babel 轉換。我們在許多場景中觀察到 **18-25% 的 TTI 改善** 以及 **整體字節碼大小減少**，並期待在開源社群中看到類似結果。

除了啟動性能外，我們發現記憶體佔用是 React Native 應用（尤其是 [虛擬實境](https://reactnative.dev/blog/2021/08/26/many-platform-vision#expanding-to-new-platforms)）的另一個改進機會。得益於我們作為 JavaScript 引擎的低層級控制能力，我們通過細微優化實現了多輪記憶體改進：

1. 先前，所有 JavaScript 值都以 64 位元 NaN-boxing 編碼的標記值表示，用於在 64 位元架構上表示浮點雙精度數和指針。然而，這在實踐中是浪費的，因為大多數數字是小整數（SMI），且客戶端應用的 JavaScript 堆通常不會超過 4GiB。為解決此問題，我們引入了一種新的 32 位元編碼，其中 SMI 和指針以 29 位元編碼（因為指針是 8 位元組對齊的，我們可以假設最低 3 位元始終為零），其餘 JS 數字則被裝箱到堆上。**這使 JavaScript 堆大小減少了約 30%。**
2. 不同類型的 JavaScript 物件在 JavaScript 堆中表示為不同類型的 GC 管理單元。通過積極優化這些單元的標頭記憶體布局，**我們進一步減少了約 15% 的記憶體使用量**。

我們在 Hermes 中的一個關鍵決策是不實現 [即時編譯器（JIT）](https://en.wikipedia.org/wiki/Just-in-time_compilation)，因為我們認為對於大多數 React Native 應用，額外的暖機成本和二進位及記憶體上的額外佔用並不值得。多年來，我們投入大量精力優化解釋器性能和編譯器優化，使 Hermes 的吞吐量在 React Native 工作負載上能與其他引擎競爭。我們持續通過識別各處的性能瓶頸（解釋器調度循環、堆疊布局、物件模型、GC 等）來改善吞吐量。期待在未來的版本中看到更多數字！

### 垂直整合的開創者

在 Facebook，我們偏好將專案集中在一個大型 [單一倉庫](https://en.wikipedia.org/wiki/Monorepo) 中。通過讓引擎（Hermes）和宿主（React Native）緊密協同迭代，我們開創了許多垂直整合的機會。舉例來說：

- Hermes supports [on-device JavaScript debugging with the Chrome debugger](https://reactnative.dev/docs/hermes#debugging-js-on-hermes-using-google-chromes-devtools) by speaking the [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/). It’s better than the legacy “[Remote JS Debugging](https://reactnative.dev/docs/debugging#chrome-developer-tools)” (which uses an in-app proxy to run JS in desktop Chrome) because it supports debugging synchronous native calls and guaranteed a consistent runtime environment. Together with React DevTools, Metro, Inspector, and so on, Hermes debugger is now part of [Flipper](https://reactnative.dev/blog/2020/03/26/version-0.62) to provide a one-stop developer experience.
- Objects allocated during the initialization path of React Native apps are often long-lived and don’t follow the _generational_ _hypothesis_ leveraged by generational GCs. Therefore, we [configured Hermes in React Native](https://github.com/facebook/react-native/blob/main/ReactAndroid/src/main/java/com/facebook/hermes/reactexecutor/OnLoad.cpp#L37-L42) to allocate the first 32MiB directly into old generations (known as _pre-tenuring_) to avoid triggering GC pauses and delaying TTI.
- The new React Native architecture is heavily based on [JSI (or JavaScript Interface)](https://github.com/react-native-community/discussions-and-proposals/issues/91), a lightweight, general-purposed API for embedding a JavaScript engine into a C++ program. By having the team maintaining the JS engine also maintains the JSI API implementation, we are confident in providing the best possible integration that is reliable, performant and battle-tested at the Facebook’s scale.
- Getting JavaScript concurrency primitives (e.g. [promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)) and platform concurrency primitives (e.g. [microtasks](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide)) both semantically correct and performant are critical to React concurrent rendering and the future of React Native apps. Historically, promises in React Native were [polyfilled](https://github.com/facebook/react-native/blob/main/Libraries/Core/polyfillPromise.js#L37) using non-standardized [`setImmediate`](https://developer.mozilla.org/en-US/docs/Web/API/Window/setImmediate) APIs. We are working on making native promises and microtasks from JS engines available via JSI, and introducing [`queueMicrotask`](https://developer.mozilla.org/en-US/docs/Web/API/queueMicrotask), a recent addition to the web standard, to the platform, to better support modern asynchronous JavaScript code.

## 攜手整個社群

Hermes 對 Facebook 而言表現卓越，但唯有當整個生態系統都能運用 Hermes 來驅動體驗時，我們的工作才算完成，讓所有人充分發揮其功能與潛力。

### 擴展至新平台

Hermes 最初僅針對 Android 版 React Native 開源。此後，我們欣喜地看到社群成員將 Hermes 支援擴展至[React Native 生態系涵蓋的眾多其他平台](https://reactnative.dev/blog/2021/08/26/many-platform-vision)。

[Callstack](https://callstack.com/)主導了[在 React Native 0.64 中將 Hermes 引入 iOS](https://reactnative.dev/blog/2021/03/12/version-0.64)的工作。他們撰寫了[系列文章](https://callstack.com/blog/bringing-hermes-to-ios-in-react-native/)並舉辦[播客](https://callstack.com/podcasts/react-native-0-64-with-hermes-for-ios-ep-5)分享實作過程。根據其基準測試，在 Mattermost 應用中，Hermes 能**在 iOS 上持續帶來約 40% 的啟動效能提升與約 18% 的記憶體減少**，相較於 JSC，且僅增加 2.4 MiB 的應用大小負擔。建議您[親眼見證實際效果](https://callstack.com/blog/hermes-performance-on-ios/)。

微軟一直致力於將 [Hermes 引入 React Native for Windows 和 macOS](https://microsoft.github.io/react-native-windows/docs/hermes)。[在 Microsoft Build 2020](https://youtu.be/QMFbrHZnvvw?t=389) 上，微軟分享 Hermes 的記憶體影響（[工作集](https://en.wikipedia.org/wiki/Working_set)）比 React Native for Windows 上的 Chakra 引擎低 13%。最近，在一些合成基準測試中，他們發現 Hermes 0.8（搭載 Hades 及前述的 SMI 和指標壓縮優化）**比其他引擎少用 30%-40% 的記憶體**。毫不意外，基於 React Native 構建的 [桌面版 Messenger](https://www.messenger.com/desktop) 視訊通話體驗，同樣由 Hermes 驅動。

最後但同樣重要的是，Hermes 也驅動了所有在 Oculus 上使用 React 技術家族構建的虛擬實境體驗，包括 Oculus Home。

### 支援我們的社群

我們承認目前仍有部分障礙阻止社群某些成員採用 Hermes，而我們致力於為這些缺失的功能提供支援。我們的目標是實現完整功能，使 Hermes 成為大多數 React Native 應用的理想選擇。以下是社群如何已經塑造了 Hermes 的路線圖：

<!-- alex ignore just fellowship -->

- [`Proxy` and `Reflect`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Meta_programming) were originally excluded from Hermes because Facebook does not use them. We were also concerned that adding Proxy would hurt property lookup performance even when Proxy is not used. But Proxy quickly become [the most requested feature](https://github.com/facebook/hermes/issues/33) of Hermes due to popular libraries such as [MobX](https://mobx.js.org/README.html) and [Immer](https://immerjs.github.io/immer/). We carefully evaluated and decided to build it just for the community, and we managed to implement it with very low cost. Since this is a feature we don’t use, we relied on our community to prove its stability. We started by testing Proxy behind a flag and created opt-in npm packages for [release v0.4](https://github.com/facebook/hermes/issues/33#issuecomment-668374607) and [v0.5](https://github.com/facebook/hermes/issues/33#issuecomment-668374607), and it’s [enabled by default starting from v0.7](https://github.com/facebook/hermes/releases/tag/v0.7.0).
- [ECMAScript Internationalization API Specification (ECMA-402, or `Intl`)](https://hermesengine.dev/docs/intl) was [the second most requested feature](https://github.com/facebook/hermes/issues/23). `Intl` is a huge set of APIs and often requires the implementation to include **6MB worth** of [Unicode CLDR](https://cldr.unicode.org/index) data. This is why polyfills like [FormatJS (a.k.a. `react-intl`)](https://github.com/formatjs/formatjs) and JS engines like the [international variant build of community JSC](https://github.com/react-native-community/jsc-android-buildscripts#international-variant) are so huge. To avoid substantially increasing the binary size of Hermes, we decided to implement it with another strategy by consuming and mapping the ICU facilities provided by the libraries included in the operating systems, at the cost of some (often minor) variance in behaviors across platforms.
  - Microsoft collaborated to build support on Android. It covers almost everything from ECMA-402 up to ES2020, **with only a size impact as small as 3% (57-62K per ABI)**. We ran [a poll on Twitter](https://twitter.com/tmikov/status/1336442786694893568) and the results were strongly in favor of including `Intl` by default, so that’s what we did and it’s available starting from [release v0.8](https://github.com/facebook/hermes/releases/tag/v0.8.0).
  - Facebook has sponsored [Major League Hacking](https://mlh.io/) to launch a [remote open source fellowship program](https://news.mlh.io/welcoming-facebook-back-as-a-sponsor-of-the-2020-2021-mlh-fellowship-08-12-2020). Last year, we launched the [Hermes sampling profiler](https://reactnative.dev/docs/profile-hermes). This year, our fellows will be working with members from Hermes, React Native, and Callstack, to add support for Hermes `Intl` on iOS. Stayed tuned!
- We appreciate that people have been working with us to discover issues affecting the community.
  - People have helped us identify critical spec divergence such as [stability of `Array.prototype.sort`](https://github.com/facebook/hermes/issues/212) amended in [ES2019](https://github.com/tc39/ecma262/pull/1340). This has been fixed and will be available in the next release.
  - People found out that our default heap size limit was too small and caused [unnecessary GC pressure](https://github.com/facebook/hermes/issues/295) and [OOM crashes](https://github.com/facebook/hermes/issues/511) for many users who are not familiar with customizing Hermes’s GC configs. So we increased it from 512MiB to 3GiB to be more than sufficient for most users by default.
  - People also reported that our specialized `Function.prototype.toString` implementation [caused performance to drop in libraries doing improper feature detection](https://github.com/facebook/hermes/issues/471#issuecomment-820123463) and [blocked users from doing source code injecting](https://github.com/facebook/hermes/issues/114#issuecomment-887106990). This helped us strengthen our stance that Hermes, whenever possible, should not get in the way of developers and to respect de-facto practices.

## 總結

總結來說，我們的願景是讓 Hermes 成為所有 React Native 平台的預設 JavaScript 引擎。我們已經開始朝這個方向努力，並且希望聽取大家對這個方向的意見。

為生態系統做好順利採用的準備對我們來說極為重要。我們鼓勵您試用 Hermes，並在我們的 [GitHub 儲存庫](https://github.com/facebook/hermes) 中提交任何反饋、問題、功能請求和不兼容性的問題。

## 感謝

我們要感謝 Hermes 團隊、React Native 團隊以及來自 React Native 社群的許多貢獻者，感謝他們為改進 Hermes 所做的工作。

<!-- alex ignore white -->

我個人也要感謝（按字母順序）Eli White、Luna Wei、Neil Dhar、Tim Yung、Tzvetan Mikov 以及許多其他在撰寫過程中提供幫助的人。