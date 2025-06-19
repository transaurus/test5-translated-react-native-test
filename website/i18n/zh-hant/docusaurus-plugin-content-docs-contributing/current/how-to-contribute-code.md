---
title: How to Contribute Code
---

感謝您對貢獻 React Native 感興趣！從評論和分類問題，到審查和提交 PR，[我們歡迎所有形式的貢獻](/contributing/overview)。本文件將說明如何貢獻程式碼至 React Native。

若您希望立即開始貢獻程式碼，我們有一份 [`good first issues`](https://github.com/facebook/react-native/labels/good%20first%20issue) 清單，其中包含範圍相對有限的錯誤。
標記為 [`help wanted`](https://github.com/facebook/react-native/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3A%22help+wanted+%3Aoctocat%3A%22+sort%3Aupdated-desc+) 的問題也很適合提交 PR。

## 必要條件

:::info
請參閱[環境設定指南](/docs/environment-setup)，根據您使用的平台和開發目標平台來設定必要工具與開發環境。
:::

## 開發工作流程

克隆 React Native 後，開啟目錄並執行 `yarn` 以安裝其相依套件。

現在您已準備好執行以下指令：

- `yarn start` 啟動 Metro 打包伺服器。
- `yarn lint` 檢查程式碼風格。
- `yarn format` 自動格式化您的程式碼。
- `yarn test` 執行基於 Jest 的 JavaScript 測試套件。
  - `yarn test --watch` 執行互動式 JavaScript 測試監視器。
  - `yarn test <pattern>` 執行符合檔名模式的 JavaScript 測試。
- `yarn flow` 執行 [Flow](https://flowtype.org/) 型別檢查。
  - `yarn flow-check-android` 對 `*.android.js` 檔案執行完整 Flow 檢查。
  - `yarn flow-check-ios` 對 `*.ios.js` 檔案執行完整 Flow 檢查。
- `yarn test-typescript` 執行 [TypeScript](https://www.typescriptlang.org/) 型別檢查。
- `yarn test-ios` 執行 iOS 測試套件（需 macOS）。
- `yarn build` 建置所有設定的套件 — 通常此指令僅需由 CI 在發布前執行。
  - 需要建置的套件設定於 [scripts/build/config.js](https://github.com/facebook/react-native/blob/main/scripts/build/config.js)。

## 測試您的變更

測試能協助我們預防程式碼庫引入退化問題。建議您在開發變更時執行 `yarn test` 或上述平台專屬指令碼，以確保未引入任何退化問題。

GitHub 儲存庫透過 CircleCI 進行[持續測試](/contributing/how-to-run-and-write-tests#continuous-testing)，測試結果可透過 [commits](https://github.com/facebook/react-native/commits/main) 和 pull requests 上的 Checks 功能查看。

您可以在[如何執行與撰寫測試](/contributing/how-to-run-and-write-tests)頁面瞭解更多相關資訊。

## 程式碼風格

我們使用 Prettier 來格式化 JavaScript 程式碼。這能節省您的時間與精力，因為您可以透過其編輯器整合自動修復格式問題，或手動執行 `yarn run prettier`。我們也使用 linter 來捕捉程式碼中的風格問題。您可以執行 `yarn run lint` 來檢查程式碼風格狀態。

然而，仍有部分風格問題是 linter 無法捕捉的，特別是 Java 或 Objective-C 程式碼。

### Objective-C

- Space after `@property` declarations
- Brackets on _every_ `if`, on the _same_ line
- `- method`, `@interface`, and `@implementation` brackets on the following line
- _Try_ to keep it around 80 characters line length (sometimes it's not possible...)
- `*` operator goes with the variable name (e.g. `NSObject *variableName;`)

### Java

- 若方法調用跨越多行，閉合括號應與最後一個參數位於同一行。
- 若方法標頭無法單行顯示，則每個參數應獨立成行。
- 行長限制為100個字符

## 提交拉取請求

對React Native的代碼貢獻通常以[拉取請求](https://help.github.com/en/articles/about-pull-requests)的形式進行。向React Native提議更改的流程可概括如下：

1. 複製React Native倉庫並從`main`分支創建你的分支。
2. 若添加了應測試的代碼，請添加測試。
3. 若更改了API，請更新文檔。
4. 確保測試套件通過，無論是在本地還是在你開啟拉取請求後的CI上。
5. 確保代碼風格檢查通過（例如通過`yarn lint --fix`）。
6. 將更改推送至你的複製倉庫。
7. 向React Native倉庫創建拉取請求。
8. 審查並處理拉取請求上的評論。
9. 機器人可能會評論建議。通常我們要求你先解決這些問題，維護者才會審查你的代碼。
10. 若尚未提交，請提交[貢獻者許可協議（"CLA"）](#contributor-license-agreement)。

若一切順利，你的拉取請求將被合併。若未被合併，維護者將盡力解釋原因。

若這是首次提交拉取請求，我們已創建[逐步指南幫助你開始](/contributing/how-to-open-a-pull-request)。有關拉取請求處理的更詳細信息，請參閱[管理拉取請求頁面](managing-pull-requests)。

### 貢獻者許可協議

為了接受你的拉取請求，我們需要你提交[貢獻者許可協議（CLA）](/contributing/contribution-license-agreement)。你只需執行一次此操作，即可參與Meta的任何開源項目。這僅需一分鐘，因此你可以在等待依賴項安裝時完成。

## 許可證

通過貢獻給React Native，你同意你的貢獻將根據React Native倉庫根目錄中的[LICENSE](https://github.com/facebook/react-native/blob/main/LICENSE)文件獲得許可。