---
id: react-native-gradle-plugin
title: React Native Gradle Plugin
---

本指南說明如何配置 **React Native Gradle 插件**（通常簡稱 RNGP），以便為 Android 構建 React Native 應用程式。

## 使用插件

React Native Gradle 插件作為獨立的 NPM 套件分發，會隨 `react-native` 自動安裝。

The plugin is **already configured** for new projects created using `npx react-native init`. You don't need to do any extra steps to install it if you created your app with this command.

若需將 React Native 整合至現有專案，請參閱[對應頁面](/docs/next/integration-with-existing-apps#configuring-gradle)，其中包含插件安裝的具體指引。

## 配置插件

插件預設採用**開箱即用**的合理配置。僅在需要時才需參考本指南調整行為。

To configure the plugin you can modify the `react` block, inside your `android/app/build.gradle`:

```groovy
apply plugin: "com.facebook.react"

/**
 * This is the configuration block to customize your React Native Android app.
 * By default you don't need to apply any configuration, just uncomment the lines you need.
 */
react {
  // Custom configuration goes here.
}
```

各配置參數說明如下：

### `root`

此為 React Native 專案的根目錄（即存放 `package.json` 的資料夾）。預設為 `..`。可依以下方式自訂：

```groovy
root = file("../")
```

### `reactNativeDir`

此為存放 `react-native` 套件的資料夾。預設為 `../node_modules/react-native`。
若使用 monorepo 或其他套件管理器，可調整 `reactNativeDir` 以符合您的架構。

自訂方式如下：

```groovy
reactNativeDir = file("../node_modules/react-native")
```

### `codegenDir`

此為存放 `react-native-codegen` 套件的資料夾。預設為 `../node_modules/react-native-codegen`。
若使用 monorepo 或其他套件管理器，可調整 `codegenDir` 以符合您的架構。

自訂方式如下：

```groovy
codegenDir = file("../node_modules/@react-native/codegen")
```

### `cliFile`

此為 React Native CLI 的入口檔案。預設為 `../node_modules/react-native/cli.js`。
插件需透過此入口檔案呼叫 CLI 來打包及建立應用程式。

若使用 monorepo 或其他套件管理器，可調整 `cliFile` 以符合您的架構。
自訂方式如下：

```groovy
cliFile = file("../node_modules/react-native/cli.js")
```

### `debuggableVariants`

此為可除錯的變體清單（關於變體的詳細說明請參閱[使用變體](#using-variants)）。

By default the plugin is considering as `debuggableVariants` only `debug`, while `release` is not. If you have other
variants (like `staging`, `lite`, etc.) you'll need to adjust this accordingly.

列為 `debuggableVariants` 的變體不會包含預先打包的 bundle，因此需透過 Metro 執行。

自訂方式如下：

```groovy
debuggableVariants = ["liteDebug", "prodDebug"]
```

### `nodeExecutableAndArgs`

此為執行所有腳本時應呼叫的 node 指令及其參數清單。預設為 `[node]`，但可添加額外參數如下：

```groovy
nodeExecutableAndArgs = ["node"]
```

### `bundleCommand`

這是建立應用程式套件時要呼叫的 `bundle` 指令名稱。若您使用 [RAM Bundles](https://reactnative.dev/docs/0.74/ram-bundles-inline-requires) 時會很有用。預設為 `bundle`，但可透過以下方式自訂並添加額外參數：

```groovy
bundleCommand = "ram-bundle"
```

### `bundleConfig`

此為傳遞給 `bundle --config <file>` 的設定檔路徑（若提供）。預設為空（不提供設定檔）。更多關於套件設定檔的資訊請參閱 [CLI 文件](https://github.com/react-native-community/cli/blob/main/docs/commands.md#bundle)。可透過以下方式自訂：

```groovy
bundleConfig = file(../rn-cli.config.js)
```

### `bundleAssetName`

此為應生成的套件檔案名稱。預設為 `index.android.bundle`。可透過以下方式自訂：

```groovy
bundleAssetName = "MyApplication.android.bundle"
```

### `entryFile`

用於生成套件的入口檔案。預設會搜尋 `index.android.js` 或 `index.js`。可透過以下方式自訂：

```groovy
entryFile = file("../js/MyApplication.android.js")
```

### `extraPackagerArgs`

傳遞給 `bundle` 指令的額外參數清單。可用參數列表請見 [CLI 文件](https://github.com/react-native-community/cli/blob/main/docs/commands.md#bundle)。預設為空。可透過以下方式自訂：

```groovy
extraPackagerArgs = []
```

### `hermesCommand`

The path to the `hermesc` command (the Hermes Compiler). React Native comes with a version of the Hermes compiler bundled with it, so you generally won't be needing to customize this. The plugin will use the correct compiler for your system by default.

### `hermesFlags`

傳遞給 `hermesc` 的參數清單。預設為 `["-O", "-output-source-map"]`。可透過以下方式自訂：

```groovy
hermesFlags = ["-O", "-output-source-map"]
```

## 使用風味與建置變體

建置 Android 應用時，您可能會想使用 [自訂風味](https://developer.android.com/studio/build/build-variants#product-flavors) 來從同個專案產生不同版本的應用程式。

請參閱 [官方 Android 指南](https://developer.android.com/studio/build/build-variants) 來設定自訂建置類型（如 `staging`）或自訂風味（如 `full`、`lite` 等）。
新建立的應用程式預設會有兩種建置類型（`debug` 和 `release`）且無自訂風味。

所有建置類型與風味的組合會產生一組 **建置變體**。例如對於 `debug`/`staging`/`release` 建置類型與 `full`/`lite` 風味，您將有 6 種建置變體：`fullDebug`、`fullStaging`、`fullRelease` 等。

If you're using custom variants beyond `debug` and `release`, you need to instruct the React Native Gradle Plugin specifying which of your variants are **debuggable** using the [`debuggableVariants`](#debuggablevariants) configuration as follows:

```diff
apply plugin: "com.facebook.react"

react {
+ debuggableVariants = ["fullStaging", "fullDebug"]
}
```

此步驟是必要的，因為插件會跳過所有 `debuggableVariants` 的 JS 套件生成：您需要 Metro 來執行它們。例如，若您將 `fullStaging` 列入 `debuggableVariants`，將無法將其發布至商店，因為它會缺少套件。

## 插件在底層做了什麼？

React Native Gradle 插件負責設定您的應用程式建置，以將 React Native 應用程式發布至生產環境。
該插件也用於第三方函式庫中，以執行新架構所需的 [Codegen](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/codegen.md)。

以下是該插件的主要職責概述：

- Add a `createBundle<Variant>JsAndAssets` task for every non debuggable variant, that is responsible of invoking the `bundle`, `hermesc` and `compose-source-map` commands.
- Setting up the proper version of the `com.facebook.react:react-android` and `com.facebook.react:hermes-android` dependency, reading the React Native version from the `package.json` of `react-native`.
- Setting up the proper Maven repositories (Maven Central, Google Maven Repo, JSC local Maven repo, etc.) needed to consume all the necessary Maven Dependencies.
- Setting up the NDK to let you build apps that are using the New Architecture.
- Setting up the `buildConfigFields` so that you can know at runtime if Hermes or the New Architecture are enabled.
- Setting up the Metro DevServer Port as an Android resource so the app knows on which port to connect.
- Invoking the [React Native Codegen](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/codegen.md) if a library or app is using the Codegen for the New Architecture.