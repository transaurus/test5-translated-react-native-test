---
id: architecture-glossary
title: Glossary
slug: /glossary
---

## 開發者選單

應用內開發者選單（僅在開發版本中可用），提供各種開發與除錯功能的存取。[在文件中了解更多關於開發者選單的資訊](/docs/debugging)。

## Fabric 渲染器

React Native 執行與網頁版 React 相同的 React 框架程式碼。然而，React Native 渲染至通用平台視圖（宿主視圖），而非 DOM 節點（可視為網頁的宿主視圖）。透過 Fabric 渲染器實現了渲染至宿主視圖的功能。Fabric 讓 React 能與各平台溝通並管理其宿主視圖實例。Fabric 渲染器存在於 JavaScript 中，並以 C++ 程式碼提供的介面為目標。[在此部落格文章中了解更多關於 React 渲染器的資訊。](https://overreacted.io/react-as-a-ui-runtime/#renderers)

## 宿主平台

嵌入 React Native 的平台（例如 Android、iOS、macOS、Windows）。

## 宿主視圖樹（與宿主視圖）

宿主平台中視圖的樹狀表示（例如 Android、iOS）。在 Android 上，宿主視圖是 `android.view.ViewGroup`、`android.widget.TextView` 等的實例，這些是宿主視圖樹的基礎構件。每個宿主視圖的大小與位置基於由 Yoga 計算的 `LayoutMetrics`，而每個宿主視圖的樣式與內容則基於來自 React Shadow Tree 的資訊。

## JavaScript 介面（JSI）

一種輕量級 API，用於將 JavaScript 引擎嵌入 C++ 應用程式。Fabric 使用它在 Fabric 的 C++ 核心與 React 之間進行通訊。

## Java 原生介面（JNI）

一種[用於編寫 Java 原生方法的 API](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/)，用於在 Fabric 的 C++ 核心與以 Java 編寫的 Android 之間進行通訊。

## React 元件

一個 JavaScript 函數或類別，指示如何建立 React 元素。[在此部落格文章中了解更多關於 React 元件與元素的資訊。](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)

## React 複合元件

具有 `render` 實作的 React 元件，可簡化為其他 React 複合元件或 React 宿主元件。

## React 宿主元件或宿主元件

React Components whose view implementation is provided by a host view (e.g., `<View>, <Text>` ). On the Web, ReactDOM's Host components would be components like `<p>` and `<div>`.

## React 元素樹（與 React 元素）

A _React Element Tree_ is created by React in JavaScript and consists of React Elements. A _React Element_ is a plain JavaScript object that describes what should appear on the screen. It includes props, styles, and children. React Elements only exist in JavaScript and can represent instantiations of either React Composite Components or React Host Components. [Read more about React components and elements in this blog post.](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)

## React Native 框架

React Native 讓開發者能使用 [React 程式設計範式](https://react.dev/learn/thinking-in-react)將應用程式部署至原生目標。React Native 團隊專注於建立**核心 API** 與**功能**，以滿足開發原生應用程式時最通用的使用案例。

將原生應用程式部署至生產環境通常需要一組工具與函式庫，這些並非 React Native 預設提供，但對於部署應用程式至生產環境至關重要。此類工具的範例包括：支援將應用程式發布至專用商店，或支援路由與導航機制。

當這些工具與函式庫被整合形成一個基於 React Native 的連貫框架時，我們將其定義為**React Native 框架**。

開源 React Native 框架的一個範例是 [Expo](https://expo.dev/)。

## React 陰影樹（與 React 陰影節點）

A _React Shadow Tree_ is created by the Fabric Renderer and consists of React Shadow Nodes. A React Shadow Node is an object that represents a React Host Component to be mounted, and contains props that originate from JavaScript. They also contain layout information (x, y, width, height). In Fabric, React Shadow Node objects exist in C++. Before Fabric, these existed in the mobile runtime heap (e.g. Android JVM).

## Yoga 樹（與 Yoga 節點）

The _Yoga Tree_ is used by [Yoga](https://www.yogalayout.dev/) to calculate layout information for a React Shadow Tree. Each React Shadow Node typically creates a _Yoga Node_ because React Native employs Yoga to calculate layout. However, this is not a hard requirement. Fabric can also create React Shadow Nodes that do not use Yoga; the implementation of each React Shadow Node determines how to calculate layout.