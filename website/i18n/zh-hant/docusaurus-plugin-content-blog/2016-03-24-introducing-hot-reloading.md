---
title: Introducing Hot Reloading
author: Martín Bigio
authorTitle: Software Engineer at Instagram
authorURL: 'https://twitter.com/martinbigio'
authorImageURL: 'https://avatars3.githubusercontent.com/u/535661?v=3&s=128'
authorTwitter: martinbigio
tags: [engineering]
---

React Native 的目標是為開發者提供最佳的開發體驗。其中關鍵環節在於從保存文件到查看變更所需的時間。我們的目標是將這個反饋循環控制在 1 秒內，即使您的應用規模不斷擴大。

我們通過三大核心功能接近這個理想狀態：

- 使用 JavaScript 作為開發語言，因其編譯週期短暫。
- 開發名為 Packager 的工具，將 es6/flow/jsx 文件轉換為虛擬機可理解的標準 JavaScript。該工具設計為服務器模式，將中間狀態保留在內存中以實現快速增量更新，並充分利用多核處理能力。
- 構建名為 Live Reload（即時重載）的功能，在保存時自動刷新應用。

至此，開發者的瓶頸不再是應用重載時間，而是應用狀態的丟失。典型場景是：當您開發的功能需要經過多個頁面才能到達時，每次重載後都需重複操作相同路徑才能返回開發界面，導致每次修改的週期延長至數秒。

## 熱重載（Hot Reloading）

熱重載的核心原理是保持應用運行狀態，在運行時注入修改後的新版文件。這種方式能完整保留當前狀態，對於調整 UI 界面時尤其實用。

百聞不如一見。請觀看即時重載（現有）與熱重載（新功能）的實際對比效果：

<iframe
  width="100%"
  height="315"
  src="https://www.youtube.com/embed/2uQzVi-KFuc"
  frameborder="0"
  allowfullscreen></iframe>

仔細觀察可發現：系統能從紅屏錯誤中恢復，且無需完全重載即可動態導入新增模塊。

**重要提示：** 由於 JavaScript 是高度狀態化的語言，熱重載無法實現完美無缺。實際應用中，當前方案已能覆蓋大多數常見使用場景，若出現異常隨時可進行完整重載。

熱重載功能自 0.22 版本起提供，啟用方式如下：

- 打開開發者選單
- 點擊「啟用熱重載」

## 實現原理概覽

了解其價值與使用方法後，現在進入最有趣的部分：技術實現原理。

熱重載基於 [Hot Module Replacement (HMR)](https://webpack.js.org/guides/hot-module-replacement/) 功能構建，該技術最初由 webpack 提出，我們在 React Native Packager 中實現了該機制。HMR 會監控文件變更，並通過內置於應用的輕量級 HMR 運行時接收更新。

簡而言之，HMR 更新包含發生變更的 JS 模塊新代碼。當運行時接收更新後，會用新代碼替換舊模塊：

![](/blog/assets/hmr-architecture.png)

HMR 更新內容不僅包含待修改的模塊代碼，因為單純替換代碼不足以讓運行時獲取變更。問題在於模塊系統可能已緩存了待更新模塊的 _導出對象_。例如假設應用由以下兩個模塊組成：

```
// log.js
function log(message) {
  const time = require('./time');
  console.log(`[${time()}] ${message}`);
}

module.exports = log;
```

```
// time.js
function time() {
  return new Date().getTime();
}

module.exports = time;
```

模塊 `log` 會輸出包含當前時間的訊息，時間數據由模塊 `time` 提供。

當應用打包時，React Native 會通過 `__d` 函數在模塊系統中註冊每個模塊。對於本示例應用，在眾多 `__d` 定義中會包含 `log` 模塊的定義：

```
__d('log', function() {
  ... // module's code
});
```

此調用將每個模組的程式碼封裝到一個匿名函數中，我們通常稱之為工廠函數。模組系統運行時會追蹤每個模組的工廠函數、是否已執行過，以及執行結果（導出內容）。當需要某個模組時，模組系統會提供已緩存的導出內容，或首次執行該模組的工廠函數並保存結果。

假設你啟動應用並載入 `log`。此時，`log` 和 `time` 的工廠函數都尚未執行，因此沒有導出內容被緩存。接著，使用者修改 `time` 使其返回 `MM/DD` 格式的日期：

```js
// time.js
function bar() {
  const date = new Date();
  return `${date.getMonth() + 1}/${date.getDate()}`;
}

module.exports = bar;
```

The Packager will send time's new code to the runtime (step 1), and when `log` gets eventually required the exported function gets executed it will do so with `time`'s changes (step 2):

![](/blog/assets/hmr-step.png)

現在假設 `log` 的程式碼在頂層載入 `time`：

```
const time = require('./time'); // top level require

// log.js
function log(message) {
  console.log(`[${time()}] ${message}`);
}

module.exports = log;
```

當 `log` 被載入時，運行時會緩存其導出內容和 `time` 的導出內容（步驟1）。接著，當 `time` 被修改時，HMR 流程不能僅在替換 `time` 的程式碼後就結束。如果這樣做，當 `log` 執行時，它會使用緩存的 `time`（舊程式碼）。

為了讓 `log` 獲取 `time` 的變更，我們需要清除其緩存的導出內容，因為它所依賴的模組之一已被熱替換（步驟3）。最後，當 `log` 再次被載入時，其工廠函數會執行並載入 `time` 的新程式碼。

![](/blog/assets/hmr-log.png)

## HMR API

HMR in React Native extends the module system by introducing the `hot` object. This API is based on [webpack](https://webpack.github.io/hot-module-replacement.md)'s one. The `hot` object exposes a function called `accept` which allows you to define a callback that will be executed when the module needs to be hot swapped. For instance, if we would change `time`'s code as follows, every time we save time, we'll see “time changed” in the console:

```
// time.js
function time() {
  ... // new code
}

module.hot.accept(() => {
  console.log('time changed');
});

module.exports = time;
```

請注意，只有在極少數情況下你需要手動使用此 API。熱重載在大多數常見用例中應能直接使用。

## HMR 運行時

如前所述，有時僅接受 HMR 更新是不夠的，因為使用熱替換模組的模組可能已被執行且其導入內容已緩存。例如，假設電影應用範例的依賴樹中有一個頂層 `MovieRouter`，它依賴於 `MovieSearch` 和 `MovieScreen` 視圖，而這些視圖又依賴於先前範例中的 `log` 和 `time` 模組：

![](/blog/assets/hmr-diamond.png)

如果使用者訪問了電影搜索視圖但未訪問其他視圖，則除 `MovieScreen` 外的所有模組都會有緩存的導出內容。如果對 `time` 模組進行了修改，運行時必須清除 `log` 的導出內容，以便它獲取 `time` 的變更。流程不會在此結束：運行時會遞歸地重複此過程，直到所有父模組都被接受。因此，它會抓取依賴於 `log` 的模組並嘗試接受它們。對於 `MovieScreen`，它可以跳過，因為它尚未被載入。對於 `MovieSearch`，它必須清除其導出內容並遞歸處理其父模組。最後，它會對 `MovieRouter` 執行相同操作並在此結束，因為沒有模組依賴於它。

為了遍歷依賴樹，運行時會在 HMR 更新中從打包工具接收反向依賴樹。對於此範例，運行時會收到如下 JSON 物件：

```
{
  modules: [
    {
      name: 'time',
      code: /* time's new code */
    }
  ],
  inverseDependencies: {
    MovieRouter: [],
    MovieScreen: ['MovieRouter'],
    MovieSearch: ['MovieRouter'],
    log: ['MovieScreen', 'MovieSearch'],
    time: ['log'],
  }
}
```

## React 元件

React components are a bit harder to get to work with Hot Reloading. The problem is that we can't simply replace the old code with the new one as we'd loose the component's state. For React web applications, [Dan Abramov](https://twitter.com/dan_abramov) implemented a babel [transform](https://gaearon.github.io/react-hot-loader/) that uses webpack's HMR API to solve this issue. In a nutshell, his solution works by creating a proxy for every single React component on _transform time_. The proxies hold the component's state and delegate the lifecycle methods to the actual components, which are the ones we hot reload:

![](/blog/assets/hmr-proxy.png)

除了建立代理元件外，此轉換器還會定義 `accept` 函數，其中包含強制 React 重新渲染元件的程式碼。如此一來，我們能在不損失應用狀態的情況下熱重載渲染邏輯。

React Native 內建的[轉換器](https://github.com/facebook/react-native/blob/master/packager/transformer.js#L92-L95)使用 `babel-preset-react-native`，其[配置](https://github.com/facebook/react-native/blob/master/babel-preset/configs/hmr.js#L24-L31)會啟用 `react-transform`，用法與網頁版 React 專案使用 webpack 時完全相同。

## Redux 儲存庫

若要為 [Redux](https://redux.js.org/) 儲存庫啟用熱重載，只需像在 webpack 專案中那樣使用 HMR API：

```
// configureStore.js
import { createStore, applyMiddleware, compose } from 'redux';
import thunk from 'redux-thunk';
import reducer from '@site/reducers';

export default function configureStore(initialState) {
  const store = createStore(
    reducer,
    initialState,
    applyMiddleware(thunk),
  );

  if (module.hot) {
    module.hot.accept(() => {
      const nextRootReducer = require('../reducers/index').default;
      store.replaceReducer(nextRootReducer);
    });
  }

  return store;
};
```

當修改 reducer 時，接受該 reducer 的程式碼會被發送至客戶端。客戶端會發現 reducer 本身無法處理熱更新，因此會找出所有引用此 reducer 的模組並嘗試接受更新。最終，流程會抵達單一儲存庫（即 `configureStore` 模組），由它來接受 HMR 更新。

## 結論

若您有意協助改進熱重載功能，建議閱讀 [Dan Abramov 關於熱重載未來的文章](https://medium.com/@dan_abramov/hot-reloading-in-react-1140438583bf#.jmivpvmz4)並參與貢獻。例如 Johny Days 正致力於[實現多客戶端同步熱重載](https://github.com/facebook/react-native/pull/6179)。此功能的維護與改進需要大家共同投入。

React Native 讓我們能重新思考應用開發方式以提升開發者體驗。熱重載只是其中一環，我們還能施展哪些瘋狂技巧來讓它更完善？