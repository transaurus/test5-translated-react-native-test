---
title: React Native Core Contributor Summit 2022
authors: [thymikee, cortinico]
tags: [announcement]
date: 2022-11-22
---

# React Native 核心貢獻者高峰會 2022

歷經多年疫情與僅限線上的活動後，我們深感是時候讓 React Native 的核心貢獻者們齊聚一堂！

因此，我們在九月初召集了 React Native 活躍的核心貢獻者、函式庫維護者，以及 Meta 的 React Native 與 Metro 團隊成員，舉辦了**核心貢獻者高峰會 2022**。[Callstack](https://www.callstack.com/) 在其波蘭弗羅茨瓦夫總部主辦了這場高峰會，同期也舉行了 [React Native EU](https://www.react-native.eu/) 研討會。

我們與 React Native 核心團隊共同規劃了一系列**工作坊**，供與會者參與。主題包括：

- React Native Codegen 與 TypeScript 支援
- React Native 新架構函式庫遷移
- React Native Monorepo
- Metro Web 與生態系統對齊
- Metro 簡化發布流程

這兩天密集的知識交流與協作成果令我們驚艷。本篇部落格文章將帶您一窺這場聚會的成果。

<!--truncate-->

### React Native Codegen 與 TypeScript 支援

[React Native 的 Codegen](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/codegen.md) 是新架構的關鍵元件。強化其功能是我們對 React Native 未來的首要任務之一。例如今年稍早，我們新增了從 TypeScript 規格（而非 Flow）生成泛型程式碼的支援。

本場次中，我們藉機向新貢獻者介紹 Codegen，解釋其核心概念與運作原理，並聚焦兩大重點：

#### 1. 支援 Codegen **尚未處理的新型別**。其中高度需求的功能是 [TypeScript 的字串聯集型別](https://github.com/Titozzz/react-native/tree/codegen-string-union)。

數人組成的小組進入會議室實作此功能。過程中遭遇並克服了諸多挑戰，例如如何執行 Codegen 的單元測試。他們花費相當時間理解程式執行流程後才開始編碼。經過數小時協作，最終產出能識別字串聯集的初版原型。此經驗對討論未來理想的設計模式與架構極具價值。

#### 2. 改進 **[iOS 的自動連結功能](https://github.com/facebook/react-native/pull/34580)**，補足原有缺失的使用情境。

具體而言，當函式庫與應用程式共存於 monorepo 時，自動連結無法妥善運作。Android 已支援此情境，但 iOS 尚缺對應方案。

與貢獻者協作 Codegen 的過程讓我們體認到，其程式碼基底現狀不利開發。例如新增型別支援時，需在四個地方重複相同程式碼：使用 Flow 規格的模組、使用 TypeScript 規格的模組、使用 Flow 規格的元件，以及使用 TypeScript 規格的元件。

此發現促使我們建立[統整任務](https://github.com/facebook/react-native/issues/34872)，向社群尋求協助以提升程式碼基底的可維護性。

社群響應熱烈：我們在 **5 天內迅速分配了前 40 項任務**。截至十月底，社群已完成 **47 項任務**，另有許多修改待合併。

此計畫也讓所有參與改進的貢獻者為 [Hacktoberfest](https://hacktoberfest.com/) 添磚加瓦！

### React Native 新架構函式庫遷移

React Native 領域的熱門話題是新架構。讓**函式庫**支援新架構是[整個生態系統遷移](/blog/2022/06/16/resources-migrating-your-react-native-library-to-the-new-architecture)的關鍵點。因此，我們希望協助函式庫維護者遷移至新架構。

這場會議最初以腦力激盪的形式展開，核心貢獻者有機會向 React Native 團隊提出所有關於新架構的疑問。這種面對面的回饋循環對雙方都至關重要：既能讓核心貢獻者釐清問題，也能讓 React Native 團隊收集意見。部分回饋與顧慮將在 React Native 0.71 版本中實現。

接著我們實際著手將多個函式庫遷移至新架構。在此過程中，我們啟動了數個社群套件的遷移工作，例如 `react-native-document-picker`、`react-native-store-review` 和 `react-native-orientation`。

提醒您，若您也在遷移函式庫並需要協助，請透過 GitHub 聯繫我們的[新架構工作小組](https://github.com/reactwg/react-native-new-architecture)。

### React Native Monorepo

目前發布新版 React Native 並非易事。React Native 是 NPM 上下載量最高的套件之一，我們必須確保發布流程順暢。

因此我們計劃重構 `react-native` 代碼庫並實作 **Monorepo RFC**（[#480](https://github.com/react-native-community/discussions-and-proposals/pull/480)）。

會議中，我們先收集每位貢獻者的意見進行腦力激盪，這至關重要——我們需要演進代碼庫結構，同時減少對下游依賴項的破壞性變更。

隨後我們從兩個方向展開工作：首先擴充持續整合基礎設施以支援 monorepo，在測試環境中加入 [Verdaccio](https://verdaccio.org/)；接著開始為多個套件重新命名並添加作用域，最終產出 6 項獨立貢獻。

您可透過此[統整議題](https://github.com/facebook/react-native/issues/34692)追蹤進度，我們期待在不久的將來分享更多成果。

### Metro Web 與生態系統對齊

[Metro](https://github.com/facebook/metro), our JavaScript Bundler, is a foundational and integrated part of the React Native development experience and we want to make sure it works with the latest standards in the JS ecosystem.

本場會議聚焦於討論如何強化 Metro 功能集，使其更適合網頁使用情境並與 npm 及打包工具生態系相容。主要探討兩大方向：

#### 1. 採用 `"exports"`（[套件入口點](https://nodejs.org/api/packages.html#package-entry-points)）規範

根據 [Node.js 文件](https://nodejs.org/api/packages.html#package-entry-points)：

<!-- alex ignore clearly -->

:::info
The ["exports"](https://nodejs.org/api/packages.html#exports) provides a modern alternative to ["main"](https://nodejs.org/api/packages.html#main) allowing multiple entry points to be defined, conditional entry resolution support between environments, and **preventing any other entry points besides those defined in ["exports"](https://nodejs.org/api/packages.html#exports)**. This encapsulation allows module authors to define the public interface for their package clearly.
:::

Adopting the `"exports"` specification has a lot of potential. In this session, we debated on how to handle [Platform Specific Code](/docs/platform-specific-code#platform-specific-extensions) with `"exports"`. Considering many factors, we came up with a fairly non-breaking rollout plan for `"exports"`, by adding a `"strict"` and `"non-strict"` mode to Metro resolver. We discussed how leveraging [builder-bob](https://github.com/callstack/react-native-builder-bob) would help library creators adopt the strict mode without friction.

此討論產出以下成果：

1. 一份關於 Metro 如何與 React Native 協同處理套件導出的 [RFC](https://github.com/react-native-community/discussions-and-proposals/pull/534)。
2. 一份提議 Node.js 將 "react-native" 納入社群條件的 [RFC](https://github.com/nodejs/node/pull/45367)。

#### 2. 網頁與打包工具生態系

Metro 團隊分享了與 Expo 合作後的進展，並計劃延續此合作模式來實現未來的程式碼分割（bundle splitting）與樹搖（tree-shaking）支援。我們再次討論了 ES 模組的支援，並探討了未來可能的功能，如 Yarn PnP 和網頁端的輸出優化。我們也討論了 Metro 核心如何與 Jest 共享邏輯和資料結構，以及進一步重複利用的機會。

開發者們提出了關於程式碼分割和與第三方工具互操作性的深刻用例。這促使我們討論 Metro 中潛在的擴展點以及改進現有文件的可能性。

這些討論為我們隔天關於簡化發布流程的會議奠定了良好的基礎。

### Metro 簡化發布流程

如前所述，發布 React Native 並非易事。

當我們需要同時發布 React Native、React Native CLI 和 Metro 時，情況變得更加複雜。這些工具相互關聯，因為 React Native 和 CLI 都依賴於 Metro。這在任何一個套件發布新版本時都會造成一些摩擦。

目前，我們通過直接溝通和同步發布來管理這一過程，但仍有改進的空間。

在這次會議中，我們重新審視了 React Native、Metro 和 CLI 之間的**依賴關係**。我們發現，在 [「Lean Core」計劃](https://github.com/react-native-community/discussions-and-proposals/issues/6)期間的一些設計決策，當我們將 CLI 從 React Native 中分離出來時，使得這兩個專案相互依賴，並導致部分功能在兩者之間重複實現。當時的決策有其合理性，並讓 CLI 團隊能夠以更快的速度迭代。

現在是時候重新審視這些決策，並結合兩個團隊的經驗來找到解決方案。最終，Metro 團隊將接管 [`@react-native-community/cli-plugin-metro`](https://github.com/react-native-community/cli/tree/main/packages/cli-plugin-metro) 的開發工作，暫時將其移回 React Native 核心，之後很可能會納入 Metro 的單一儲存庫（monorepo）。

![](/blog/assets/core-contributor-summit-2022.jpg)

除了在白板上花費三小時繪製套件間的依賴關係圖外，最大的收穫是 CLI 和 Metro 團隊能夠交流彼此的問題、經驗和計劃，從而增進了相互理解。

如果沒有實際的面對面交流，我們無法達到這種程度的合作。

---

我們仍然驚嘆於短短幾天的密集會議所帶來的知識共享和想法交流。在這次峰會中，我們為改善和重塑 React Native 生態系的計劃播下了種子。

我們要再次感謝 [Callstack](https://www.callstack.com/) 的熱情接待，以及所有參與 2022 年核心貢獻者峰會的與會者。

如果您有興趣參與 React Native 的開發，請務必加入我們的開放計劃並閱讀網站上的[貢獻指南](https://reactnative.dev/contributing/overview)。我們期待未來也能與您面對面交流！