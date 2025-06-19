---
id: react-native-gradle-plugin
title: React Native Gradle Plugin
---

本指南說明如何配置 **React Native Gradle 插件**（通常簡稱 RNGP），以建構 Android 平台的 React Native 應用程式。

## 使用插件

React Native Gradle 插件以獨立的 NPM 套件形式發佈，會隨 `react-native` 自動安裝。

The plugin is **already configured** for new projects created using `npx react-native init`. You don't need to do any extra steps to install it if you created your app with this command.

若需將 React Native 整合至現有專案，請參閱[對應頁面](/docs/next/integration-with-existing-apps#configuring-gradle)，該文件包含插件安裝的具體指引。

## 配置插件

插件預設採用**開箱即用**的合理配置。僅在需要調整行為時，才需參考本指南進行自訂。

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

此參數指向 React Native 專案的根目錄（即存放 `package.json` 的資料夾）。預設值為 `..`。可依下列方式自訂：

```groovy
root = file("../")
```

### `reactNativeDir`

此參數指向存放 `react-native` 套件的資料夾。預設值為 `../node_modules/react-native`。
若處於 monorepo 環境或使用不同套件管理器，可調整 `reactNativeDir` 以符合專案結構。

自訂方式如下：

```groovy
reactNativeDir = file("../node_modules/react-native")
```

### `codegenDir`

此參數指向存放 `react-native-codegen` 套件的資料夾。預設值為 `../node_modules/react-native-codegen`。
若處於 monorepo 環境或使用不同套件管理器，可調整 `codegenDir` 以符合專案結構。

自訂方式如下：

```groovy
codegenDir = file("../node_modules/@react-native/codegen")
```

### `cliFile`

此參數指向 React Native CLI 的進入點檔案。預設值為 `../node_modules/react-native/cli.js`。
插件需透過此檔案呼叫 CLI 來執行打包與應用程式建構。

若處於 monorepo 環境或使用不同套件管理器，可調整 `cliFile` 以符合專案結構。
自訂方式如下：

```groovy
cliFile = file("../node_modules/react-native/cli.js")
```

### `debuggableVariants`

此參數列出可偵錯的變體（關於變體的詳細說明請參閱[使用變體](#using-variants)）。

By default the plugin is considering as `debuggableVariants` only `debug`, while `release` is not. If you have other
variants (like `staging`, `lite`, etc.) you'll need to adjust this accordingly.

列為 `debuggableVariants` 的變體不會包含預編譯套件包，需依賴 Metro 運行。

自訂方式如下：

```groovy
debuggableVariants = ["liteDebug", "prodDebug"]
```

### `nodeExecutableAndArgs`

此參數定義執行所有腳本時呼叫的 node 指令與參數列表。預設值為 `[node]`，可依下列方式添加額外旗標：

```groovy
nodeExecutableAndArgs = ["node"]
```

### `bundleCommand`

這是建立應用程式套件時要呼叫的 `bundle` 指令名稱。如果您使用 [RAM Bundles](https://reactnative.dev/docs/0.74/ram-bundles-inline-requires)，這會很有用。預設為 `bundle`，但可以透過以下方式自訂以添加額外參數：

```groovy
bundleCommand = "ram-bundle"
```

### `bundleConfig`

這是傳遞給 `bundle --config <file>` 的設定檔案路徑（若提供）。預設為空（不提供設定檔）。有關套件設定檔的更多資訊，請參閱 [CLI 文件](https://github.com/react-native-community/cli/blob/main/docs/commands.md#bundle)。可透過以下方式自訂：

```groovy
bundleConfig = file(../rn-cli.config.js)
```

### `bundleAssetName`

這是應生成的套件檔案名稱。預設為 `index.android.bundle`。可透過以下方式自訂：

```groovy
bundleAssetName = "MyApplication.android.bundle"
```

### `entryFile`

用於套件生成的入口檔案。預設會搜尋 `index.android.js` 或 `index.js`。可透過以下方式自訂：

```groovy
entryFile = file("../js/MyApplication.android.js")
```

### `extraPackagerArgs`

傳遞給 `bundle` 指令的額外參數清單。可用參數清單請參閱 [CLI 文件](https://github.com/react-native-community/cli/blob/main/docs/commands.md#bundle)。預設為空。可透過以下方式自訂：

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

### `enableBundleCompression`

套件資源在打包成 `.apk` 時是否應壓縮。

停用 `.bundle` 的壓縮可讓其直接映射到 RAM 中，從而改善啟動時間，但會增加磁碟上的應用程式大小。請注意，`.apk` 的下載大小幾乎不受影響，因為 `.apk` 檔案在下載前會先壓縮。

預設情況下此功能是停用的，除非您非常關注應用程式的磁碟空間，否則不應啟用它。

## 使用風味與建置變體

在建置 Android 應用程式時，您可能會想使用 [自訂風味](https://developer.android.com/studio/build/build-variants#product-flavors) 來從同一個專案產生不同版本的應用程式。

請參閱 [官方 Android 指南](https://developer.android.com/studio/build/build-variants) 來設定自訂建置類型（如 `staging`）或自訂風味（如 `full`、`lite` 等）。
預設情況下，新建立的應用程式會有兩種建置類型（`debug` 和 `release`）且沒有自訂風味。

所有建置類型與風味的組合會產生一組 **建置變體**。例如，對於 `debug`/`staging`/`release` 建置類型和 `full`/`lite` 風味，您會有 6 種建置變體：`fullDebug`、`fullStaging`、`fullRelease` 等等。

If you're using custom variants beyond `debug` and `release`, you need to instruct the React Native Gradle Plugin specifying which of your variants are **debuggable** using the [`debuggableVariants`](#debuggablevariants) configuration as follows:

```diff
apply plugin: "com.facebook.react"

react {
+ debuggableVariants = ["fullStaging", "fullDebug"]
}
```

This is necessary because the plugin will skip the JS bundling for all the `debuggableVariants`: you'll need Metro to run them. For example, if you list `fullStaging` in the `debuggableVariants`, you won't be able to publish it to a store as it will be missing the bundle.

## 插件在底層做了什麼？

React Native Gradle 插件負責配置你的應用程式構建，以將 React Native 應用程式發布到生產環境。該插件也用於第三方函式庫中，以運行用於新架構的 [Codegen](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/codegen.md)。

以下是插件的主要職責概述：

- Add a `createBundle<Variant>JsAndAssets` task for every non debuggable variant, that is responsible of invoking the `bundle`, `hermesc` and `compose-source-map` commands.
- Setting up the proper version of the `com.facebook.react:react-android` and `com.facebook.react:hermes-android` dependency, reading the React Native version from the `package.json` of `react-native`.
- Setting up the proper Maven repositories (Maven Central, Google Maven Repo, JSC local Maven repo, etc.) needed to consume all the necessary Maven Dependencies.
- Setting up the NDK to let you build apps that are using the New Architecture.
- Setting up the `buildConfigFields` so that you can know at runtime if Hermes or the New Architecture are enabled.
- Setting up the Metro DevServer Port as an Android resource so the app knows on which port to connect.
- Invoking the [React Native Codegen](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/codegen.md) if a library or app is using the Codegen for the New Architecture.