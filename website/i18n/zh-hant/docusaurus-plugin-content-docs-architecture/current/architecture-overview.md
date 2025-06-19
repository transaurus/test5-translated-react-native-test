---
id: architecture-overview
title: Architecture Overview
slug: /overview
---

:::info
Welcome to the Architecture Overview! If you're getting started with React Native, please refer to <a href="/docs/getting-started">Guides</a> section. Continue reading to learn how internals of React Native work!

This section is a work in progress and more material will be added in the future. Please make sure to come back later to check for new information.
:::

Architecture Overview is intended to share conceptual overview of how React Native's internals work. The intended audience includes library authors and core contributors. If you are an app developer, it is not a requirement to be familiar with this material to be effective with React Native. You can still benefit from the overview as it will give you insights into how React Native works under the hood. Feel free to share your feedback on the <a href="https://github.com/reactwg/react-native-new-architecture/discussions/9">discussion inside the working group</a> for this section.

## 目錄

- [關於新架構](landing-page)
- 渲染系統
  - [Fabric](fabric-renderer)
  - [渲染、提交與掛載](render-pipeline)
  - [跨平台實作](xplat-implementation)
  - [視圖扁平化](view-flattening)
  - [執行緒模型](threading-model)
- 建置工具
  - [捆綁式 Hermes](bundled-hermes)
- [術語表](glossary)