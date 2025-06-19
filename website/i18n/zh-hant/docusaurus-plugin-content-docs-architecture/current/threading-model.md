---
id: threading-model
title: Threading Model
---

import FabricWarning from './\_fabric-warning.mdx';

<FabricWarning />

#### React Native 渲染器將[渲染管線](render-pipeline)的工作分配至多個執行緒

此處我們定義執行緒模型，並提供範例說明渲染管線的執行緒使用方式。

React Native 渲染器設計為執行緒安全。高層級上，執行緒安全是通過在框架內部使用不可變數據結構來保證的（由 C++ 的「const correctness」特性強制執行）。這意味著 React 中的每次更新都會在渲染器中創建或克隆新對象，而非更新現有數據結構。此設計允許框架向 React 提供執行緒安全且同步的 API。

渲染器使用兩個不同的執行緒：

- **UI 執行緒**（常稱為主執行緒）：唯一能操作宿主視圖的執行緒。
- **JavaScript 執行緒**：React 的渲染階段及佈局計算在此執行。

以下檢視各階段支援的執行情境：

<figure>
  <img src="/docs/assets/Architecture/threading-model/symbols.png" alt="Threading model symbols" />
</figure>

## 渲染情境

### 在 JS 執行緒中渲染

這是最常見的情境，大部分渲染管線流程發生在 JavaScript 執行緒上。

<figure>
	<img src="/docs/assets/Architecture/threading-model/case-1.jpg" alt="Threading model use case one" />
</figure>

---

### 在 UI 執行緒中渲染

當 UI 執行緒有高優先級事件時，渲染器能同步在 UI 執行緒上執行整個渲染管線。

<figure>
	<img src="/docs/assets/Architecture/threading-model/case-2.jpg" alt="Threading model use case two" />
</figure>

---

### 預設或連續事件中斷

此情境展示 UI 執行緒低優先級事件中斷渲染階段。React 與 React Native 渲染器能中斷渲染階段，並將其狀態與 UI 執行緒上執行的低優先級事件合併。此時渲染流程會繼續在 JS 執行緒執行。

<figure>
	<img src="/docs/assets/Architecture/threading-model/case-3.jpg" alt="Threading model use case three" />
</figure>

---

### 離散事件中斷

渲染階段可被中斷。此情境展示 UI 執行緒高優先級事件中斷渲染階段。React 與渲染器能中斷渲染階段，並將其狀態與 UI 執行緒上執行的高優先級事件合併。渲染階段會同步在 UI 執行緒執行。

<figure>
	<img src="/docs/assets/Architecture/threading-model/case-4.jpg" alt="Threading model use case four" />
</figure>

---

### C++ 狀態更新

源自 UI 執行緒的更新會跳過渲染階段。詳見 [React Native 渲染器狀態更新](render-pipeline#react-native-renderer-state-updates)。

<figure>
	<img src="/docs/assets/Architecture/threading-model/case-6.jpg" alt="Threading model use case six" />
</figure>