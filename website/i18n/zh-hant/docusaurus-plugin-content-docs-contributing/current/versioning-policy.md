---
title: Versioning Policy
---

本頁面描述了我們為 `react-native` 套件所遵循的版本控制政策。

我們對每個 React Native 版本都進行了徹底測試，包括手動和自動化測試，以確保品質不會倒退。

React Native 的 `stable` 頻道遵循以下描述的 0.x.y 發布政策。

React Native 還提供 `nightly` 發布頻道，以鼓勵對實驗性功能的早期反饋。

本頁面描述了我們對 `react-native` 及 `@react-native` 範圍內套件的版本號碼處理方式。

## 穩定發布版本

React Native 以固定節奏發布穩定版本。

我們遵循 0.x.y 版本控制架構：

- 重大變更將在新的次要版本中發布，即增加 x 數字（例如：0.78.0 至 0.79.0）。
- 新功能和 API 也將在新的次要版本中發布，即增加 x 數字（例如：0.78.0 至 0.79.0）。
- 關鍵錯誤修復將在新的修補版本中發布，即增加 y 數字（例如：0.78.1 至 0.78.2）。

穩定版本會定期發布，最新版本在 NPM 上標記為 `latest`。

同一次要版本號下的一系列發布稱為**次要系列**（例如 0.76.x 是 0.76.0、0.76.1、0.76.2 等的次要系列）。

## 穩定性承諾

為了支援使用者升級 React Native 版本，我們承諾維護最新的 3 個次要系列（例如當 0.78 是最新發布時，維護 0.78.x、0.77.x 和 0.76.x）。

我們將為這些版本定期發布更新和錯誤修復。

您可以在 [react-native-releases 工作組](https://github.com/reactwg/react-native-releases/blob/main/docs/support.md) 閱讀更多關於我們的支援政策。

### 重大變更

重大變更對所有人都不便，我們正努力將其減少到最低限度。每個穩定版本中的所有重大變更都將在以下位置突出顯示：

- The _Breaking_ and the _Removed_ section of [the React Native Changelog](https://github.com/facebook/react-native/blob/main/CHANGELOG.md)
- Each release blogpost in the _Breaking Changes_ section

對於每個重大變更，我們承諾解釋其背後的原因，提供替代 API（如果可能），並最小化對最終使用者的影響。

### 什麼是重大變更？

我們認為 React Native 的重大變更包括：

- An incompatible API change (i.e. an API that is changed or removed so that your code won’t compile/run anymore due to that change). Examples:
  - Changes of any JS/Java/Kotlin/Obj-c/C++ APIs that would require your code to be changed in order to compile.
  - Changes inside `@react-native/codegen` that are not backward compatible.
- A significant behavior/runtime change. Example:
  - The layout logic of a prop is changed drastically.
- A significant change in the development experience. Example:
  - A debugging feature is entirely removed.
- A major bump of any of our transitive dependencies. Examples:
  - Bumping React from 18.x to 19.x
  - Bumping the Target SDK on Android from 34 to 35).
- A reduction of any of our supported platform versions. Examples:
  - Bumping min SDK on Android from 21 to 23
  - Bumping the min iOS version to 15.1.

我們不認為以下變更屬於重大變更：

- 修改以 `unstable_` 為前綴的 API：這些 API 提供實驗性功能，我們對其最終形態尚無十足把握。透過以 `unstable_` 前綴發布這些 API，我們能更快迭代並早日達成穩定的 API 設計。
- 變更私有或內部 API：這些 API 通常以 `internal_`、`private_` 為前綴，或位於 `internal/`、`private/` 目錄/套件中。儘管部分 API 可能因工具限制而具有公開可見性，我們不將其視為公共 API 的一部分，因此會直接變更而不另行通知。
  - 同理，若您存取內部屬性名稱如 `__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED` 或 `__reactInternalInstance$uk43rzhitjg`，我們無法提供任何保證。此類使用需自行承擔風險。
  - 標註為 `@FrameworkAPI` 的類別亦屬內部 API
- 變更工具/開發用 API：React Native 部分公開 API 專供框架與其他工具整合使用。例如，某些 Metro API 或 React Native DevTools API 僅限其他框架或工具使用。此類 API 的變更會直接與受影響工具方討論，不視為破壞性變更（我們不會在版本發布部落格中廣泛公告）。
- 開發警告：由於警告不影響運行時行為，我們可能在任何版本間新增或修改現有警告。

若預期某項變更會對社群造成廣泛影響，我們仍會盡力為生態系統提供漸進式遷移路徑。

### 廢棄週期

隨著 React Native 持續開發演進，我們會編寫新 API 並有時需廢棄現有 API。這些 API 將經歷廢棄週期。

API 一旦被標記為廢棄後，仍會**持續提供**於**後續**穩定版本中。

例如：若某 API 在 React Native 0.76.x 被廢棄，它仍會存在於 0.77.x 版本，且不會早於 React Native 0.78.x 被移除。

有時我們會因生態系統需要更長遷移時間，而決定延長廢棄 API 的保留期限。對此類 API，我們通常會提供警告訊息協助使用者遷移。

## 發布頻道

React Native 依賴活躍的開源社群提交錯誤報告、發起拉取請求與提案討論。為鼓勵回饋，我們支援多種發布頻道。

:::note
本節內容主要與開發框架、函式庫或工具鏈的開發者相關。以 React Native 開發終端使用者應用的開發者通常只需關注 latest 頻道。
:::

### latest

`latest` 頻道提供符合語意化版本規範的穩定版 React Native 發布。此為透過 npm 安裝 React Native 時的預設頻道，也是您當前使用的版本。直接整合 React Native 的終端應用應使用此頻道。

我們會定期發布新版次要系列，並更新 `latest` 標籤指向最新穩定版本。

### next

在宣告新 React Native 版本穩定前，我們會發布一系列**候選版本**（從 RC0 開始）。這些預發布版本遵循 `0.79.0-rc.0` 格式版本號，並標記為 NPM 上的 `next` 標籤。

當新分支切割完成且 RC 版本開始發布至 NPM 與 GitHub 時，建議測試您的函式庫/框架是否相容 React Native 的 `next` 版本。

此舉能確保您的專案在未來 React Native 版本中仍可正常運作。

但請勿直接於終端應用中使用預發布/RC 版本，因其未達生產環境標準。

### nightly

We also publish a `nightly` release channel. Nightlies are published every day starting from the `main` branch of [facebook/react-native](https://github.com/facebook/react-native). Nightlies are considered unstable versions of React Native and are not recommended for production use.

夜間版本遵循 `0.80.0-nightly-<DATE>-<SHA>` 的版本命名規則，其中 `<DATE>` 為發佈日期，`<SHA>` 為該版本對應的提交 SHA 值。

夜間版本僅供測試用途，我們無法保證不同夜間版本之間的行為一致性。這些版本不遵循我們在 latest/next 頻道使用的語意化版本規範(semver)。

建議設置持續整合(CI)流程，每日針對 react-native@nightly 版本測試您的函式庫，以確保其能相容於未來的正式版本。