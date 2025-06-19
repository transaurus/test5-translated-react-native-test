---
title: 'New Architecture is here'
authors: [reactteam]
tags: [announcement]
date: 2024-10-23T16:01
---

React Native 0.76 版已預設啟用新架構，現已於 npm 上發布！

在 [0.76 版發布部落格文章](/blog/2024/10/23/release-0.76-new-architecture) 中，我們列出了此版本包含的重大變更。本文將概述新架構及其如何塑造 React Native 的未來。

新架構完整支援現代 React 功能，包含 [Suspense](https://react.dev/blog/2022/03/29/react-v18#new-suspense-features)、[Transitions](https://react.dev/blog/2022/03/29/react-v18#new-feature-transitions)、[自動批次處理](https://react.dev/blog/2022/03/29/react-v18#new-feature-automatic-batching) 及 [`useLayoutEffect`](https://react.dev/reference/react/useLayoutEffect)。新架構同時引入全新的 [原生模組系統](/docs/next/turbo-native-modules-introduction) 與 [原生元件系統](/docs/next/fabric-native-components-introduction)，讓您能撰寫型別安全的程式碼，並直接存取原生介面而無需透過橋接層。

此版本是我們自 2018 年起徹底重寫 React Native 的成果，我們特別注重讓新架構對多數應用程式保持漸進式遷移。2021 年，我們成立了 [新架構工作小組](https://github.com/reactwg/react-native-new-architecture/)，與社群共同確保整個 React 生態系的升級體驗流暢。

多數應用程式能以與其他版本相同的升級成本採用 React Native 0.76。最熱門的 React Native 函式庫已支援新架構。新架構同時內建自動相容層，可與針對舊架構開發的函式庫保持向後相容。

<!--truncate-->

在過去數年的開發過程中，我們的團隊已公開分享對新架構的願景。若您錯過這些演講，請參考以下內容：

- [React Native EU 2019 - 全新的 React Native](https://www.youtube.com/watch?v=52El0EUI6D0)
- [React Conf 2021 - React 18 主題演講](https://www.youtube.com/watch?v=FZ0cG47msEk)
- [App.js 2022 - 將 React Native 新架構帶入開源社群](https://www.youtube.com/watch?v=Q6TkkzRJfUo)
- [React Conf 2024 - 第二天主題演講](https://www.youtube.com/watch?v=Q5SMmKb7qVI)

## 什麼是新架構

新架構是對 React Native 底層核心系統的全面重寫，包含元件渲染機制、JavaScript 抽象層與原生抽象層的溝通方式，以及跨執行緒的工作排程系統。雖然多數使用者無需理解這些系統的運作原理，這些變革將帶來效能提升與新功能。

在舊架構中，React Native 透過非同步橋接層與原生平台溝通。無論是渲染元件或呼叫原生函式，React Native 都需將原生函式呼叫序列化並排入橋接佇列，以非同步方式處理。此架構的優勢在於主執行緒永遠不會因渲染更新或原生模組函式呼叫而阻塞，所有工作均在背景執行緒完成。

然而，使用者期望互動能獲得即時反饋以符合原生應用體驗。這意味著某些更新需同步渲染以響應用戶輸入，可能需中斷正在進行的渲染程序。由於舊架構僅支援非同步模式，我們必須重寫架構以同時支援非同步與同步更新。

此外，在舊架構中，透過橋接層序列化函式呼叫很快成為效能瓶頸，特別是高頻更新或大型物件傳遞時。這使得應用程式難以穩定維持 60+ FPS。同步化問題亦存在：當 JavaScript 層與原生層不同步時，無法立即同步協調，導致如清單顯示空白框架、視覺 UI 跳動等因中間狀態渲染產生的錯誤。

最後，由於舊架構使用原生層級結構保存單一 UI 副本並就地變更該副本，佈局計算只能在單一執行緒上進行。這使得無法處理使用者輸入等緊急更新，且無法同步讀取佈局（例如在佈局效果中讀取以更新工具提示位置）。

所有這些問題意味著無法正確支援 React 的並行功能。為解決這些問題，新架構包含四個主要部分：

- 全新原生模組系統
- 全新渲染器
- 事件循環
- 移除橋接層

新模組系統讓 React Native 渲染器能同步存取原生層，從而能同步/非同步處理事件、排程更新與讀取佈局。新原生模組預設也採用惰性載入，為應用帶來顯著效能提升。

新渲染器能跨多執行緒處理多個進行中的樹狀結構，讓 React 可處理多種並行更新優先級（主執行緒或背景執行緒皆可）。同時支援從多執行緒同步/非同步讀取佈局，以實現更流暢無卡頓的 UI 回應。

新事件循環能在 JavaScript 執行緒上依明確定義的順序處理任務。這讓 React 可中斷渲染來處理事件，使緊急使用者事件優先於低優先級 UI 轉場。事件循環也與網頁規格對齊，因此能支援微任務、`MutationObserver` 和 `IntersectionObserver` 等瀏覽器功能。

最後，移除橋接層可加速啟動並實現 JavaScript 與原生運行時的直接通訊，最小化工作切換成本。這也改善錯誤報告、除錯能力，並減少未定義行為導致的崩潰。

新架構現已準備好用於生產環境。Meta 已在 Facebook 應用等產品中大規模使用，我們也在為 [Quest 裝置](https://engineering.fb.com/2024/10/02/android/react-at-meta-connect-2024/) 開發的 Facebook 和 Instagram 應用中成功採用。

合作夥伴們已將新架構用於生產環境數月：可參考 [Expensify](https://blog.swmansion.com/sunrising-new-architecture-in-the-new-expensify-app-729d237a02f5) 與 [Kraken](https://blog.kraken.com/product/engineering/how-kraken-fixed-performance-issues-via-incremental-adoption-of-the-react-native-new-architecture) 的成功案例，並試用 [Bluesky](https://github.com/bluesky-social/social-app/releases/tag/1.92.0-na-rc.2) 的新版本。

### 全新原生模組

新原生模組系統徹底改寫 JavaScript 與原生平台的通訊方式，完全以 C++ 編寫並帶來多項新能力：

- 與原生運行時的同步雙向存取
- JavaScript 與原生程式碼間的型別安全
- 跨平台程式碼共享
- 預設惰性模組載入

在新系統中，JavaScript 與原生層現在能透過 JavaScript 介面 (JSI) 直接同步通訊，無需非同步橋接。這表示自訂原生模組現在能同步呼叫函式、回傳值並將值傳遞給其他原生模組函式。

舊架構中，要處理原生函式呼叫的回應，必須提供回呼函式且回傳值需可序列化：

```ts
// ❌ Sync callback from Native Module
nativeModule.getValue(value => {
  // ❌ value cannot reference a native object
  nativeModule.doSomething(value);
});
```

新架構中，可直接同步呼叫原生函式：

```ts
// ✅ Sync response from Native Module
const value = nativeModule.getValue();

// ✅ value can be a reference to a native object
nativeModule.doSomething(value);
```

新架構讓您終於能充分發揮 C++ 原生實作的威力，同時仍透過 JavaScript/TypeScript API 存取。新模組系統支援[以 C++ 編寫的模組](/docs/next/the-new-architecture/pure-cxx-modules)，實現「一次編寫，跨平台運行」（含 Android、iOS、Windows 和 macOS）。用 C++ 實作模組可進行更精細的記憶體管理與效能優化。

此外，透過 [Codegen](/docs/next/the-new-architecture/what-is-codegen)，您的模組可以在 JavaScript 層與原生層之間定義強型別的合約。根據我們的經驗，跨邊界型別錯誤是跨平台應用程式中最常見的當機原因之一。Codegen 能協助您解決這些問題，同時自動生成樣板程式碼。

最後，模組現在採用惰性載入機制：僅在實際需要時才會載入記憶體，而非在啟動時就預先載入。這減少了應用程式的啟動時間，並在應用程式複雜度增加時保持低載入時間。

熱門函式庫如 [react-native-mmkv](https://github.com/mrousavy/react-native-mmkv) 已從遷移至新原生模組系統中獲益：

> “The new Native Modules greatly simplified setup, autolinking, and initialization for `react-native-mmkv`. Thanks to the New Architecture, `react-native-mmkv` is now a pure C++ Native Module, which allows it to work on any platform. The new Codegen allows MMKV to be fully type-safe, which fixed a long-standing `NullPointerReference` issue by enforcing null-safety, and being able to call Native Module functions synchronously allowed us to replace custom JSI access with the new Native Module API.”
>
> [Marc Rousavy](https://twitter.com/mrousavy), creator of `react-native-mmkv`

### 新渲染器

我們也徹底重寫了原生渲染器，帶來多項優勢：

- 更新可在不同執行緒上以不同優先級渲染。
- 佈局可跨執行緒同步讀取。
- 渲染器以 C++ 編寫並跨平台共享。

更新後的原生渲染器現在將視圖層級結構儲存為不可變的樹狀結構。這意味著 UI 以無法直接修改的方式儲存，允許安全地跨執行緒處理更新。它能同時處理多個進行中的樹狀結構，每個代表使用者介面的不同版本。因此，更新可在背景渲染而不阻塞 UI（例如轉場動畫期間），或在主執行緒渲染（回應使用者輸入）。

透過支援多執行緒，React 能中斷低優先級更新來渲染緊急更新（如使用者輸入觸發的更新），並在需要時恢復低優先級更新。新渲染器還能跨執行緒同步讀取佈局資訊，這使得低優先級更新能在背景計算，而需要時（例如重新定位工具提示）可進行同步讀取。

最後，以 C++ 重寫渲染器使其能跨所有平台共享，確保相同程式碼在 iOS、Android、Windows、macOS 及其他 React Native 支援的平台上運行，提供一致的渲染能力而無需針對各平台重新實作。

這是實現我們 [多平台願景](/blog/2021/08/26/many-platform-vision) 的重要一步。例如，視圖扁平化原本是 Android 專屬的優化技術，用於避免深層佈局樹。擁有共享 C++ 核心的新渲染器 [將此功能帶到 iOS](https://github.com/reactwg/react-native-new-architecture/discussions/110)。此優化是自動化的，無需額外設定，隨共享渲染器免費獲得。

透過這些變更，React Native 現在完整支援並行 React 功能（如 Suspense 和 Transitions），讓開發者能更輕鬆建構複雜的使用者介面，快速響應使用者輸入而不會出現卡頓、延遲或視覺跳動。未來我們將利用這些新功能為內建元件（如 FlatList 和 TextInput）帶來更多改進。

熱門函式庫如 [Reanimated](https://docs.swmansion.com/react-native-reanimated/) 已開始運用新渲染器的優勢：

> “Reanimated 4, currently in development, introduces a new animation engine that works directly with the New Renderer, allowing it to handle animations and manage layout across different threads. The New Renderer’s design is what truly enables these features to be built without relying on numerous workarounds. Moreover, because it’s implemented in C++ and shared across platforms, large portions of Reanimated can be written once, reducing platform-specific issues, minimizing the codebase, and streamlining adoption for out-of-tree platforms.”
>
> [Krzysztof Magiera](https://x.com/kzzzf), creator of [Reanimated](https://docs.swmansion.com/react-native-reanimated/)

### 事件循環機制

新架構讓我們能實作明確定義的事件循環處理模型，如這份 [RFC](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0744-well-defined-event-loop.md) 所述。該提案遵循 [HTML 標準](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model) 規範，明確定義 React Native 應如何在 JavaScript 線程執行任務。

明確定義的事件循環縮小了 React DOM 與 React Native 的差距：React Native 應用的行為現在更接近 React DOM 應用，實現「學習一次，隨處編寫」。

事件循環為 React Native 帶來多項優勢：

- 可中斷渲染以處理事件和任務
- 更貼近網頁技術規範
- 為更多瀏覽器功能奠定基礎

透過事件循環，React 能可預測地排序更新與事件。這讓 React 能優先處理緊急用戶事件，中斷低優先級更新，而新渲染器則允許我們獨立渲染這些更新。

事件循環也使計時器等任務行為更符合網頁規範，意味著 React Native 的運作方式更貼近開發者熟悉的網頁環境，有利於 React DOM 與 React Native 間的程式碼共享。

這也為實作更完整的瀏覽器功能（如微任務、`MutationObserver` 和 `IntersectionObserver`）奠定基礎。這些功能尚未開放使用，但我們正積極開發中。

最後，事件循環與新渲染器支援同步讀取佈局的特性，讓 React Native 能完整實作 `useLayoutEffect`，在同幀內同步讀取佈局資訊並更新 UI，確保元素在顯示給用戶前已正確定位。

詳見 [`useLayoutEffect`](/blog/2024/10/23/the-new-architecture-is-here#uselayouteffect) 章節說明。

### 移除橋接層

新架構中，我們徹底移除了 React Native 對橋接層的依賴，改採 JSI 實現 JavaScript 與原生代碼間的直接高效通訊：

![](/blog/assets/0.76-bridge-diagram.png)

移除橋接層可避免初始化延遲，改善啟動速度。例如舊架構中，為了向 JavaScript 提供全局方法，必須在啟動時初始化 JavaScript 模組，導致輕微延遲：

```js
// ❌ Slow initialization
import {NativeTimingModule} from 'NativeTimingModule';
global.setTimeout = timer => {
  NativeTimingModule.setTimeout(timer);
};

// App.js
setTimeout(() => {}, 100);
```

新架構中，我們能直接從 C++ 綁定方法：

```cpp
// ✅ Initialize directly in C++
runtime.global().setProperty(runtime, "setTimeout", createTimer);
```

```js
// App.js
setTimeout(() => {}, 100);
```

此重構也強化了錯誤報告機制（特別是啟動時的 JavaScript 崩潰），並減少未定義行為導致的崩潰。若發生崩潰，新版 [React Native DevTools](/docs/next/react-native-devtools) 能簡化除錯流程並完整支援新架構。

橋接層目前仍保留以支援漸進式遷移，未來將完全移除相關代碼。

### 漸進式遷移

我們預期多數應用程式升級至 0.76 版本的工作量與常規版本更新相當。

當您升級至 0.76 版本時，新架構與 React 18 將預設啟用。然而，若要使用並行功能並充分獲得新架構的優勢，您的應用程式與函式庫需逐步遷移以完整支援新架構。

初次升級時，您的應用程式會在新架構上運行，並透過自動互操作性層與舊架構相容。多數應用程式無需修改即可運作，但互操作性層存在[已知限制](https://github.com/reactwg/react-native-new-architecture/discussions/237)，例如不支援存取自訂陰影節點或並行功能。

若要使用並行功能，應用程式還需更新以支援[並行 React](https://react.dev/blog/2022/03/29/react-v18#what-is-concurrent-react)，並遵循 [React 規則](https://react.dev/reference/rules)。請參照 [React 18 升級指南](https://react.dev/blog/2022/03/08/react-18-upgrade-guide)遷移您的 JavaScript 程式碼至 React 18 及其語義。

整體策略是讓您的應用程式在新架構上運行而不破壞現有程式碼。之後您可依自身步調逐步遷移。對於已遷移所有模組至新架構的新介面，您可立即開始使用並行功能。現有介面則可能需要解決部分問題並遷移模組後，才能新增並行功能。

我們已與最熱門的 React Native 函式庫合作，確保其支援新架構。超過 850 個函式庫已相容，包含所有每週下載量超過 20 萬次的函式庫（約佔下載函式庫的 10%）。您可於 [reactnative.directory](https://reactnative.directory) 網站檢查函式庫與新架構的相容性：

![](/blog/assets/0.76-directory.png)

更多升級細節，請參閱下方的[如何升級](/blog/2024/10/23/the-new-architecture-is-here#how-to-upgrade)。

## 新功能

新架構包含對 React 18、並行功能及 React Native 中 `useLayoutEffect` 的完整支援。React 18 完整功能清單請參閱 [React 18 部落格文章](https://react.dev/blog/2021/12/17/react-conf-2021-recap#react-18-and-concurrent-features)。

### 轉場效果

轉場效果是 React 18 的新概念，用於區分緊急與非緊急更新。

- **緊急更新**反映直接互動，如輸入與點擊。
- **轉場更新**將介面從一個畫面轉換至另一個。

緊急更新需立即回應以符合我們對實體物件行為的直覺。但轉場效果不同，因為使用者不預期在畫面上看到每個中間值。新架構中，React Native 能分別支援渲染緊急更新與轉場更新。

通常，為達最佳使用者體驗，單一使用者輸入應同時觸發緊急與非緊急更新。與 ReactDOM 類似，`press` 或 `change` 等事件會被視為緊急處理並立即渲染。您可在輸入事件中使用 `startTransition` API 告知 React 哪些更新屬「轉場」並可延遲至背景處理：

```jsx
import {startTransition} from 'react';

// Urgent: Show the slider value
setCount(input);

// Mark any state updates inside as transitions
startTransition(() => {
  // Transition: Show the results
  setNumberOfTiles(input);
});
```

分離緊急事件與轉場效果能創造更靈敏的使用者介面與更直覺的使用體驗。

此為舊架構（無轉場效果）與新架構（含轉場效果）的比較。假設每個區塊非僅是帶背景色的簡單視圖，而是包含圖片與其他昂貴渲染元件的豐富元件。**使用** `useTransition` 後，您可避免應用程式因更新而卡頓或落後。

<div className="TwoColumns TwoFigures">
<figure>
 <img src="/img/new-architecture/without-transitions.gif" alt="A video demonstrating an app rendering many views (tiles) according to a slider input. The views are rendered in batches as the slider is quickly adjusted from 0 to 1000." />
 <figcaption><b>Before:</b> rendering tiles without marking it as a transition.</figcaption>
</figure>
<figure>
 <img src="/img/new-architecture/with-transitions.gif" alt="A video demonstrating an app rendering many views (tiles) according to a slider input. The views are rendered in batches as the slider is quickly adjusted from 0 to 1000. There are less batch renders in comparison to the next video." />
 <figcaption><b>After:</b> rendering tiles <em>with transitions</em> to interrupt in-progress renders of stale state.</figcaption>
</figure>
</div>

更多資訊請參閱[並行渲染器與功能支援](/docs/0.75/the-new-architecture/landing-page#support-for-concurrent-renderer-and-features)。

### 自動批次處理

當您升級至新架構時，將自動獲得 React 18 的批次處理功能優勢。

自動批次處理允許 React 在渲染時將多個狀態更新批次處理，避免渲染中間狀態。這使得 React Native 運行更快速且不易延遲，無需開發者額外編寫程式碼。

<div className="TwoColumns TwoFigures">
<figure>
 <img src="/img/new-architecture/legacy-renderer.gif" alt="A video demonstrating an app rendering many views according to a slider input. The slider value is adjusted from 0 to 1000 and the UI slowly catches up to rendering 1000 views." />
 <figcaption><b>Before:</b> rendering frequent state updates with legacy renderer.</figcaption>
</figure>
<figure>
 <img src="/img/new-architecture/react18-renderer.gif" alt="A video demonstrating an app rendering many views according to a slider input. The slider value is adjusted from 0 to 1000 and the UI resolves to 1000 views faster than the previous example, without as many intermediate states." />
 <figcaption><b>After:</b> rendering frequent state updates with <em>automatic batching</em>.</figcaption>
</figure>
</div>

在舊架構中，會渲染更多中間狀態，即使滑桿停止移動，UI 仍持續更新。新架構透過自動批次處理更新，渲染更少的中間狀態並更快完成渲染。

更多資訊請參閱[並行渲染器與功能支援](/docs/0.75/the-new-architecture/landing-page#support-for-concurrent-renderer-and-features)。

### useLayoutEffect

基於事件循環與同步讀取佈局的能力，新架構中我們為 React Native 的 `useLayoutEffect` 添加了完整支援。

在舊架構中，您需要使用非同步的 `onLayout` 事件來讀取視圖的佈局資訊（同樣是非同步的）。這會導致至少有一幀的佈局顯示錯誤，直到佈局被讀取並更新，造成如工具提示位置錯誤等問題：

```tsx
// ❌ async onLayout after commit
const onLayout = React.useCallback(event => {
  // ❌ async callback to read layout
  ref.current?.measureInWindow((x, y, width, height) => {
    setPosition({x, y, width, height});
  });
}, []);

// ...
<ViewWithTooltip
  onLayout={onLayout}
  ref={ref}
  position={position}
/>;
```

新架構透過允許在 `useLayoutEffect` 中同步存取佈局資訊來解決此問題：

```tsx
// ✅ sync layout effect during commit
useLayoutEffect(() => {
  // ✅ sync call to read layout
  const rect = ref.current?.getBoundingClientRect();
  setPosition(rect);
}, []);

// ...
<ViewWithTooltip ref={ref} position={position} />;
```

此變更讓您能同步讀取佈局資訊並在同一幀中更新 UI，確保元素在顯示給使用者前已正確定位：

<div className="TwoColumns TwoFigures">
<figure>
 <img src="/img/new-architecture/async-on-layout.gif" alt="A view that is moving to the corners of the viewport and center with a tooltip rendered either above or below it. The tooltip is rendered after a short delay after the view moves" />
 <figcaption>In the old architecture, layout was read asynchronously in `onLayout`, causing the position of the tooltip to be delayed.</figcaption>
</figure>
<figure>
 <img src="/img/new-architecture/sync-use-layout-effect.gif" alt="A view that is moving to the corners of the viewport and center with a tooltip rendered either above or below it. The view and tooltip move in unison." />
 <figcaption>In the New Architecture, layout can be read in `useLayoutEffect` synchronously, updating the tooltip position before displaying.</figcaption>
</figure>
</div>

更多資訊請參閱[同步佈局與效果](/docs/0.75/the-new-architecture/landing-page#synchronous-layout-and-effects)文件。

### 完整支援 Suspense

Suspense 讓您能宣告式地指定元件樹中尚未準備好顯示部分的載入狀態：

```jsx
<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

我們數年前引入了有限版本的 Suspense，而 React 18 新增了完整支援。在此之前，React Native 無法支援 Suspense 的並行渲染。

新架構包含 React 18 引入的完整 Suspense 支援。這意味著您現在可以在 React Native 中使用 Suspense 處理元件的載入狀態，被暫停的內容會在背景渲染同時顯示載入狀態，並優先處理使用者對可見內容的輸入。

更多資訊請參閱[React 18 的 Suspense RFC](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md)。

## 如何升級

要升級至 0.76 版，請遵循[發布公告](/blog/2024/10/23/release-0.76-new-architecture#upgrade-to-076)中的步驟。由於此版本同時升級至 React 18，您還需遵循[React 18 升級指南](https://react.dev/blog/2022/03/08/react-18-upgrade-guide)。

由於新架構與舊架構的互操作層，這些步驟應足以讓大多數應用程式完成升級。但要充分發揮新架構優勢並使用並行功能，您需將自訂原生模組與原生元件遷移至支援新的原生模組與元件 API。

若不遷移自訂原生模組，您將無法獲得共享 C++、同步方法呼叫或程式碼生成類型安全等優勢。若不遷移原生元件，您將無法使用並行功能。我們建議盡快將所有原生元件與模組遷移至新架構。

:::note
在未來版本中，我們將移除互操作層，模組需支援新架構。
:::

### 應用程式

如果您是應用程式開發者，要完整支援新架構，您需要將您的函式庫、自訂原生元件和自訂原生模組升級以全面支援新架構。

我們已與最受歡迎的 React Native 函式庫合作，確保它們支援新架構。您可以在 [reactnative.directory](https://reactnative.directory) 網站上檢查函式庫與新架構的相容性。

如果您的應用程式依賴的任何函式庫尚未相容，您可以：

- Open an issue with the library and ask the author to migrate to the New Architecture.
- If the library is not maintained, consider alternative libraries with the same features.
- [Opt-out from the New Architecture](/blog/2024/10/23/the-new-architecture-is-here#opt-out) while those libraries are migrated.

如果您的應用程式有自訂原生模組或自訂原生元件，由於我們的[互操作層](https://github.com/reactwg/react-native-new-architecture/discussions/135)，它們應該能正常運作。然而，我們建議將它們升級至新的原生模組和原生元件 API，以全面支援新架構並採用並行功能。

請遵循以下指南將您的模組和元件遷移至新架構：

- [原生模組](/docs/next/turbo-native-modules-introduction)
- [原生元件](/docs/next/fabric-native-components-introduction)

### 函式庫

如果您是函式庫維護者，請先測試您的函式庫是否能與互操作層一起運作。如果不能，請在[新架構工作小組](https://github.com/reactwg/react-native-new-architecture/)提交問題。

為了全面支援新架構，我們建議您盡快將函式庫遷移至新的原生模組和原生元件 API。這將讓您的函式庫使用者能充分利用新架構並支援並行功能。

您可以遵循以下指南將模組和元件遷移至新架構：

- [原生模組](/docs/next/turbo-native-modules-introduction)
- [原生元件](/docs/next/fabric-native-components-introduction)

### 選擇退出

如果由於任何原因，新架構在您的應用程式中無法正常運作，您隨時可以選擇退出，直到您準備好再次啟用它。

要選擇退出新架構：

- 在 Android 上，修改 `android/gradle.properties` 檔案並關閉 `newArchEnabled` 標誌

```diff title=”gradle.properties”
-newArchEnabled=true
+newArchEnabled=false
```

- 在 iOS 上，您可以執行以下命令重新安裝依賴項：

```shell
RCT_NEW_ARCH_ENABLED=0 bundle exec pod install
```

## 致謝

將新架構交付給開源社群是一項巨大的努力，耗費了我們數年的研究與開發。我們想藉此機會感謝所有幫助我們達成這一成果的 React 團隊現任與前任成員。

我們也非常感謝所有與我們合作實現這一目標的合作夥伴。具體來說，我們要特別感謝：

- [Expo](https://expo.dev/)，感謝他們早期採用新架構，並協助遷移最受歡迎的函式庫。
- [Software Mansion](https://swmansion.com/)，感謝他們維護生態系統中的關鍵函式庫，早期遷移至新架構，並協助調查和修復各種問題。
- [Callstack](https://www.callstack.com/)，感謝他們維護生態系統中的關鍵函式庫，早期遷移至新架構，並支援社群 CLI 的開發工作。
- [Microsoft](https://opensource.microsoft.com/)，感謝他們為 `react-native-windows` 和 `react-native-macos` 添加新架構實現，以及開發多項其他開發者工具。
- [Expensify](https://www.expensify.com/)、[Kraken](https://www.kraken.com/)、[Bluesky](https://bsky.app/) 和 [Brigad](https://www.brigad.co/)，感謝他們率先採用新架構並回報各種問題，讓我們能為所有人修復這些問題。
- 所有獨立函式庫維護者和開發者，感謝他們測試新架構、修復部分問題，並提出不清楚的事項讓我們能釐清。