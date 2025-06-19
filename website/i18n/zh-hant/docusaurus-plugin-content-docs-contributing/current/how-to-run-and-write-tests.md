---
title: How to Run and Write Tests
---

## 執行測試

本節內容針對貢獻者如何測試對 React Native 的修改。若您尚未完成，請先依照[使用原生程式碼建置專案](/docs/environment-setup)的步驟設定開發環境。

### JavaScript 測試

執行 JavaScript 測試套件最簡單的方式，是在 React Native 程式碼庫的根目錄下使用以下指令：

```bash
yarn test
```

此指令會使用 [Jest](https://jestjs.io) 執行測試。

您還應確保程式碼通過 [Flow](https://flowtype.org/) 與 lint 測試：

```bash
yarn flow
yarn lint
```

### iOS 測試

Follow the [README.md](https://github.com/facebook/react-native/blob/main/packages/rn-tester/README.md) instructions in the `packages/rn-tester` directory.

接著返回 React Native 程式碼庫的根目錄並執行 `yarn`，這將設定 JavaScript 相依套件。

此時，您可以透過在 React Native 程式碼庫根目錄執行以下腳本來運行 iOS 測試：

```bash
./scripts/objc-test.sh test
```

您也可以使用 Xcode 執行 iOS 測試。開啟 `RNTester/RNTesterPods.xcworkspace` 後，按 <kbd>Command + U</kbd> 或從選單列選擇 `Product` > `Test` 即可在本機運行測試。

Xcode 還允許透過其測試導覽器執行個別測試。您也可以使用 <kbd>Command + 6</kbd> 快捷鍵。

:::note
`objc-test.sh` 會確保您的測試環境已設定為執行所有測試，同時停用已知不穩定或損壞的測試。使用 Xcode 運行測試時請注意這點。若遇到非預期的失敗，請先檢查該測試是否在 `objc-test.sh` 中被停用。
:::

#### iOS Podfile/Ruby 測試

若您正在修改 `Podfile` 配置，則有 Ruby 測試可驗證這些變更。

執行 Ruby 測試的指令如下：

```bash
cd scripts
sh run_ruby_tests.sh
```

### Android 測試

Android 單元測試不會在模擬器中運行，而是在您本機的 JVM 上執行。

要執行 Android 單元測試，請在 React Native 程式碼庫根目錄執行以下腳本：

```bash
./gradlew test
```

## 撰寫測試

當您修復錯誤或為 React Native 新增功能時，建議添加涵蓋該變更的測試。根據修改內容的不同，適合的測試類型也會有所差異。

### JavaScript 測試

JavaScript 測試位於 `__test__` 目錄中，與被測試檔案處於相同位置。參見 [`TouchableHighlight-test.js`][js-jest-test] 以了解基本範例。您也可以參考 Jest 的[測試 React Native 應用程式][jest-tutorial]教學以獲取更多資訊。

[js-jest-test]: https://github.com/facebook/react-native/blob/main/Libraries/Components/Touchable/__tests__/TouchableHighlight-test.js

[jest-tutorial]: https://jestjs.io/docs/en/tutorial-react-native

### iOS 整合測試

React Native 提供工具來簡化測試需要原生與 JS 元件透過橋接通訊的整合元件。

兩個主要元件是 `RCTTestRunner` 和 `RCTTestModule`。`RCTTestRunner` 設定 React Native 環境並提供在 Xcode 中以 `XCTestCase` 運行測試的功能（`runTest:module` 是最簡單的方法）。`RCTTestModule` 會以 `NativeModules.TestModule` 的形式匯出至 JavaScript。

測試本身是用 JavaScript 編寫的，必須在完成時調用 `TestModule.markTestCompleted()`，否則測試會因超時而失敗。

測試失敗主要通過拋出 JavaScript 異常來表示。也可以使用 `runTest:module:initialProps:expectErrorRegex:` 或 `runTest:module:initialProps:expectErrorBlock:` 來測試錯誤條件，這些方法會預期拋出錯誤並驗證錯誤是否符合提供的條件。

請參考以下範例用法和整合點：

- [`IntegrationTestHarnessTest.js`][f-ios-test-harness]
- [`RNTesterIntegrationTests.m`][f-ios-integration-tests]
- [`IntegrationTestsApp.js`][f-ios-integration-test-app]

[f-ios-test-harness]: https://github.com/facebook/react-native/blob/main/IntegrationTests/IntegrationTestHarnessTest.js

[f-ios-integration-tests]: https://github.com/facebook/react-native/blob/main/RNTester/RNTesterIntegrationTests/RNTesterIntegrationTests.m

[f-ios-integration-test-app]: https://github.com/facebook/react-native/blob/main/IntegrationTests/IntegrationTestsApp.js

### iOS 快照測試

A common type of integration test is the snapshot test. These tests render a component, and verify snapshots of the screen against reference images using `TestModule.verifySnapshot()`, using the [`FBSnapshotTestCase`](https://github.com/facebook/ios-snapshot-test-case) library behind the scenes. Reference images are recorded by setting `recordMode = YES` on the `RCTTestRunner`, then running the tests.

快照在 32 位和 64 位系統以及不同操作系統版本之間會略有不同，因此建議確保測試在[正確配置](https://github.com/facebook/react-native/blob/main/scripts/.tests.env)下運行。

強烈建議模擬所有網絡數據以及其他可能造成問題的依賴項。請參考 [`SimpleSnapshotTest`](https://github.com/facebook/react-native/blob/main/IntegrationTests/SimpleSnapshotTest.js) 查看基本範例。

如果你在拉取請求中進行了影響快照測試的更改（例如在快照範例中添加了一個新的案例），則需要重新記錄快照參考圖片。

To do this, change `recordMode` flag to `_runner.recordMode = YES;` in [RNTester/RNTesterSnapshotTests.m](https://github.com/facebook/react-native/blob/136666e2e7d2bb8d3d51d599fc1384a2f68c43d3/RNTester/RNTesterIntegrationTests/RNTesterSnapshotTests.m#L29), re-run the failing tests, then flip record back to `NO` and submit/update your pull request and wait to see if the CircleCI build passes.

### Android 單元測試

當你處理可以僅通過 Java/Kotlin 代碼測試的代碼時，建議添加一個 Android 單元測試。Android 單元測試位於 `packages/react-native/ReactAndroid/src/test/`。

建議瀏覽這些測試以了解良好的單元測試應該是什麼樣子。

## 持續測試

我們使用 [CircleCI][config-circleci] 自動運行開源測試。每當拉取請求中添加提交時，CircleCI 都會運行這些測試，以幫助維護者了解代碼更改是否引入了回歸。這些測試也會在提交到 `main` 和 `*-stable` 分支時運行，以跟踪這些分支的健康狀況。

[config-circleci]: https://github.com/facebook/react-native/blob/main/.circleci/config.yml

還有另一組測試在 Meta 的內部測試基礎設施中運行。其中一些測試是由 React Native 的內部使用者定義的整合測試（例如 Facebook 應用中 React Native 表面的單元測試）。

這些測試在每次提交到 Facebook 源代碼控制中託管的 React Native 副本時運行。它們也會在拉取請求導入到 Facebook 的源代碼控制時運行。

如果這些測試失敗，您需要 Meta 的內部人員協助檢查。由於只有 Meta 員工能導入 pull request，負責導入的人員應能協助處理相關細節。

:::note
**在本地執行 CI 測試：**
大多數開源協作者依賴 CircleCI 查看測試結果。若您希望使用與 CircleCI 相同的配置在本地驗證變更，可透過 CircleCI 提供的[命令行介面](https://circleci.com/docs/local-cli)在本地執行任務。
:::

### 常見問題

#### 如何升級 CI 測試使用的 Xcode 版本？

升級至新版 Xcode 時，請先確認其[受 CircleCI 支援](https://circleci.com/docs/testing-ios#supported-xcode-versions)。

您還需更新測試環境配置，確保測試在 CircleCI 機器已安裝的 iOS 模擬器上執行。

此資訊亦可查閱 [CircleCI 的 Xcode 版本參考](https://circleci.com/docs/2.0/testing-ios/#supported-xcode-versions)，點選目標版本後查看「Runtimes」段落。

接著編輯以下兩個檔案：

- `.circleci/config.yml`

  Edit the `xcode:` line under `macos:` (search for `_XCODE_VERSION`).

- `scripts/.tests.env`

  Edit the `IOS_TARGET_OS` envvar to match the desired iOS Runtime.

If you intend to merge this change on GitHub, please make sure to notify a Meta employee as they'll need to update the value of `_XCODE_VERSION` used in the internal Sandcastle RN OSS iOS test in `react_native_oss.py` when they import your pull request.