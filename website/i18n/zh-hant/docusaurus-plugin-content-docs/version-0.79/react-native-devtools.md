---
id: react-native-devtools
title: React Native DevTools
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

React Native DevTools 是我們專為 React Native 打造的現代化除錯體驗。從頭開始設計，旨在比以往的除錯方法更深度整合、更準確且更可靠。

![React Native DevTools 開啟至「歡迎」面板](/docs/assets/debugging-rndt-welcome.jpg)

React Native DevTools 專注於除錯 React 應用層問題，並非用來取代原生工具。若需檢查 React Native 底層平台（例如開發原生模組時），請使用 Android Studio 和 Xcode 的除錯工具（參見[原生程式碼除錯](/docs/debugging-native-code)）。

<details>
<summary>**💡 Compatibility** — released in 0.76</summary>

React Native DevTools supports all React Native apps running Hermes. It replaces the previous Flipper, Experimental Debugger, and Hermes debugger (Chrome) frontends.

It is not possible to set up React Native DevTools with any older versions of React Native.

- **Chrome Browser DevTools — unsupported**
  - Connecting to React Native via `chrome://inspect` is no longer supported. Features may not work correctly, as the latest versions of Chrome DevTools (which are built to match the latest browser capabilities and APIs) have not been tested, and this frontend lacks our customisations. Instead, we ship a supported version with React Native DevTools.
- **Visual Studio Code — unsupported** (pre-existing)
  - Third party extensions such as [Expo Tools](https://github.com/expo/vscode-expo) and [Radon IDE](https://ide.swmansion.com/) may have improved compatibility, but are not directly supported by the React team.

</details>

<details>
<summary>**💡 Feedback & FAQs**</summary>

We want the tooling you use to debug React across all platforms to be reliable, familiar, simple, and cohesive. All the features described on this page are built with these principles in mind, and we also want to offer more capabilities in future.

We are actively iterating on the future of React Native DevTools, and have created a centralized [GitHub discussion](https://github.com/react-native-community/discussions-and-proposals/discussions/819) to keep track of issues, frequently asked questions, and feedback.

</details>

## 核心功能

React Native DevTools 基於 Chrome DevTools 前端架構。若您有網頁開發背景，其功能應相當熟悉。建議先瀏覽[Chrome DevTools 文件](https://developer.chrome.com/docs/devtools)，內含完整指南與影片資源。

### 主控台

![React Native DevTools 原始碼檢視中的日誌序列與裝置並列](/docs/assets/debugging-rndt-console.jpg)

主控台面板可檢視/過濾訊息、執行 JavaScript、檢查物件屬性等。

[主控台功能參考 | Chrome DevTools](https://developer.chrome.com/docs/devtools/console/reference)

#### 實用技巧

- 若應用日誌過多，請使用過濾框或調整顯示的日誌層級
- 透過[即時運算式](https://developer.chrome.com/docs/devtools/console/live-expressions)監控數值變化
- 使用[保留日誌](https://developer.chrome.com/docs/devtools/console/reference#persist)功能在重新載入時保持訊息
- 快捷鍵 <kbd>Ctrl</kbd> + <kbd>L</kbd> 可清除主控台視圖

### 原始碼與中斷點

![React Native DevTools 原始碼檢視中暫停的中斷點與裝置並列](/docs/assets/debugging-rndt-sources-paused-with-device.jpg)

原始碼面板可檢視應用原始檔並設置中斷點。中斷點能指定程式暫停執行的程式碼行，讓您檢查即時狀態並逐步執行代碼。

[使用中斷點暫停程式碼 | Chrome DevTools](https://developer.chrome.com/docs/devtools/javascript/breakpoints)

:::tip

#### Mini-guide

Breakpoints are a fundamental tool in your debugging toolkit!

1. Navigate to a source file using the sidebar or <kbd>Cmd ⌘</kbd>+<kbd>P</kbd> / <kbd>Ctrl</kbd>+<kbd>P</kbd>.
2. Click in the line number column next to a line of code to add a breakpoint.
3. Use the navigation controls at the top right to [step through code](https://developer.chrome.com/docs/devtools/javascript/reference#stepping) when paused.

:::

#### 實用技巧

- A "Paused in Debugger" overlay will appear when your app is paused. Tap it to resume.
- Pay attention to the right hand side panels when on a breakpoint, which allow you to inspect the current scope and call stack, and set watch expressions.
- Use a `debugger;` statement to quickly set a breakpoint from your text editor. This will reach the device immediately via Fast Refresh.
- There are multiple kinds of breakpoints! For example, [Conditional Breakpoints and Logpoints](https://developer.chrome.com/docs/devtools/javascript/breakpoints#overview).

### 記憶體

![在記憶體面板檢查堆積快照](/docs/assets/debugging-rndt-memory.jpg)

記憶體面板可擷取堆積快照，監控 JavaScript 程式碼的記憶體使用狀況。

[記錄堆積快照 | Chrome DevTools](https://developer.chrome.com/docs/devtools/memory-problems/heap-snapshots)

#### 實用技巧

- 使用 <kbd>Cmd ⌘</kbd>+<kbd>F</kbd> / <kbd>Ctrl</kbd>+<kbd>F</kbd> 篩選堆積中的特定物件。
- 建立[記憶體分配時間軸報告](https://developer.chrome.com/docs/devtools/memory-problems/allocation-profiler)有助於以圖表形式觀察記憶體使用情況隨時間的變化，從而識別可能的記憶體洩漏。

## React DevTools 功能

在整合的「元件」和「分析器」面板中，您會找到[React DevTools](https://react.dev/learn/react-developer-tools)瀏覽器擴充功能的所有功能。這些功能在 React Native DevTools 中無縫運作。

### React 元件

![使用 React 元件面板選取並定位元素](/docs/assets/debugging-rndt-react-components.gif)

React 元件面板讓您可以檢查並更新渲染的 React 元件樹。

- 在 DevTools 中懸停或選取元素，即可在裝置上高亮顯示該元素。
- 若要在 DevTools 中定位元素，請點擊左上角的「選取元素」按鈕，然後點擊應用中的任何元素。

#### 實用技巧

- 元件的 props 和 state 可以在執行時透過右側面板查看和修改。
- 使用 [React Compiler](https://react.dev/learn/react-compiler) 優化的元件會標註「Memo ✨」徽章。

:::tip

#### 專業技巧：高亮顯示重新渲染

重新渲染可能是 React 應用效能問題的重要來源。DevTools 可以在元件重新渲染時高亮顯示。

- 啟用方式：點擊「檢視設定」圖標（`⚙︎`）並勾選「元件渲染時高亮顯示更新」。

![「高亮顯示更新」設定的位置，旁邊是即時渲染疊加層的錄製畫面](/docs/assets/debugging-rndt-highlight-renders.gif)

:::

### React 分析器

![以火焰圖形式呈現的分析結果](/docs/assets/debugging-rndt-react-profiler.jpg)

React 分析器面板讓您可以錄製效能分析資料，以了解元件渲染和 React 提交的時間分佈。

更多資訊，請參閱[2018 年原始指南](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html#reading-performance-data)（請注意部分內容可能已過時）。

## 重新連接 DevTools

有時 DevTools 可能會與目標裝置斷開連接。這可能發生在以下情況：

- 應用程式關閉。
- 應用程式重新建置（安裝了新的原生建置版本）。
- 應用程式在原生端崩潰。
- 開發伺服器（Metro）停止運作。
- 實體裝置斷開連接。

斷開連接時，會顯示一個對話框，訊息為「偵錯連線已關閉」。

![裝置斷開連接時顯示的重新連接對話框](/docs/assets/debugging-reconnect-menu.jpg)

此時，您可以選擇：

- **忽略**：點擊關閉（`×`）圖標或對話框外部，返回斷開連接前的 DevTools 介面狀態。
- **重新連接**：選擇「重新連接 DevTools」，前提是已解決斷開連接的原因。