---
id: roottag
title: RootTag
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

`RootTag` 是一個不透明的識別符，用於標記 React Native 介面的原生根視圖——即 Android 的 `ReactRootView` 或 iOS 的 `RCTRootView` 實例。簡而言之，它是一個介面識別符。

## 何時需要使用 RootTag？

對於大多數 React Native 開發者來說，您可能不需要直接處理 `RootTag`。

`RootTag`s are useful for when an app renders **multiple React Native root views** and you need to handle native API calls differently depending on the surface. An example of this is when an app is using native navigation and each screen is a separate React Native root view.

在原生導航中，每個 React Native 根視圖都會渲染在平台的導航視圖中（例如 Android 的 `Activity` 或 iOS 的 `UINavigationViewController`）。這樣可以充分利用平台的導航特性，例如原生外觀和導航過渡效果。與原生導航 API 互動的功能可以通過[原生模組](https://reactnative.dev/docs/next/native-modules-intro)暴露給 React Native。

舉例來說，若要更新某個畫面的標題欄，您需要調用導航模組的 API `setTitle("Updated Title")`，但該 API 需要知道要更新導航堆疊中的哪個畫面。此時就需要 `RootTag` 來識別根視圖及其宿主容器。

另一個使用 `RootTag` 的情況是當應用程式需要根據 JavaScript 調用的來源根視圖來區分處理原生調用時。`RootTag` 可用於區分來自不同介面的調用來源。

## 如何存取 RootTag（如果需要）

在 0.65 及以下版本中，RootTag 是通過[舊版 Context](https://github.com/facebook/react-native/blob/v0.64.1/Libraries/ReactNative/AppContainer.js#L56) 存取的。為了讓 React Native 準備迎接 React 18 及更高版本的 Concurrent 特性，我們在 0.66 版本中遷移到了最新的 [Context API](https://reactjs.org/docs/context.html#api)，即通過 `RootTagContext` 存取。版本 0.65 同時支援舊版 Context 和推薦的 `RootTagContext`，以便開發者有時間遷移調用點。詳見重大變更摘要。

How to access `RootTag` via the `RootTagContext`.

```js
import {RootTagContext} from 'react-native';
import NativeAnalytics from 'native-analytics';
import NativeNavigation from 'native-navigation';

function ScreenA() {
  const rootTag = useContext(RootTagContext);

  const updateTitle = title => {
    NativeNavigation.setTitle(rootTag, title);
  };

  const handleOneEvent = () => {
    NativeAnalytics.logEvent(rootTag, 'one_event');
  };

  // ...
}

class ScreenB extends React.Component {
  static contextType: typeof RootTagContext = RootTagContext;

  updateTitle(title) {
    NativeNavigation.setTitle(this.context, title);
  }

  handleOneEvent() {
    NativeAnalytics.logEvent(this.context, 'one_event');
  }

  // ...
}
```

關於 Context API 的更多資訊，請參閱 React 文件中關於[類別](https://reactjs.org/docs/context.html#classcontexttype)和[鉤子](https://reactjs.org/docs/hooks-reference.html#usecontext)的說明。

### 0.65 版本的重大變更

`RootTagContext` 在 0.65 版本之前名為 `unstable_RootTagContext`，並在 0.65 版本中更名為 `RootTagContext`。請更新程式碼中所有使用 `unstable_RootTagContext` 的地方。

### 0.66 版本的重大變更

舊版 Context 存取 `RootTag` 的方式將被移除，並由 `RootTagContext` 取代。從 0.65 版本開始，我們建議開發者主動將 `RootTag` 的存取方式遷移到 `RootTagContext`。

## 未來計劃

隨著新的 React Native 架構不斷推進，`RootTag` 將會有進一步的迭代，目的是保持 `RootTag` 類型的不透明性並減少 React Native 程式碼庫的變動。請不要依賴目前 RootTag 實際上是數值類型的特性！如果您的應用程式依賴 RootTag，請密切關注我們的版本變更日誌，您可以在[這裡](https://github.com/facebook/react-native/blob/main/CHANGELOG.md)找到相關資訊。