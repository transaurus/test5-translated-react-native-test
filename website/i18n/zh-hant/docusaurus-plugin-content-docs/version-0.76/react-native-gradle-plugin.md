---
id: react-native-gradle-plugin
title: React Native Gradle Plugin
---

本指南說明如何配置 **React Native Gradle 插件**（通常簡稱為 RNGP），以便為 Android 構建 React Native 應用程序。

## 使用插件

React Native Gradle 插件作為一個獨立的 NPM 包分發，安裝 `react-native` 時會自動安裝。

The plugin is **already configured** for new projects created using `npx react-native init`. You don't need to do any extra steps to install it if you created your app with this command.

如果您正在將 React Native 集成到現有項目中，請參考[對應頁面](/docs/next/integration-with-existing-apps#configuring-gradle)：其中包含有關如何安裝插件的具體說明。

## 配置插件

默認情況下，插件將**開箱即用**，並採用合理的默認設置。只有在需要時，才應參考本指南並自定義其行為。

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

每個配置鍵的說明如下：

### `root`

這是您的 React Native 項目的根文件夾，即 `package.json` 文件所在的位置。默認為 `..`。您可以按如下方式自定義它：

```groovy
root = file("../")
```

### `reactNativeDir`

這是 `react-native` 包所在的文件夾。默認為 `../node_modules/react-native`。
如果您使用的是單倉庫（monorepo）或不同的包管理器，可以根據您的設置調整 `reactNativeDir`。

您可以按如下方式自定義它：

```groovy
reactNativeDir = file("../node_modules/react-native")
```

### `codegenDir`

這是 `react-native-codegen` 包所在的文件夾。默認為 `../node_modules/react-native-codegen`。
如果您使用的是單倉庫或不同的包管理器，可以根據您的設置調整 `codegenDir`。

您可以按如下方式自定義它：

```groovy
codegenDir = file("../node_modules/@react-native/codegen")
```

### `cliFile`

這是 React Native CLI 的入口文件。默認為 `../node_modules/react-native/cli.js`。
插件需要調用 CLI 來打包和創建您的應用程序，因此需要此入口文件。

如果您使用的是單倉庫或不同的包管理器，可以根據您的設置調整 `cliFile`。
您可以按如下方式自定義它：

```groovy
cliFile = file("../node_modules/react-native/cli.js")
```

### `debuggableVariants`

這是可調試的變體列表（有關變體的更多上下文，請參見[使用變體](#using-variants)）。

By default the plugin is considering as `debuggableVariants` only `debug`, while `release` is not. If you have other
variants (like `staging`, `lite`, etc.) you'll need to adjust this accordingly.

列為 `debuggableVariants` 的變體不會附帶打包好的 bundle，因此您需要運行 Metro 來使用它們。

您可以按如下方式自定義它：

```groovy
debuggableVariants = ["liteDebug", "prodDebug"]
```

### `nodeExecutableAndArgs`

這是應為所有腳本調用的 node 命令和參數列表。默認為 `[node]`，但可以按如下方式自定義以添加額外標誌：

```groovy
nodeExecutableAndArgs = ["node"]
```

### `bundleCommand`

這是建立應用程式套件時要呼叫的 `bundle` 指令名稱。若您使用 [RAM Bundles](https://reactnative.dev/docs/0.74/ram-bundles-inline-requires) 時會很有用。預設值為 `bundle`，但可透過以下方式自訂額外參數：

```groovy
bundleCommand = "ram-bundle"
```

### `bundleConfig`

此為傳遞給 `bundle --config <file>` 的設定檔路徑（若提供）。預設為空（不提供設定檔）。更多關於套件設定檔的資訊可參閱 [CLI 文件](https://github.com/react-native-community/cli/blob/main/docs/commands.md#bundle)。可透過以下方式自訂：

```groovy
bundleConfig = file(../rn-cli.config.js)
```

### `bundleAssetName`

此為應生成的套件檔案名稱。預設為 `index.android.bundle`。可透過以下方式自訂：

```groovy
bundleAssetName = "MyApplication.android.bundle"
```

### `entryFile`

用於生成套件的進入點檔案。預設會搜尋 `index.android.js` 或 `index.js`。可透過以下方式自訂：

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

## 使用風味與建置變體

建置 Android 應用程式時，您可能會想使用 [自訂風味](https://developer.android.com/studio/build/build-variants#product-flavors) 來從同個專案產生不同版本的應用程式。

請參閱 [官方 Android 指南](https://developer.android.com/studio/build/build-variants) 來設定自訂建置類型（如 `staging`）或自訂風味（如 `full`、`lite` 等）。
新建立的應用程式預設會有兩種建置類型（`debug` 和 `release`）且無自訂風味。

所有建置類型與風味的組合會產生一組 **建置變體**。例如針對 `debug`/`staging`/`release` 建置類型與 `full`/`lite` 風味，您將有 6 種建置變體：`fullDebug`、`fullStaging`、`fullRelease` 等。

If you're using custom variants beyond `debug` and `release`, you need to instruct the React Native Gradle Plugin specifying which of your variants are **debuggable** using the [`debuggableVariants`](#debuggablevariants) configuration as follows:

```diff
apply plugin: "com.facebook.react"

react {
+ debuggableVariants = ["fullStaging", "fullDebug"]
}
```

This is necessary because the plugin will skip the JS bundling for all the `debuggableVariants`: you'll need Metro to run them. For example, if you list `fullStaging` in the `debuggableVariants`, you won't be able to publish it to a store as it will be missing the bundle.

## 插件在底層做了什麼？

React Native Gradle 插件負責設定您的應用程式建置，以將 React Native 應用程式發布至生產環境。
該插件也用於第三方函式庫中，以執行新架構使用的 [Codegen](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/codegen.md)。

以下是該插件的主要職責概述：

- Add a `createBundle<Variant>JsAndAssets` task for every non debuggable variant, that is responsible of invoking the `bundle`, `hermesc` and `compose-source-map` commands.
- Setting up the proper version of the `com.facebook.react:react-android` and `com.facebook.react:hermes-android` dependency, reading the React Native version from the `package.json` of `react-native`.
- Setting up the proper Maven repositories (Maven Central, Google Maven Repo, JSC local Maven repo, etc.) needed to consume all the necessary Maven Dependencies.
- Setting up the NDK to let you build apps that are using the New Architecture.
- Setting up the `buildConfigFields` so that you can know at runtime if Hermes or the New Architecture are enabled.
- Setting up the Metro DevServer Port as an Android resource so the app knows on which port to connect.
- Invoking the [React Native Codegen](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/codegen.md) if a library or app is using the Codegen for the New Architecture.