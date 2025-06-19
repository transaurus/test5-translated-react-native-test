---
title: 'React Native 0.79 - Faster tooling and much more'
authors: [alanjhughes, shubham, fabriziocucci, cortinico]
tags: [engineering]
date: 2025-04-08
---

# React Native 0.79 - 更快的工具鏈、更快的啟動速度及其他改進

我們很高興宣布 React Native 0.79 正式發布！

此版本在多個方面帶來了效能改進與錯誤修復。首先，得益於延遲哈希計算，Metro 的啟動速度變得更快，並穩定支援 package exports 功能。此外，透過 JavaScript 套件壓縮方式的改變，Android 的啟動時間也將獲得提升。

### 重點更新

- [Metro 新功能](/blog/2025/04/08/react-native-0.79#metro-faster-startup-and-package-exports-support)
- [JSC 遷移至社群套件](/blog/2025/04/08/react-native-0.79#jsc-moving-to-community-package)
- [iOS：支援 Swift 的原生模組註冊](/blog/2025/04/08/react-native-0.79#ios-swift-compatible-native-modules-registration)
- [Android：更快的應用啟動](/blog/2025/04/08/react-native-0.79#android-faster-app-startup)
- [移除遠端 JavaScript 除錯功能](/blog/2025/04/08/react-native-0.79#removal-of-remote-js-debugging)

<!--truncate-->

## 重點功能詳解

### Metro：更快的啟動速度與 package exports 支援

本次發布搭載 [Metro 0.82](https://github.com/facebook/metro/releases/tag/v0.82.0)。此版本採用延遲哈希計算技術，使首次執行 `yarn start` 的速度提升超過 3 倍（在大型專案與 monorepo 中效果更顯著），大幅改善日常開發體驗與 CI 建置效率。

![metro 啟動速度比較](/blog/assets/0.79-metro-startup-comparison.gif)

Metro 0.82 同時將 `package.json` 的 `"exports"` 與 `"imports"` 欄位解析功能升級為穩定版。`"exports"` 解析功能最初[隨 React Native 0.72 推出](/blog/2023/06/21/package-exports-support)，而 `"imports"` 支援則由社群貢獻新增——這兩項功能將在 React Native 0.79 中預設啟用。

此改進增強了與現代 npm 依賴套件的相容性，並為專案組織提供了符合標準的新方式。

:::warning[Breaking change]

While we've been testing `package.json` `"exports"` in the community for a while, this switchover can be a breaking change for certain packages and project setups.

In particular, we're aware of user reported incompatibilities for some popular packages including Firebase and AWS Amplify, and are working to get these fixed at source.

If you're encountering issues:

- Please update to the Metro [0.81.5 hotfix](https://github.com/facebook/metro/releases/tag/v0.81.5), or set [`resolver.unstable_enablePackageExports = false`](https://metrobundler.dev/docs/configuration/#unstable_enablepackageexports-experimental) to opt out.
- See [expo/expo#36551](https://github.com/expo/expo/discussions/36551) for affected packages and future updates.

:::

### JSC 遷移至社群套件

作為精簡 React Native API 範疇的計畫一環，我們正將 JavaScriptCore (JSC) 引擎遷移至社群維護的套件：`@react-native-community/javascriptcore`

此變更不影響使用 Hermes 的用戶。

從 React Native 0.79 開始，您可依照[讀我檔案的安裝說明](https://github.com/react-native-community/javascriptcore#installation)使用社群支援的 JSC 版本。React Native 核心提供的 JSC 版本在 0.79 中仍可使用，但我們計畫[於近期](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0836-lean-core-jsc.md)將其移除。

將 JSC 移轉至社群維護的套件將使我們能更頻繁地更新 JSC 版本，並提供您最新功能。社群維護的 JSC 將遵循與 React Native 不同的發佈週期。

### iOS：Swift 相容的原生模組註冊

在此版本中，我們重新設計了將原生模組註冊到 React Native 運行時的方式。新方法遵循與元件相同的架構，詳見[官方文件](/docs/next/the-new-architecture/using-codegen#configuring-codegen)。

Starting from this version of React Native, you can register your modules by modifying the `package.json` file. We introduced a new `modulesProvider` field in the `ios` property:

```diff
"codegenConfig": {
     "ios": {
+       "modulesProvider": {
+         "JS Name for the module": "ObjC Module provider for the pure C++ TM or a class conforming to RCTTurboModule"
+     }
    }
}
```

Codegen 將根據您的 `package.json` 檔案自動生成所有相關程式碼。

若您使用純 C++ 原生模組，則需遵循以下建議配置：

<details>
<summary>Configure pure C++Native Modules in your app</summary>

For pure C++ Native Modules, you need to add a new ObjectiveC++ class to glue together the C++ Native Module with the rest of the App:

```objc title="CppNativeModuleProvider.h"
#import <Foundation/Foundation.h>
#import <ReactCommon/RCTTurboModule.h>

NS_ASSUME_NONNULL_BEGIN

@interface <YourNativeModule>Provider : NSObject <RCTModuleProvider>

@end
```

```objc title="CppNativeModuleProvider.mm"
NS_ASSUME_NONNULL_END

#import "<YourNativeModule>Provider.h"
#import <ReactCommon/CallInvoker.h>
#import <ReactCommon/TurboModule.h>
#import "<YourNativeModule>.h"

@implementation NativeSampleModuleProvider

- (std::shared_ptr<facebook::react::TurboModule>)getTurboModule:
    (const facebook::react::ObjCTurboModule::InitParams &)params
{
  return std::make_shared<facebook::react::NativeSampleModule>(params.jsInvoker);
}
```

</details>

透過這種新方法，我們統一了應用程式開發者與函式庫維護者的原生模組註冊流程。函式庫可在其 `package.json` 中指定相同屬性，Codegen 將處理後續工作。

此方法解決了我們在 0.77 版本中引入的限制，該限制導致無法在 Swift 實現的 `AppDelegate` 中註冊純 C++ 原生模組。如您所見，這些變更均未修改 `AppDelegate`，且生成的程式碼可同時適用於 Swift 與 Objective-C 實現的 `AppDelegate`。

### Android：更快的應用程式啟動

我們也推出了一項變更，可大幅提升 Android 應用程式的啟動速度。

從此版本開始，我們將不再壓縮 APK 內的 JavaScript 套件。先前 Android 系統需先解壓縮 JavaScript 套件才能啟動應用程式，這導致應用程式啟動時顯著減速。

自本版本起，我們預設將以未壓縮形式提供 JavaScript 套件，因此您的 Android 應用程式啟動速度將普遍提升。

The [Margelo](https://margelo.com) team tested this feature on the Discord app and got a significant performance boost: Discord’s time-to-interactive (TTI) was reduced by 400ms, which was a 12% speedup with a one-line change (tested on a Samsung A14).

On the other hand, storing the bundle uncompressed, will result in a higher space consumption for your application on the user device. If this is a concern to you, you can toggle this behavior using the `enableBundleCompression` property in your `app/build.gradle` file.

```kotlin title="app/build.gradle"
react {
  // ...
  // If you want to compress the JS bundle (slower startup, less
  // space consumption)
  enableBundleCompression = true
  // If don't you want to compress the JS bundle (faster startup,
  // higher space consumption)
  enableBundleCompression = false

  // Default is `false`
}
```

請注意，此版本中 APK 大小將會增加，但使用者在從網路下載 APK 時不會承擔額外成本，因為下載時 APK 會經過壓縮。

## 重大變更

### 移除遠端 JS 除錯功能

作為持續改進除錯體驗的一部分，我們移除了透過 Chrome 進行的遠端 JS 除錯功能。此傳統除錯方法已在 [React Native 0.73 中被棄用並改為運行時選擇加入](/blog/2023/12/06/0.73-debugging-improvements-stable-symlinks#remote-javascript-debugging)。請使用 [React Native DevTools](/docs/react-native-devtools) 進行現代化且可靠的除錯。

這也意味著 React Native 不再相容於社群專案 [react-native-debugger](https://github.com/jhen0409/react-native-debugger)。對於想使用第三方除錯擴充功能（如 Redux DevTools）的開發者，我們建議使用 [Expo DevTools 外掛](https://github.com/expo/dev-plugins)，或整合這些工具的獨立版本。

詳情請參閱[此專文](https://github.com/react-native-community/discussions-and-proposals/discussions/872)。

### 內部模組更新為 `export` 語法

作為現代化我們 JavaScript 程式碼庫的一部分，我們已更新了 `react-native` 中的多個實作模組，使其一致使用 `export` 語法而非 `module.exports`。

我們總共更新了約 **46 個 API**，詳細清單可參閱 [更新日誌](https://github.com/facebook/react-native/blob/main/CHANGELOG.md#v0790)。

此變更對現有導入方式有細微影響：

<details>
<summary>**Case 1: Default export**</summary>

```diff
  // CHANGED - require() syntax
- const ImageBackground = require('react-native/Libraries/Image/ImageBackground');
+ const ImageBackground = require('react-native/Libraries/Image/ImageBackground').default;

// Unchanged - import syntax
import ImageBackground from 'react-native/Libraries/Image/ImageBackground';

// RECOMMENDED - root import
import {ImageBackground} from 'react-native';

```

</details>

<details>

<summary>**Case 2: Secondary exports**</summary>

There are very few cases of this pattern, again unaffected when using the root `'react-native'` import.

```diff
  // Unchanged - require() syntax
  const BlobRegistry = require('react-native/Libraries/Blob/BlobRegistry');

  // Unchanged - require() syntax with destructuring
  const {register, unregister} = require('react-native/Libraries/Blob/BlobRegistry');

  // CHANGED - import syntax as single object
- import BlobRegistry from 'react-native/Libraries/Blob/BlobRegistry';
+ import * as BlobRegistry from 'react-native/Libraries/Blob/BlobRegistry';


  // Unchanged - import syntax with destructuring
  import {register, unregister} from 'react-native/Libraries/Blob/BlobRegistry';

  // RECOMMENDED - root import
  import {BlobRegistry} from 'react-native';
```

</details>

我們預期此變更的影響範圍極小，特別是對於使用 TypeScript 和 `import` 語法的專案。請檢查並修正任何類型錯誤以更新您的程式碼。

:::tip

**強烈建議使用根路徑 `react-native` 導入**

作為通用建議，我們強烈建議從根路徑 `'react-native'` 導入，以避免未來非必要的破壞性變更。在下個版本中，我們將棄用深層導入，作為明確定義 React Native 公共 JavaScript API 的一部分（參閱 [RFC 提案](https://github.com/react-native-community/discussions-and-proposals/pull/894)）。

:::

### 其他破壞性變更

以下清單列出我們預期可能對產品程式碼產生輕微影響的其他破壞性變更，值得特別注意：

- **Invalid unitless lengths in box shadows and filters**:
  - In order to make React Native more compliant with the CSS/Web specs, we now don’t support anymore unitless lengths in `box-shadow` and `filter`. This means that if you were using a `box-shadow` of `1 1 black` we won’t be rendering. You should instead specify units such as `1px 1px black`
- **Remove incorrect hwb() syntax support from normalize-color:**
  - In order to make React Native more compliant with the CSS/Web specs, we now restrict some invalid syntax for `hwb()`. Historically React Native used to support comma separated values (e.g. `hwb(0, 0%, 100%)`) which we now don’t support anymore (you should migrate to `hwb(0 0% 100%)`). You can read more about this change [here](https://github.com/facebook/react-native/commit/676359efd9e478d69ad430cff213acc87b273580).
- **Libraries/Core/ExceptionsManager exports update**
  - As part of our effort to modernize the React Native JS API, we updated <code>[ExceptionsManager](https://github.com/facebook/react-native/blob/0.79-stable/packages/react-native/Libraries/Core/ExceptionsManager.js)</code> to now export a default `ExceptionsManager` object, and `SyntheticError` as a secondary export.

## 致謝

React Native 0.79 版本包含來自 100 位貢獻者的超過 944 次提交。感謝所有人的辛勤付出！

我們特別感謝在此版本中貢獻重要功能的社群成員：

- [Marc Rousavy](https://github.com/mrousavy) 開發並撰寫了「Android: 更快的應用啟動」功能
- [Kudo Chien](https://github.com/Kudo) 與 [Oskar Kwaśniewski](https://github.com/okwasniewski) 負責 `@react-native-community/javascriptcore` 套件及「JSC 遷移至社群套件」章節
- [James Lawson](https://github.com/facebook/metro/pull/1302) 在 [Metro](https://github.com/facebook/metro/pull/1302) 中新增了子路徑解析支援

此外，我們也感謝為此版本說明文件貢獻內容的作者們：

- [Rob Hogan](https://github.com/robhogan) 撰寫「新 Metro 功能」章節
- [Alex Hunt](https://github.com/huntie) 負責「移除遠端 JS 除錯」與「內部模組更新為 export 語法」章節
- [Riccardo Cipolleschi](https://github.com/cipolleschi) 完成 iOS 原生模組註冊相關工作

## 升級至 0.79 版本

請使用 [React Native Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) 查看現有專案在 React Native 版本間的程式碼變更，並參閱升級文件。

若要建立新專案：

```sh
npx @react-native-community/cli@latest init MyProject --version latest
```

若您使用 Expo，React Native 0.79 將在即將發布的 Expo SDK 53 中作為預設的 React Native 版本獲得支援。

:::info

0.79 現為 React Native 的最新穩定版本，而 0.76.x 已轉為不受支援。更多資訊請參閱 [React Native 的支援政策](https://github.com/reactwg/react-native-releases/blob/main/docs/support.md)。我們預計在近期發布 0.76 的最終終止支援更新。

:::