---
id: bundled-hermes
title: Bundled Hermes
---

This page gives an overview of **how** Hermes and React Native **are built**.

若您需要關於如何在應用程式中使用 Hermes 的指引，請參閱另一頁面：[使用 Hermes](/docs/hermes)

:::caution
請注意，本頁面屬於技術深度解析，主要針對在 Hermes 或 React Native 之上開發擴充功能的用戶。React Native 的一般使用者無需深入了解 React Native 與 Hermes 的互動機制。
:::

## 何謂「內建 Hermes」

自 React Native 0.69.0 起，每個 React Native 版本都將**同步建置**對應的 Hermes 版本。我們稱此分發模式為**內建 Hermes**。

從 0.69 開始，您將始終獲得一個與 React Native 版本同步建置並通過測試的 JavaScript 引擎。

## 為何轉向「內建 Hermes」

過去，React Native 與 Hermes 遵循**兩套獨立的發佈流程**與版本號體系。這種雙軌制導致開源生態系產生混淆——使用者難以判斷特定 Hermes 版本是否相容特定 React Native 版本（例如需知曉 Hermes 0.11.0 僅相容 React Native 0.68.0 等）。

Hermes 與 React Native 共享 JSI 程式碼（[Hermes 端](https://github.com/facebook/hermes/tree/main/API/jsi/jsi)與 [React Native 端](https://github.com/facebook/react-native/tree/main/packages/react-native/ReactCommon/jsi/jsi)）。若兩者的 JSI 版本不同步，建置出的 Hermes 將無法與 React Native 相容。您可於[此處](https://github.com/react-native-community/discussions-and-proposals/issues/257)閱讀更多關於 ABI 相容性問題的說明。

為解決此問題，我們擴展了 React Native 發佈流程，使其能下載並建置 Hermes，同時確保建置 Hermes 時僅使用單一 JSI 程式碼副本。

Thanks to this, we can release a version of Hermes whenever we release a version of React Native, and be sure that the Hermes engine we built is **fully compatible** with the React Native version we're releasing. We're shipping this version of Hermes alongside the React Native version we're doing, hence the name _Bundled Hermes_.

## 對應用程式開發者的影響

如前言所述，若您是應用程式開發者，此變更**不應直接影響**您的工作流程。

以下段落將說明我們在底層實施的變更及其設計考量，以保持透明度。

### iOS 使用者

在 iOS 端，我們調整了您所使用的 `hermes-engine` 來源。

在 React Native 0.69 之前，使用者需下載 pod（參見此處的 [podspec](https://github.com/CocoaPods/Specs/blob/master/Specs/5/d/0/hermes-engine/0.11.0/hermes-engine.podspec.json)）。

On React Native 0.69, users would instead use a podspec that is defined inside the `sdks/hermes-engine/hermes-engine.podspec` file in the `react-native` NPM package.
That podspec relies on a pre-built tarball of Hermes that we upload to Maven and to the React Native GitHub Release, as part of the React Native release process (i.e. [see the assets of this release](https://github.com/facebook/react-native/releases/tag/v0.70.4)).

### Android 使用者

在 Android 端，我們將更新預設模板中的 [`android/app/build.gradle`](https://github.com/facebook/react-native/blob/main/template/android/app/build.gradle) 檔案如下：

```diff
dependencies {
    // ...

    if (enableHermes) {
+       implementation("com.facebook.react:hermes-engine:+") {
+           exclude group:'com.facebook.fbjni'
+       }
-       def hermesPath = "../../node_modules/hermes-engine/android/";
-       debugImplementation files(hermesPath + "hermes-debug.aar")
-       releaseImplementation files(hermesPath + "hermes-release.aar")
    } else {
        implementation jscFlavor
    }
}
```

Prior to React Native 0.69, users will be consuming `hermes-debug.aar` and `hermes-release.aar` from the `hermes-engine` NPM package.

On React Native 0.69, users will be consuming the Android multi-variant artifacts available inside the `android/com/facebook/react/hermes-engine/` folder in the `react-native` NPM package.
Please also note that we're going to [remove the dependency](https://github.com/facebook/react-native/blob/c418bf4c8fe8bf97273e3a64211eaa38d836e0a0/package.json#L105) on `hermes-engine` entirely in one of the future version of React Native.

#### 採用新架構的 Android 使用者

由於我們原生程式碼建置設定的性質（即我們如何使用 NDK），採用新架構的使用者將**從原始碼建置 Hermes**。

這讓採用新架構的使用者在建置機制上對 React Native 和 Hermes 保持一致（他們將從原始碼建置這兩個框架）。
這意味著這些 Android 使用者在首次建置時可能會經歷建置時間上的效能影響。

您可以在這個頁面找到優化建置時間並減少對建置影響的指引：[加速建置階段](/docs/next/build-speed)。

#### 在 Windows 上採用新架構建置的 Android 使用者

在 Windows 機器上使用新架構建置 React Native 應用程式的使用者需要遵循以下額外步驟以確保建置正確運作：

- Make sure the [environment is configured properly](https://reactnative.dev/docs/environment-setup), with Android SDK & node.
- Install [cmake](https://community.chocolatey.org/packages/cmake) with Chocolatey
- Install either:
  - [Build Tools for Visual Studio 2022](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2022).
  - [Visual Studio 22 Community Edition](https://visualstudio.microsoft.com/vs/community/) - Picking only the C++ desktop development is sufficient.
- Make sure the [Visual Studio Command Prompt](https://docs.microsoft.com/en-us/visualstudio/ide/reference/command-prompt-powershell?view=vs-2022) is configured correctly. This is required as the proper C++ compiler environment variable is configured in those command prompt.
- Run the app with `npx react-native run-android` inside a Visual Studio Command Prompt.

### 使用者是否仍可使用其他引擎？

是的，使用者可以自由啟用/停用 Hermes（在 Android 上使用 `enableHermes` 變數，在 iOS 上使用 `hermes_enabled`）。
「捆綁式 Hermes」的變更僅會影響**Hermes 的建置和捆綁方式**。

從 React Native 0.70 開始，`enableHermes`/`hermes_enabled` 的預設值為 `true`。

## 這將如何影響貢獻者和擴展開發者

如果您是 React Native 的貢獻者，或正在基於 React Native 或 Hermes 開發擴展，請繼續閱讀，我們將解釋捆綁式 Hermes 的運作方式。

### 捆綁式 Hermes 在底層是如何運作的？

This mechanism relies on **downloading a tarball** with the Hermes source code from the `facebook/hermes` repository inside the `facebook/react-native` repository. We have a similar mechanism in place for other native dependencies (Folly, Glog, etc.) and we aligned Hermes to follow the same setup.

When building React Native from `main`, we will be fetching a tarball of `main` of facebook/hermes and building it as part of the build process of React Native.

當從發行分支（例如 `0.69-stable`）建置 React Native 時，我們將改為使用 Hermes 儲存庫中的一個**標籤**來**同步兩個儲存庫之間的程式碼**。所使用的特定標籤名稱將儲存在 React Native 發行分支中的 `sdks/.hermesversion` 檔案中（例如，這是 0.69 發行分支上的[該檔案](https://github.com/facebook/react-native/blob/0.69-stable/sdks/.hermesversion)）。

從某種意義上說，您可以將這種方法視為類似於 **git 子模組**。

如果您是基於 Hermes 進行開發，可以依賴這些標籤來理解在構建 React Native 時使用了哪個版本的 Hermes，因為標籤名稱中會指定 React Native 的版本（例如 `hermes-2022-05-20-RNv0.69.0-ee8941b8874132b8f83e4486b63ed5c19fc3f111`）。

#### Android 實作細節

為了在 Android 上實作這一點，我們在 React Native 的 `/ReactAndroid/hermes-engine` 中新增了一個構建任務，負責構建 Hermes 並打包以供使用（[更多背景請參閱此處](https://github.com/facebook/react-native/pull/33396)）。

現在您可以透過以下指令觸發 Hermes 引擎的構建：

```bash
// Build a debug version of Hermes
./gradlew :ReactAndroid:hermes-engine:assembleDebug
// Build a release version of Hermes
./gradlew :ReactAndroid:hermes-engine:assembleRelease
```

（此指令適用於 React Native 的 `main` 分支）。

您無需在機器上安裝額外工具（如 `cmake`、`ninja` 或 `python3`），因為我們已配置構建過程使用 NDK 版本的這些工具。

在 Gradle 消費端，我們也進行了一項小改進：從 `releaseImplementation` 和 `debugImplementation` 改為使用 `implementation`。這是可行的，因為新版 `hermes-engine` Android 工件具有**變體感知**能力，能正確將引擎的 debug 構建與您應用的 debug 構建匹配。您無需在此進行任何自定義配置（即使您使用 `staging` 或其他建置類型/風味）。

然而，這使得模板中需要加入這一行：

```
exclude group:'com.facebook.fbjni'
```

這是必要的，因為 React Native 是透過非 prefab 方式使用 `fbjni`（即解壓 `.aar` 並提取 `.so` 文件）。而 Hermes-engine 和其他函式庫則使用 prefab 來消費 fbjni。我們正在研究[解決此問題](https://github.com/facebook/react-native/pull/33397)，未來 Hermes 的導入將只需一行代碼。

#### iOS 實作細節

iOS 的實作依賴於位於以下位置的一系列腳本：

- [`/scripts/hermes`](https://github.com/facebook/react-native/tree/main/scripts/hermes). Those scripts contain logic to download the Hermes tarball, unzip it, and configure the iOS build. They're invoked at `pod install` time if you have the `hermes_enabled` field set to `true`.
- [`/sdks/hermes-engine`](https://github.com/facebook/react-native/tree/main/sdks/hermes-engine). Those scripts contain the build logic that is effectively building Hermes. They were copied and adapted from the `facebook/hermes` repo to properly work within React Native. Specifically, the scripts inside the `utils` folder are responsible of building Hermes for all the Mac platforms.

Hermes is built as part of the `build_hermes_macos` Job on CircleCI. The job will produce as artifact a tarball which will be downloaded by the `hermes-engine` podspec when using a published React Native release ([here is an example of the artifacts created for React Native 0.69 in `build_hermes_macos`](https://app.circleci.com/pipelines/github/facebook/react-native/13679/workflows/5172f8e4-6b02-4ccb-ab97-7cb954911fae/jobs/258701/artifacts)).

##### 預構建的 Hermes

If there are no prebuilt artifacts for the React Native version that is being used (i.e. you may be working with React Native from the `main` branch), then Hermes will need to be built from source. First, the Hermes compiler, `hermesc`, will be built for macOS during `pod install`, then Hermes itself will be built as part of the Xcode build pipeline using the `build-hermes-xcode.sh` script.

##### 從原始碼構建 Hermes

當使用 React Native 的 `main` 分支時，Hermes 總是從原始碼構建。如果您使用的是穩定版的 React Native，可以透過在 CocoaPods 使用時設定環境變數 `CI` 為 `true` 來強制從原始碼構建 Hermes：`CI=true pod install`。

##### 除錯符號

預構建的 Hermes 成品預設不包含除錯符號（dSYMs）。我們計劃在未來為每個版本分發這些除錯符號。在此之前，如果您需要 Hermes 的除錯符號，則需要從原始碼構建 Hermes。在構建目錄中，與每個 Hermes 框架一起會生成一個 `hermes.framework.dSYM`。

### 我擔心這個變更會影響到我

We'd like to stress that this is essentially an organizational change on _where_ Hermes is built and _how_ the code is syncronized between the two repositories. The change should be fully transparent to our users.

過去，我們會為特定版本的 React Native 發布一個 Hermes 版本（例如 [`v0.11.0 for RN0.68.x`](https://github.com/facebook/hermes/releases/tag/v0.11.0)）。

使用「捆綁式 Hermes」，您可以依賴一個標籤來代表特定版本 React Native 發布時所使用的版本。