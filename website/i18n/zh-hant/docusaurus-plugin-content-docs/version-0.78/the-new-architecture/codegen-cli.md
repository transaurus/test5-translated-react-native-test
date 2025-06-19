# Codegen 命令行工具

手動調用 Gradle 或執行腳本可能難以記憶且需要繁瑣步驟。

為簡化流程，我們開發了 **Codegen** 命令行工具來協助執行這些任務。此命令會為您的專案執行 [@react-native/codegen](https://www.npmjs.com/package/@react-native/codegen)，提供以下選項：

```sh
npx @react-native-community/cli codegen --help
Usage: rnc-cli codegen [options]

Options:
  --verbose            Increase logging verbosity
  --path <path>        Path to the React Native project root. (default: "/Users/MyUsername/projects/my-app")
  --platform <string>  Target platform. Supported values: "android", "ios", "all". (default: "all")
  --outputPath <path>  Path where generated artifacts will be output to.
  -h, --help           display help for command
```

## 範例

- 從當前工作目錄讀取 `package.json`，根據其 codegenConfig 生成程式碼。

```shell
npx @react-native-community/cli codegen
```

- 從當前工作目錄讀取 `package.json`，在 codegenConfig 定義的位置生成 iOS 程式碼。

```shell
npx @react-native-community/cli codegen --platform ios
```

- Read `package.json` from `third-party/some-library`, generate Android code in `third-party/some-library/android/generated`.

```shell
npx @react-native-community/cli codegen \
    --path third-party/some-library \
    --platform android \
    --outputPath third-party/some-library/android/generated
```

# 將生成程式碼整合至函式庫

Codegen CLI 是函式庫開發者的利器，可預覽生成程式碼以確認需實作的介面。

通常生成程式碼不會包含在函式庫中，而是由使用該函式庫的應用程式在建置時執行 Codegen。此模式適用於多數情境，但 Codegen 也提供透過 `includesGeneratedCode` 屬性將生成程式碼直接打包進函式庫的機制。

需特別理解啟用 `includesGeneratedCode = true` 的影響。包含生成程式碼的優勢包括：

- 無需依賴應用程式執行 **Codegen**，生成程式碼已預先存在
- 實作檔案始終與生成介面保持一致（增強函式庫程式碼對 Codegen API 變更的相容性）
- Android 上無需維護兩套檔案支援新舊架構，僅需保留新架構檔案即可確保向後相容
- 由於所有原生程式碼已存在，可將函式庫原生部分預先編譯發佈

但同時需注意一項缺點：

- 生成程式碼將使用函式庫內定義的 React Native 版本。例如若函式庫基於 React Native 0.76 發佈，生成程式碼將以該版本為基礎，可能導致與使用**舊版** React Native 的應用程式（如運行於 0.75 版的應用）不相容。

## 啟用 `includesGeneratedCode`

啟用此設定的步驟：

- Add the `includesGeneratedCode` property into your library's `codegenConfig` field in the `package.json` file. Set its value to `true`.
- Run **Codegen** locally with the codegen CLI.
- Update your `package.json` to include the generated code.
- Update your `podspec` to include the generated code.
- Update your `build.Gradle` file to include the generated code.
- Update `cmakeListsPath` in `react-native.config.js` so that Gradle doesn't look for CMakeLists file in the build directory but instead in your outputDir.