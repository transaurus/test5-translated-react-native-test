# Codegen 命令行工具

手動調用 Gradle 或執行腳本可能難以記憶且流程繁瑣。

為簡化流程，我們開發了可協助執行這些任務的 CLI 工具：**Codegen** 命令行工具。此命令會為您的專案執行 [@react-native/codegen](https://www.npmjs.com/package/@react-native/codegen)。以下是可用的選項：

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

- 從當前工作目錄讀取 `package.json`，並根據其 codegenConfig 生成程式碼。

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

## 將生成程式碼納入函式庫

Codegen CLI 是函式庫開發者的絕佳工具，可用於預覽生成的程式碼以確認需實作的介面。

通常生成的程式碼不會包含在函式庫中，而是由使用該函式庫的應用程式負責在建置時執行 Codegen。這對多數情況是合適的配置，但 Codegen 也提供透過 `includesGeneratedCode` 屬性將生成程式碼直接包含在函式庫內的機制。

需特別理解啟用 `includesGeneratedCode = true` 的影響。包含生成程式碼具有以下優勢：

- 無需依賴應用程式為您執行 **Codegen**，生成的程式碼始終存在。
- 實作檔案與生成介面保持同步（使函式庫程式碼更能抵禦 codegen 的 API 變更）。
- 無需在 Android 上維護兩套檔案來支援兩種架構。僅需保留新架構檔案，且保證向後兼容。
- 由於所有原生程式碼均已包含，可將函式庫的原生部分預先建置後發布。

但同時也需注意一項缺點：

- 生成的程式碼將使用函式庫內部定義的 React Native 版本。例如若函式庫基於 React Native 0.76 發布，生成程式碼將以該版本為基礎。這可能導致生成的程式碼與應用程式使用的**較舊** React Native 版本（如運行於 React Native 0.75 的應用）不相容。

## 啟用 `includesGeneratedCode`

啟用此配置的步驟：

- Add the `includesGeneratedCode` property into your library's `codegenConfig` field in the `package.json` file. Set its value to `true`.
- Run **Codegen** locally with the codegen CLI.
- Update your `package.json` to include the generated code.
- Update your `podspec` to include the generated code.
- Update your `build.Gradle` file to include the generated code.
- Update `cmakeListsPath` in `react-native.config.js` so that Gradle doesn't look for CMakeLists file in the build directory but instead in your outputDir.