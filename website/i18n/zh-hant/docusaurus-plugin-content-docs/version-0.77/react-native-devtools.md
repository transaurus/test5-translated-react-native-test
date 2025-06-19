---
id: react-native-devtools
title: React Native DevTools
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

React Native DevTools 是我們專為 React Native 打造的現代化除錯體驗。從頭開始構建，旨在比以往的除錯方法更整合、更準確且更可靠。

![React Native DevTools 開啟至「歡迎」面板](/docs/assets/debugging-rndt-welcome.jpg)

React Native DevTools 專為除錯 React 應用問題而設計，並非取代原生工具。若需檢查 React Native 底層平台（例如開發原生模組時），請使用 Android Studio 和 Xcode 的除錯工具（參見[除錯原生程式碼](/docs/debugging-native-code)）。

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

React Native DevTools 基於 Chrome DevTools 前端。若有網頁開發背景，其功能應相當熟悉。建議先瀏覽[Chrome DevTools 文件](https://developer.chrome.com/docs/devtools)，內含完整指南與影片資源。

### 主控台

![React Native DevTools 原始碼視圖中的一系列日誌，與裝置並列](/docs/assets/debugging-rndt-console.jpg)

主控台面板可檢視與篩選訊息、執行 JavaScript、檢查物件屬性等。

[主控台功能參考 | Chrome DevTools](https://developer.chrome.com/docs/devtools/console/reference)

#### 實用技巧

- 若應用日誌過多，可使用篩選框或變更顯示的日誌層級。
- 透過[即時運算式](https://developer.chrome.com/docs/devtools/console/live-expressions)觀察值隨時間變化。
- 使用[保留日誌](https://developer.chrome.com/docs/devtools/console/reference#persist)功能在重新載入時保持訊息。
- 使用 <kbd>Ctrl</kbd> + <kbd>L</kbd> 清除主控台視圖。

### 原始碼與中斷點

![React Native DevTools 原始碼視圖中暫停的中斷點，與裝置並列](/docs/assets/debugging-rndt-sources-paused-with-device.jpg)

原始碼面板可檢視應用中的原始檔並設定中斷點。中斷點用於定義程式暫停的程式碼行，讓您檢查程式即時狀態並逐步執行。

[使用中斷點暫停程式碼 | Chrome DevTools](https://developer.chrome.com/docs/devtools/javascript/breakpoints)

:::tip

#### 迷你指南

中斷點是除錯工具中的基本功能！

1. 使用側邊欄或 <kbd>Cmd ⌘</kbd>+<kbd>P</kbd> / <kbd>Ctrl</kbd>+<kbd>P</kbd> 導覽至原始檔。
2. 點擊程式碼行號旁的行號欄新增中斷點。
3. 暫停時使用右上角的導覽控制[逐步執行程式碼](https://developer.chrome.com/docs/devtools/javascript/reference#stepping)。

:::

#### 實用技巧

- 應用暫停時會顯示「除錯器中暫停」疊加層，點擊即可繼續。
- 中斷點時注意右側面板，可檢查當前作用域、呼叫堆疊及設定監控運算式。
- 使用 `debugger;` 陳述式快速從文字編輯器設定中斷點，將透過快速重新整理立即傳至裝置。
- 中斷點有多種類型！例如[條件式中斷點與日誌點](https://developer.chrome.com/docs/devtools/javascript/breakpoints#overview)。

### 記憶體

![在記憶體面板中檢查堆積快照](/docs/assets/debugging-rndt-memory.jpg)

記憶體面板可擷取堆積快照，檢視 JavaScript 程式碼隨時間的記憶體使用情況。

[記錄堆積快照 | Chrome DevTools](https://developer.chrome.com/docs/devtools/memory-problems/heap-snapshots)

#### 實用技巧

- 使用 <kbd>Cmd ⌘</kbd>+<kbd>F</kbd> / <kbd>Ctrl</kbd>+<kbd>F</kbd> 篩選堆積中的特定物件。
- 生成[記憶體分配時間軸報告](https://developer.chrome.com/docs/devtools/memory-problems/allocation-profiler)有助於以圖表形式觀察記憶體使用隨時間變化的情況，從而識別可能的記憶體洩漏問題。

## React DevTools 功能

在整合的「元件」與「效能分析器」面板中，您將找到[React DevTools](https://react.dev/learn/react-developer-tools)瀏覽器擴充功能的所有功能。這些功能在 React Native DevTools 中可無縫運作。

### React 元件

![使用 React 元件面板選取並定位元素](/docs/assets/debugging-rndt-react-components.gif)

「React 元件」面板可讓您檢查並更新渲染的 React 元件樹。

- 在 DevTools 中懸停或選取元素，即可在裝置上高亮顯示該元素。
- 若要定位 DevTools 中的元素，請點擊左上角的「選取元素」按鈕，然後點擊應用中的任意元素。

#### 實用技巧

- 可透過右側面板查看並修改元件運行時的 props 與 state。
- 使用 [React Compiler](https://react.dev/learn/react-compiler) 優化的元件會標註「Memo ✨」徽章。

:::tip

#### 專業技巧：高亮重新渲染

元件重新渲染可能是 React 應用效能問題的主要成因。DevTools 可在重新渲染發生時高亮標示元件。

- 啟用方式：點擊視圖設定圖標（`⚙︎`）並勾選「元件渲染時高亮更新」。

![「高亮更新」設定位置，旁邊是即時渲染疊加層的錄製畫面](/docs/assets/debugging-rndt-highlight-renders.gif)

:::

### React 效能分析器

![以火焰圖形式呈現的效能分析結果](/docs/assets/debugging-rndt-react-profiler.jpg)

「React 效能分析器」面板可讓您錄製效能分析檔案，以了解元件渲染與 React 提交階段的時間分佈。

詳情請參閱[2018年原始指南](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html#reading-performance-data)（請注意部分內容可能已過時）。

## 重新連接 DevTools

有時 DevTools 可能會與目標裝置斷開連接。可能原因包括：

- 應用程式關閉。
- 應用程式重新建置（安裝了新的原生建置版本）。
- 應用程式在原生端崩潰。
- 開發伺服器（Metro）停止運行。
- 實體裝置斷開連接。

斷開連接時，會顯示「偵錯連線已關閉」的對話框。

![裝置斷開連接時顯示的重新連接對話框](/docs/assets/debugging-reconnect-menu.jpg)

此時您可以選擇：

- **關閉**：點擊關閉圖標（`×`）或點擊對話框外部，返回斷開連接前最後狀態的 DevTools 介面。
- **重新連接**：解決斷開原因後，點選「重新連接 DevTools」。