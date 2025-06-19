---
id: xplat-implementation
title: Cross Platform Implementation
---

import FabricWarning from './\_fabric-warning.mdx';

<FabricWarning />

#### React Native 渲染器採用核心渲染實現以跨平台共享

在 React Native 先前的渲染系統中，**[React 陰影樹](architecture-glossary.md#react-shadow-tree-and-react-shadow-node)**、佈局邏輯及 **[視圖扁平化](view-flattening.md)** 演算法需針對每個平台各自實作一次。現行渲染器設計為跨平台解決方案，透過共享核心 C++ 實現達成。

React Native 團隊計劃將動畫系統整合至渲染系統，並將渲染系統擴展至新平台，如 Windows 及遊戲主機、電視等作業系統。

採用 C++ 作為核心渲染系統帶來多項優勢。單一實現降低了開發與維護成本，並提升建立 React 陰影樹與佈局計算的效能，因為在 Android 上整合 [Yoga](architecture-glossary.md#yoga-tree-and-yoga-node) 與渲染器的開銷被最小化（例如不再需要為 Yoga 使用 [JNI](architecture-glossary.md#java-native-interface-jni)）。此外，以 C++ 分配的每個 React 陰影節點記憶體佔用，比從 Kotlin 或 Swift 分配更小。

團隊亦運用 C++ 強制不可變性的特性，確保不會發生並行存取共享但未受保護資源的問題。

需特別注意的是，Android 的渲染器使用案例仍須負擔 [JNI](architecture-glossary.md#java-native-interface-jni) 的兩項主要成本：

- 複雜視圖（如 `Text`、`TextInput` 等）的佈局計算需透過 JNI 傳送屬性。
- 掛載階段需透過 JNI 傳送變異操作。

The team is exploring replacing `ReadableMap` with a new mechanism to serialize data using `ByteBuffer` to reduce overhead of JNI. Our goal is to reduce overhead of JNI by 35–50%.

渲染器提供其 C++ API 的兩個面向：

- **(一)** 與 React 通訊
- **(二)** 與宿主平台通訊

對於 **(一)**，React 與渲染器通訊以**渲染** React 樹，並「監聽」**事件**（如 `onLayout`、`onKeyPress`、觸控等）。

對於 **(二)**，React Native 渲染器與宿主平台通訊以在螢幕上掛載宿主視圖（建立、插入、更新或刪除宿主視圖），並監聽使用者在宿主平台產生的**事件**。

![跨平台實現示意圖](/docs/assets/Architecture/xplat-implementation/xplat-implementation-diagram.png)