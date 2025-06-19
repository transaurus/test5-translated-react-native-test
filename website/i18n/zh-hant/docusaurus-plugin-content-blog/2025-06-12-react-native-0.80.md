---
title: 'React Native 0.80 - React 19.1, JS API Changes, Freezing Legacy Arch and much more'
authors: [hezi, fabriziocucci, gabrieldonadel, chrfalch]
tags: [engineering]
date: 2025-06-12T16:01
---

# React Native 0.80 - React 19.1、JS API 變更、凍結舊版架構與更多更新

今天我們興奮地發布 React Native 0.80！

此版本將 React Native 內建的 React 版本更新至最新穩定版：19.1.0。

我們同時推出了一系列 JavaScript API 的穩定性改進：深層導入現在會觸發警告，並提供新的選擇性嚴格 TypeScript API，提供更精確且安全的型別。

此外，React Native 的舊版架構現已正式凍結，您將開始看到針對未來停止支援 API 的警告訊息。

### 重點更新

- [JavaScript 深層導入棄用](/blog/2025/06/12/react-native-0.80#javascript-deep-imports-deprecation)
- [凍結舊版架構與警告](/blog/2025/06/12/react-native-0.80#legacy-architecture-freezing--warnings)
- [React 19.1.0](/blog/2025/06/12/react-native-0.80#react-1910)
- [實驗性功能 - React Native iOS 相依項預編譯](/blog/2025/06/12/react-native-0.80#experimental---react-native-ios-dependencies-are-now-prebuilt)

<!--truncate-->

## 重點說明

### JavaScript 深層導入棄用

此版本中，我們正採取措施改進並穩定 React Native 的公開 JavaScript API。第一步是更明確界定哪些 API 可供應用程式與框架導入。據此，我們正式棄用 React Native 的深層導入（參見 [RFC](https://github.com/react-native-community/discussions-and-proposals/pull/894)），並透過 ESLint 與 JS 控制台引入警告機制。

這些警告僅針對專案原始碼中的導入行為，可透過[設定選擇退出](/docs/strict-typescript-api)。但請注意，我們計劃在未來版本中完全移除深層導入，建議您將這些導入更新為根目錄導入方式。

```js
// Before - import from subpath
import {Alert} from 'react-native/Libraries/Alert/Alert';

// After - import from `react-native`
import {Alert} from 'react-native';
```

部分 API 未從根目錄導出，未來將無法透過深層導入使用。這是刻意為之，旨在減少 React Native API 的整體暴露範圍。我們開設了[意見回饋討論串](https://github.com/react-native-community/discussions-and-proposals/discussions/893)，將與社群共同決定最終保留的 API 清單（至少涵蓋未來兩個 React Native 版本）。歡迎分享您的意見！

詳見專文說明：[邁向穩定的 JavaScript API](/blog/2025/06/12/moving-towards-a-stable-javascript-api)。

#### 選擇性嚴格 TypeScript API

隨著公開 API 導出方式的重新定義，我們在 0.80 版本中同時為 `react-native` 套件提供一組新型 TypeScript 型別，稱為「嚴格 TypeScript API」。

啟用嚴格 TypeScript API 可預覽 React Native 未來穩定版 JavaScript API 的樣貌。這些新型別具有以下特點：

1. **直接從原始碼生成** — 提升覆蓋率與正確性，提供更強的相容性保證
2. **限制僅能從 React Native 索引檔案導入** — 更嚴格定義公開 API，避免因內部檔案變更導致 API 斷裂

這些新型別與現有型別並存，您可自行選擇遷移時機。若您使用標準 React Native API，多數應用程式應能**無需修改**即通過驗證。我們強烈建議早期採用者與新專案透過 `tsconfig.json` 檔案啟用此功能。

待社群準備就緒後，嚴格 TypeScript API 將成為未來預設 API — 並與深層導入移除時程同步實施。

了解更多關於此變更的詳細資訊，請參閱我們的專文：[邁向穩定的 JavaScript API](/blog/2025/06/12/moving-towards-a-stable-javascript-api)。

### 舊版架構凍結與警告

React Native 的新架構自 [0.76 版本](/blog/2024/10/23/the-new-architecture-is-here)起已成為預設選擇，我們也讀到了許多專案與工具[成功案例](https://blog.kraken.com/product/engineering/how-kraken-fixed-performance-issues-via-incremental-adoption-of-the-react-native-new-architecture)的分享，這些案例都大幅受益於新架構。

[我們近期宣布](https://github.com/reactwg/react-native-new-architecture/discussions/290)，現已將舊版架構視為凍結狀態。我們將不再為舊版架構開發新的錯誤修正或功能，且在發布新版本時也不再測試舊版架構。

為了讓遷移過程更順利，我們仍允許使用者選擇退出新架構，若您遇到錯誤或退步問題時可使用此選項。

然而，在 React Native 中同時維護兩種架構是一項巨大挑戰，這會影響運行時效能、應用程式大小以及程式碼庫的維護工作。

因此，我們最終必須在未來某個時間點淘汰舊版架構。

在 0.80 版本中，我們新增了一系列警告訊息，這些訊息會在 React Native DevTools 中彈出，提醒您若使用了在新架構中無法運作的 API。

我們建議您不要忽略這些警告，並考慮將您的應用程式與函式庫遷移至新架構，以迎接未來的變革。

![舊版架構警告](/blog/assets/0.80-legacy-arch-warnings.png)

您可以在我們近期於 App.js 發表的演講「Life After Legacy: The New Architecture Future」中，了解更多關於這些變更的資訊 [觀看影片](https://www.youtube.com/live/K2JTTKpptGs?si=tRkT535f0GzguVGt&t=9011)。

### React 19.1.0

此版本的 React Native 搭載了最新的 React 穩定版：19.1.0

您可以在 [發布說明](https://github.com/facebook/react/releases/tag/v19.1.0) 中閱讀 React 19.1.0 的所有新功能與錯誤修正。

:::warning

React 19.1.0 的一項顯著功能是擁有者堆疊（owner stacks）的實作與改進。這是一項僅限開發階段的功能，可協助您識別是哪個元件導致特定錯誤。

然而，我們注意到若您使用了 `@babel/plugin-transform-function-name` Babel 插件（此插件在 React Native Babel Preset 中預設啟用），擁有者堆疊在 React Native 中可能無法如預期運作。我們將在未來的 React Native 版本中發布修正。

:::

### 實驗性功能 - React Native iOS 相依套件現已預先建置

若您正在建置 React Native iOS 應用程式，可能已注意到首次原生建置會耗費較長時間：在老舊機器上可能需要數分鐘甚至更久。這是因為我們需要編譯整個 React Native iOS 程式碼及其所有相依套件。

過去幾週，我們一直在實驗為 iOS 預先建置部分 React Native 核心程式碼（類似 Android 的做法），以縮短首次運行 React Native 應用程式時的建置時間。

React Native 0.80 是首個能將部分 iOS 版 React Native 以預先建置形式發布的版本，有助於減少建置時間。

在 React Native 的發布過程中，我們會產生一個名為 `ReactNativeDependencies.xcframework` 的 XCFramework，這是 React Native 所依賴的第三方套件的預先建置版本。

我們進行了實驗與效能基準測試，測量此 iOS 預先建置功能節省的時間。在 M4 機器上進行的測試顯示，使用預先建置的 iOS 建置速度比從原始碼建置快了約 12%。

根據我們的經驗，我們也觀察到許多使用者的錯誤報告是由於 React Native 第三方依賴項的建置問題所導致（例如 [#39568](https://github.com/facebook/react-native/issues/39568)）。
預先建置第三方依賴項讓我們能為您處理這些建置步驟，從而避免您再遇到這些建置問題。

請注意，我們並非預先建置整個 React Native：目前僅預先建置 Meta 不直接控制的函式庫，例如 Folly 和 GLog。

在未來的版本中，我們也將把 React Native 核心的其餘部分以預先建置的形式發布。

#### 如何使用此功能

此功能目前仍處於實驗階段，因此預設未啟用。

若想使用此功能，您可透過設定 `RCT_USE_RN_DEP` 環境變數來安裝 pods：

```bash
RCT_USE_RN_DEP=1 bundle exec pod install
```

或者，若想為所有開發人員啟用此功能，可修改 Podfile 如下：

```diff
if linkage != nil
  Pod::UI.puts "Configuring Pod with #{linkage}ally linked Frameworks".green
  use_frameworks! :linkage => linkage.to_sym
end

+ENV[‘`RCT_USE_RN_DEP`’] = ‘1’

target 'HelloWorld' do
  config = use_native_modules!
```

請將預先建置可能造成的任何問題回報至[此討論串](https://github.com/react-native-community/discussions-and-proposals/discussions/912)。我們承諾會調查這些問題，並確保此功能對您的應用程式完全透明。

## 其他變更

### Android - 透過 IPO 實現更小的 APK 體積

此版本為所有使用 React Native 建置的 Android 應用程式帶來顯著的體積縮減。從 0.80 開始，我們為 React Native 和 Hermes 建置啟用了[程序間優化](https://en.wikipedia.org/wiki/Interprocedural_optimization)。

這項變更為所有 Android 應用程式節省了約 1MB 的空間。

![android apk 體積比較](/blog/assets/0.80-android-apk-size.png)

只需將 React Native 版本更新至 0.80 即可獲得此體積優化，無需對專案進行其他變更。

### 新應用程式畫面重新設計

若您未使用 Expo 而是使用社群 CLI 與範本，在此版本中我們已將新應用程式畫面移至[獨立套件](https://www.npmjs.com/package/@react-native/new-app-screen)並進行了視覺翻新。這減少了使用社群範本建立新應用程式時的初始樣板程式碼，同時在較大螢幕上提供更好的體驗。

![新應用程式畫面重新設計](/blog/assets/0.80-new-community-template-landing.jpg)

### 關於 JSC 社群支援的公告

React Native 0.80 是最後一個提供第一方 JSC 支援的版本。後續對 JSC 的支援將透過社群維護的套件 `@react-native-community/javascriptcore` 提供。

若您錯過了先前的公告，可[在此閱讀更多資訊](/blog/2025/04/08/react-native-0.79#jsc-moving-to-community-package)

## 重大變更

### 在主套件中新增 `"exports"` 欄位

As part of our JS Stable API changes, we've introduced an [`"exports"` field](https://nodejs.org/api/packages.html#package-entry-points) on the `package.json` manifest of `react-native`.

In 0.80, this mapping continues to expose **all JavaScript subpaths** by default, and therefore should not be a major breaking change. At the same time, this may subtly affect how modules within the `react-native` package are resolved:

- 在 Metro 下，[平台特定擴展](https://metrobundler.dev/docs/package-exports#replacing-platform-specific-extensions)不會自動針對 `"exports"` 匹配進行擴展。我們已提供多個墊片模組來適應此變更（[#50426](https://github.com/facebook/react-native/pull/50426)）。
- 在 Jest 下，模擬深層導入的功能可能會受到影響，這可能需要更新測試。

### 其他重大變更

以下列出我們認為可能對您的產品代碼產生輕微影響且值得注意的其他重大變更：

### JavaScript

- 我們將 `eslint-plugin-react-hooks` 從 v4.6.0 升級至 v5.2.0（完整[變更日誌在此](https://github.com/facebook/react/blob/main/packages/eslint-plugin-react-hooks/CHANGELOG.md)）。`react-hooks` 的 lint 規則可能會產生新的錯誤信號，您需要修復或抑制這些錯誤。

### Android

- This release bumps the Kotlin version shipped with React Native to version 2.1.20. Kotlin 2.1 introduces new language features in preview that you can start using in your modules/components. You can read more about it in [the official release notes](https://kotlinlang.org/docs/whatsnew21.html).
- We deleted the `StandardCharsets` class. It has been deprecated since 0.73. You should use the `java.nio.charset.StandardCharsets` class instead.
- We made several classes internal. Those classes are not part of the public API and should not be accessed. We already notified or submitted patches to the affected libraries:
  - `com.facebook.react.fabric.StateWrapperImpl`
  - `com.facebook.react.modules.core.ChoreographerCompat`
  - `com.facebook.react.modules.common.ModuleDataCleaner`
- We migrated several classes from Java to Kotlin. If you’re using those classes, the nullability and types of some parameter changed so you might need to adjust your code:
  - All the classes inside `com.facebook.react.devsupport`
  - `com.facebook.react.bridge.ColorPropConverter`
  - `com.facebook.react.views.textinput.ReactEditText`
  - `com.facebook.react.views.textinput.ReactTextInputManager`

### iOS

- 我們移除了 RCTUtils.h 中的 `RCTFloorPixelValue` 欄位 — `RCTFloorPixelValue` 方法在 React Native 中未被使用，因此我們將其完全移除。

更多較小的重大變更列於 [0.80 版的變更日誌](https://github.com/facebook/react-native/blob/main/CHANGELOG.md#v0800)中。

## 致謝

React Native 0.80 版包含來自 127 位貢獻者的超過 1167 次提交。感謝大家的辛勤工作！

<!--alex ignore special white-->

我們特別感謝在此版本中做出重要貢獻的社群成員：

- [Christian Falch](https://github.com/chrfalch) 為 React Native 依賴項的 iOS 預建工作
- [Iwo Plaza](https://x.com/iwoplaza)、[Jakub Piasecki](https://x.com/breskin67) 和 [Dawid Małecki](https://github.com/coado) 為嚴格 TypeScript API 的工作

此外，我們也感謝在此版本發布文章中記錄功能的額外作者：

- [Riccardo Cipolleschi](https://github.com/cipolleschi) 撰寫了關於 React Native 依賴項的 iOS 預建部分。
- [Alex Hunt](https://x.com/huntie) 負責深層導入棄用、選擇加入嚴格 TypeScript API、新應用程式畫面重新設計。
- [Nicola Corti](https://x.com/cortinico) 負責舊版架構凍結與警告。

## 升級至 0.80

請使用 [React Native 升級助手](https://react-native-community.github.io/upgrade-helper/) 查看現有專案在 React Native 版本間的代碼變更，並參考升級文件。

若要建立新專案：

若您使用 Expo，React Native 0.80 將在 Expo SDK 的 Canary 版本中獲得支援。關於如何在 Expo 中使用 React Native 0.80 的詳細說明，亦可參閱[專屬部落格文章](https://expo.dev/changelog/react-native-80)。

:::info

0.80 版現為 React Native 的最新穩定版本，0.77.x 系列將轉為不受支援狀態。更多資訊請參閱 React Native 的支援政策。我們預計在近期發布 0.77 版的最終終止支援更新。

:::