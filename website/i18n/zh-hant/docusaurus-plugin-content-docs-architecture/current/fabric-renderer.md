---
id: fabric-renderer
title: Fabric
---

Fabric 是 React Native 的新渲染系統，是傳統渲染系統的概念演進。其核心原則在於將更多渲染邏輯統一至 C++、提升與[宿主平台](architecture-glossary.md#host-platform)的互通性，並為 React Native 解鎖新功能。開發始於 2018 年，2021 年起 Facebook 應用程式中的 React Native 已由新渲染器驅動。

本文件提供[新渲染器](architecture-glossary.md#fabric-render)及其概念的概覽。內容避免平台特定細節，不含程式碼片段或指引。涵蓋關鍵概念、動機、優勢，以及不同情境下的渲染管線綜覽。

## 新渲染器的動機與優勢

此渲染架構旨在實現傳統架構無法達成的更佳使用者體驗，例如：

- With improved interoperability between [host views](architecture-glossary.md#host-view-tree-and-host-view) and React views, the renderer is able to measure and render React surfaces synchronously. In the legacy architecture, React Native layout was asynchronous which led to a layout “jump” issue when embedding a React Native rendered view in a _host view_.
- With support of multi-priority and synchronous events, the renderer can prioritize certain user interactions to ensure they are handled in a timely manner.
- [Integration with React Suspense](https://reactjs.org/blog/2019/11/06/building-great-user-experiences-with-concurrent-mode-and-suspense.html) which allows for more intuitive design of data fetching in React apps.
- Enable React [Concurrent Features](https://github.com/reactwg/react-18/discussions/4) on React Native.
- Easier to implement server side rendering for React Native.

新架構在程式碼品質、效能與擴展性上也帶來優勢：

- **Type safety:** code generation to ensure type safety across the JS and [host platforms](architecture-glossary.md#host-platform). The code generation uses JavaScript component declarations as source of truth to generate C++ structs to hold the props. Mismatch between JavaScript and host component props triggers a build error.
- **Shared C++ core**: the renderer is implemented in C++ and the core is shared among platforms. This increases consistency and makes it easier to adopt React Native on new platforms.
- **Better Host Platform Interoperability**: Synchronous and thread-safe layout calculation improves user experiences when embedding host components into React Native, which means easier integration with host platform frameworks that require synchronous APIs.
- **Improved Performance**: With the new cross-platform implementation of the renderer system, every platform benefits from performance improvements that may have been motivated by limitations of one platform. For example, view flattening was originally a performance solution for Android and is now provided by default on both Android and iOS.
- **Consistency**: The new render system is cross-platform, it is easier to keep consistency among different platforms.
- **Faster Startup**: Host components are lazily initialized by default.
- **Less serialization of data between JS and host platform**: React used to transfer data between JavaScript and _host platform_ as serialized JSON. The new renderer improves the transfer of data by accessing JavaScript values directly using [JavaScript Interfaces (JSI)](architecture-glossary.md#javascript-interfaces-jsi).