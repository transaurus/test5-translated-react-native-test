---
id: roottag
title: RootTag
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

`RootTag` 是一個不透明的標識符，用於標識 React Native 表面的原生根視圖——即 Android 或 iOS 上的 `ReactRootView` 或 `RCTRootView` 實例。簡而言之，它是一個表面標識符。

## 何時需要使用 RootTag？

對於大多數 React Native 開發者來說，您可能不需要直接處理 `RootTag`。

`RootTag`s are useful for when an app renders **multiple React Native root views** and you need to handle native API calls differently depending on the surface. An example of this is when an app is using native navigation and each screen is a separate React Native root view.

在原生導航中，每個 React Native 根視圖都會渲染在平台的導航視圖中（例如 Android 的 `Activity` 或 iOS 的 `UINavigationViewController`）。這樣，您可以利用平台的導航範式，例如原生的外觀和導航過渡。與原生導航 API 交互的功能可以通過[原生模塊](https://reactnative.dev/docs/next/native-modules-intro)暴露給 React Native。

例如，要更新屏幕的標題欄，您需要調用導航模塊的 API `setTitle("Updated Title")`，但它需要知道要更新堆棧中的哪個屏幕。此時，`RootTag` 就是用來標識根視圖及其宿主容器的必要工具。

另一個使用 `RootTag` 的情況是當您的應用需要根據 JavaScript 調用的來源視圖來區分原生調用時。`RootTag` 可以幫助區分來自不同表面的調用源。

## 如何訪問 RootTag（如果需要）

In versions 0.65 and below, RootTag is accessed via a [legacy context](https://github.com/facebook/react-native/blob/v0.64.1/Libraries/ReactNative/AppContainer.js#L56). To prepare React Native for Concurrent features coming in React 18 and beyond, we are migrating to the latest [Context API](https://reactjs.org/docs/context.html#api) via `RootTagContext` in 0.66. Version 0.65 supports both the legacy context and the recommended `RootTagContext` to allow developers time to migrate their call-sites. See the breaking changes summary.

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

從 React 文檔中了解更多關於[類](https://reactjs.org/docs/context.html#classcontexttype)和[鉤子](https://reactjs.org/docs/hooks-reference.html#usecontext)的上下文 API。

### 0.65 版本的重大變更

`RootTagContext` 在 0.65 版本之前名為 `unstable_RootTagContext`，並在 0.65 版本中更名為 `RootTagContext`。請更新代碼庫中所有使用 `unstable_RootTagContext` 的地方。

### 0.66 版本的重大變更

舊版上下文訪問 `RootTag` 的方式將被移除，並由 `RootTagContext` 取代。從 0.65 版本開始，我們鼓勵開發者主動將 `RootTag` 的訪問方式遷移到 `RootTagContext`。

## 未來計劃

隨著新的 React Native 架構的推進，`RootTag` 將會有進一步的迭代，目的是保持 `RootTag` 類型的不透明性並減少 React Native 代碼庫的變動。請不要依賴當前 RootTag 實際上是數字類型的特性！如果您的應用依賴 RootTag，請密切關注我們的版本變更日誌，您可以在[這裡](https://github.com/facebook/react-native/blob/main/CHANGELOG.md)找到它們。