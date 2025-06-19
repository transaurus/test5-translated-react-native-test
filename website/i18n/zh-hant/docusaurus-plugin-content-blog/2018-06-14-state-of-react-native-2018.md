---
title: State of React Native 2018
author: Sophie Alpert
authorTitle: Engineering Manager on React at Facebook
authorURL: 'https://github.com/sophiebits'
authorImageURL: 'https://avatars2.githubusercontent.com/u/6820?s=460&v=4'
authorTwitter: sophiebits
tags: [engineering]
---

距離我們上一次發布關於 React Native 的狀態更新已經有一段時間了。

在 Facebook，我們比以往任何時候都更頻繁地使用 React Native，並將其應用於許多重要專案中。我們最受歡迎的產品之一是 Marketplace，這是我們應用程式中的頂級標籤之一，每月有 8 億人使用。自 2015 年創建以來，Marketplace 的所有功能都是使用 React Native 構建的，包括應用程式中不同部分的超過一百個全螢幕視圖。

我們也在應用程式的許多新部分中使用 React Native。如果你上個月觀看了 F8 主題演講，你會認出 Blood Donations、Crisis Response、Privacy Shortcuts 和 Wellness Checks——這些都是最近使用 React Native 構建的功能。而主 Facebook 應用程式之外的專案也在使用 React Native。新的 Oculus Go VR 頭戴裝置包含一個[配套的行動應用程式](https://www.oculus.com/app/)，該應用程式完全使用 React Native 構建，更不用說 React VR 為頭戴裝置本身提供了許多體驗。

當然，我們也使用許多其他技術來構建我們的應用程式。[Litho](https://fblitho.com/) 和 [ComponentKit](https://componentkit.org/) 是我們在應用程式中廣泛使用的兩個函式庫；兩者都提供了類似 React 的元件 API 來構建原生畫面。React Native 的目標從來不是取代所有其他技術——我們專注於讓 React Native 本身變得更好，但我們很高興看到其他團隊從 React Native 中借鑒想法，例如將[即時重新載入](https://instagram-engineering.com/instant-feedback-in-ios-engineering-workflows-c3f6508c76c8)帶入非 JavaScript 代碼中。

## 架構

當我們在 2013 年開始 React Native 專案時，我們設計了一個單一的「橋接器」來連接 JavaScript 和原生代碼，這個橋接器是非同步、可序列化和批次處理的。就像 React DOM 將 React 狀態更新轉換為對 DOM API 的命令式、可變的調用，如 `document.createElement(attrs)` 和 `.appendChild()`，React Native 被設計為返回一個單一的 JSON 消息，列出要執行的變更，如 `[["createView", attrs], ["manageChildren", ...]]`。我們設計了整個系統，使其永遠不需要依賴同步回應，並確保該列表中的所有內容都可以完全序列化為 JSON 並返回。我們這樣做是為了它帶給我們的靈活性：在這個架構之上，我們能夠構建像[Chrome 調試工具](/docs/debugging#chrome-developer-tools)這樣的工具，它通過 WebSocket 連接非同步運行所有 JavaScript 代碼。

在過去的 5 年中，我們發現這些初始原則使得構建某些功能變得更加困難。非同步橋接意味著你無法將 JavaScript 邏輯直接與許多需要同步回應的原生 API 集成。批次處理的橋接器會將原生調用排隊，這意味著讓 React Native 應用程式調用原生實現的函數變得更加困難。而可序列化的橋接意味著不必要的複製，而不是直接在兩個世界之間共享記憶體。對於完全使用 React Native 構建的應用程式，這些限制通常是可以忍受的。但對於在 React Native 和現有應用程式代碼之間有複雜集成的應用程式來說，這些限制令人沮喪。

**我們正在進行 React Native 的大規模重新架構，以使框架更加靈活，並更好地與混合 JavaScript/原生應用程式中的原生基礎設施集成。** 通過這個專案，我們將應用過去 5 年學到的知識，並逐步將我們的架構帶入更現代的狀態。我們正在重寫 React Native 的許多內部組件，但大多數變更都在底層：現有的 React Native 應用程式將繼續工作，幾乎不需要變更。

為了使 React Native 更加輕量級並更好地融入現有的原生應用程式，這次重新架構有三個主要的內部變更。首先，我們正在改變線程模型。不再需要每個 UI 更新在三個不同的線程上執行工作，而是可以在任何線程上同步調用 JavaScript 以進行高優先級更新，同時仍然將低優先級工作保持在主線程之外以保持響應性。其次，我們正在將[非同步渲染](https://reactjs.org/blog/2018/03/01/sneak-peek-beyond-react-16.html)功能納入 React Native，以允許多種渲染優先級並簡化非同步數據處理。最後，我們正在簡化我們的橋接器，使其更快、更輕量級；原生和 JavaScript 之間的直接調用更加高效，並且將使構建跨語言堆疊追踪等調試工具變得更加容易。

這些變更完成後，將能實現更緊密的整合。目前，若想整合原生導航、手勢處理或像是 UICollectionView 和 RecyclerView 這類原生元件，往往需要複雜的 hack 手段。在我們調整執行緒模型後，這類功能的開發將變得直觀許多。

我們將在今年稍晚這項工作接近完成時，釋出更多細節。

## 社群

除了 Facebook 內部的社群外，我們也很高興在外部擁有活躍的 React Native 使用者與協作者群體。我們希望透過更完善地服務 React Native 使用者，以及降低專案的貢獻門檻，來進一步支持這個社群。

正如我們的架構變更將使 React Native 能更乾淨地與其他原生基礎設施協作，React Native 在 JavaScript 端也應該更輕量化，以更好地融入 JavaScript 生態系——這包括讓虛擬機器（VM）與打包工具可替換。我們理解重大變更的節奏可能讓人難以跟上，因此希望能找到方法減少主要版本的發布頻率。此外，我們也意識到某些團隊需要更詳盡的文件（例如在啟動優化等領域，我們的專業知識尚未系統化記錄）。預計未來一年會逐步實現這些改進。

如果你正在使用 React Native，你就是我們社群的一員；請持續告訴我們如何讓 React Native 更符合你的需求。

React Native 只是行動開發者工具箱中的其中一項工具，但它是我們深信不疑的工具——我們每天都在讓它變得更好，過去一年已有超過 500 位貢獻者提交了 2500 多次 commit。