---
title: Pointer Events in React Native
authors: [lunaleaps, vincentriemer]
tags: [announcement]
date: 2022-12-13
---

# React Native 中的指針事件

今天我們將分享 React Native 的實驗性跨平台指針 API。我們將探討其動機、運作原理以及對 React Native 使用者的益處。文中包含啟用方法說明，我們期待聽到您的反饋！

自我們發表[多平台願景](https://reactnative.dev/blog/2021/08/26/many-platform-vision)闡述跨行動平台開發優勢已逾一年。這段期間，我們加強了對 VR、桌面端和網頁版 React Native 的投入。這些平台在硬體與互動方式上的差異，促使我們思考 React Native 應如何全局處理輸入問題。

<!--truncate-->

### 超越觸控

桌面端與 VR 長期依賴滑鼠鍵盤輸入，而行動端主要採用觸控。但隨著觸控筆電普及，以及行動端對鍵盤/觸控筆支援需求增長，現有 React Native 觸控事件系統已無法滿足需求。

這導致非官方平台開發者必須分叉 React Native 或自訂原生元件，以實現懸停偵測、左鍵點擊等核心功能。此種分歧造成屬性冗餘——相似功能卻需不同平台的事件處理器，既增加框架複雜度，也阻礙跨平台程式碼共享。團隊因此決心提供跨平台指針 API。

React Native 致力於提供強健且具表現力的 API，在支援多平台的同時保持各平台特色體驗。設計此類 API 極具挑戰性，幸運的是網頁生態已有成熟的指針規範可供借鑑。

### 借鏡網頁標準

網頁平台同樣面臨跨平台擴展與未來兼容的挑戰。萬維網聯盟（W3C）制定的標準提案，正致力於建立跨平台、跨瀏覽器的互通性方案。

其中與我們需求最相關的，是 W3C 對抽象輸入形式「指針」的行為定義。[指針事件規範](https://www.w3.org/TR/pointerevents3/)基於滑鼠事件發展而來，旨在提供統一的跨設備輸入事件接口，同時保留設備特定處理的彈性。

遵循此規範將為 React Native 使用者帶來多重效益：除解決前述問題外，更能提升傳統單一輸入平台的交互能力（例如 Android 手機連接藍牙滑鼠，或 M2 版 iPad 的 Apple Pencil 懸停功能）。

規範兼容性也創造了網頁與 React Native 間的知識轉移機會。網頁指針事件的教學資源可同時服務 React Native 開發者。我們也認知到 React Native 需求與網頁不盡相同，將採取「盡力符合規範，明確標註差異」的策略。相關工作還包括[減少 API 碎片化](https://github.com/react-native-community/discussions-and-proposals/pull/496)的無障礙與效能標準對齊。

## 移植網頁平台測試

雖然指針事件規範提供了 API 接口與行為描述，但我們發現其不足以作為修改依據。然而，瀏覽器領域另有確保規範符合性的機制——[網頁平台測試](https://web-platform-tests.org/)！

這些測試基於瀏覽器的命令式 DOM API 編寫，與 React Native 的聲明式視圖原語不兼容。因此我們建立了類比測試框架，開發專屬 React Native 的測試 API 來移植這些網頁測試案例。

我們在 RNTester 中實作了全新手動測試框架（暫稱 RNTester 平台測試）。該框架允許將測試案例直接構建為 React 元件，透過 UI 渲染並回報結果，目前仍屬基礎階段。

![GIF 並排展示「Pointer Events hoverable pointer attributes test」在 React Native (iOS) 左側與 Web (原始實作) 右側的執行對比。](/blog/assets/pointer-events-wpt-demo.gif)

這些測試將持續協助我們完善 Pointer Events 的實作。未來這些測試也能擴展至 Android 和 iOS 以外的平台。隨著測試套件規模增長，我們計劃自動化執行這些測試，以更有效捕捉實作中的回歸問題。

## 運作原理

我們的 Pointer Events 實作大量基於現有觸控事件派發基礎架構。在 Android 和 iOS 上，我們分別利用了 MotionEvent 和 UITouch 事件。下圖展示事件派發的整體流程。

![圖解 Android 和 iOS UI 輸入事件轉換為 Pointer Events 的程式流程。Android 輸入處理器 "onTouchEvent" 和 "onHoverEvent" 觸發 "MotionEvents"，經解析為 Pointer Events 後透過 JSI 派發至 React 渲染器。iOS 路徑類似，輸入處理器 "touchesBegan"、"touchesMoved"、"touchesEnded" 和 "hovering" 將 "UITouch" 與 "UIEvent" 解析為 Pointer Events。](/blog/assets/pointer-events-code-flow.png)

以 Android 為例，運用平台事件的通用方法如下：

1. 遍歷 `MotionEvent` 的所有指針，透過深度優先搜索確定每個指針的目標 React 視圖及其祖先路徑。
2. 將 `MotionEvent` 類別映射至對應的 pointer events。`MotionEvent` 與 `PointerEvent` 存在一對多關係。圖示中的虛線表示若指向裝置不支援懸停時才會觸發的事件。

![圖解 Android MotionEvents 類型轉換為觸發 Pointer Events 的關係。部分 pointer events 在指向裝置不支援懸停時才會觸發。"ACTION_DOWN" 和 "ACTION_POINTER_DOWN" 觸發 pointerdown，條件性觸發 pointerenter、pointerover。"ACTION_MOVE" 和 "ACTION_HOVER_MOVE" 觸發 pointerover、pointermove、pointerout、pointerup。"ACTION_UP" 和 "ACTION_POINTER_UP" 觸發 pointerup，條件性觸發 pointerout、pointerleave。](/blog/assets/pointer-events-motionevent-relationship.png)

1. Build the `PointerEvent` interface with platform details from the `MotionEvent` and cached state of previous interactions. (Ex. [the `button` property](https://w3c.github.io/pointerevents/#the-button-property))
2. Dispatch the pointer events from Android to React Native’s [core event queue](https://github.com/facebook/react-native/blob/main/ReactCommon/react/renderer/core/EventQueueProcessor.cpp#L20) and leverage JSI to call the [`dispatchEvent`](https://github.com/facebook/react/blob/main/packages/react-native-renderer/src/ReactFabricEventEmitter.js#L83) method in `react-native-renderer` which iterates through the React tree for the bubble and capture phase of the event.

## 實作進度

目前我們專注於實作 Pointer Events 規範中最基礎的常用事件，涵蓋按壓、懸停與移動等交互行為。

### 事件

| Implemented    | Work in Progress | Yet to be Implemented |
| -------------- | ---------------- | --------------------- |
| onPointerOver  | onPointerCancel  | onClick               |
| onPointerEnter |                  | onContextMenu         |
| onPointerDown  |                  | onGotPointerCapture   |
| onPointerMove  |                  | onLostPointerCapture  |
| onPointerUp    |                  | onPointerRawUpdate    |
| onPointerOut   |                  |                       |
| onPointerLeave |                  |                       |

:::info

onPointerCancel 已與原生平台的「取消」事件掛鉤，但這不一定對應 Web 平台預期的觸發時機。

:::

### 事件屬性

上述事件均已實作大部分 PointerEvent 物件預期的屬性（在 React Native 中透過 `event.nativeEvent` 屬性暴露）。您可在[事件物件的 Flowtype 介面定義](https://github.com/facebook/react-native/blob/59ee57352738f030b41589a450209e51e44bbb06/Libraries/Types/CoreEventTypes.js#L175)查看完整屬性列表。需特別說明的是，`relatedTarget` 屬性因涉及以非正規方式暴露原生視圖參照，目前尚未完全實作。

## 未來工作與探索方向

除了上述事件外，還有一些與 Pointer Events 相關的 API。未來我們計劃將這些 API 納入實作範圍，包括：

- 指標捕獲 API (Pointer Capture API)
  - 包含元素參考上的命令式 API，如 `setPointerCapture()`、`releasePointerCapture()` 和 `hasPointerCapture()`。
- `touch-action` 樣式屬性
  - 網頁使用此 CSS 屬性來宣告式協調瀏覽器與網站自訂事件處理程式之間的互動手勢。在 React Native 中，可用於協調 View 的指標事件處理器與父 ScrollView 之間的事件處理。
- `click`、`contextmenu`、`auxclick`
  - `click` 是互動的抽象定義，可能透過無障礙功能模式或其他平台特性互動觸發。

原生 Pointer Events 實作的另一項優勢是能讓我們重新審視並改進目前僅限觸控事件、且由 JavaScript 中 Responder、Pressability 和 PanResponder API 處理的各種手勢操作方式。

Furthermore, we are continuing to explore including an implementation of the `EventTarget` interface for React Native host components (i.e. `add`/`removeEventListener`) which we believe will make possible more user-land abstractions for handling pointer interactions.

## 試用方式

我們的 Pointer Events 實作仍處於實驗階段，但希望獲得社群對現有成果的反饋。若您想試用此 API，需啟用以下功能旗標：

### 啟用功能旗標

:::note

Pointer Events 目前僅在[新架構 (Fabric)](https://reactnative.dev/docs/the-new-architecture/use-app-template)中實作，且僅支援 React Native 0.71+ 版本（本文撰寫時為候選發布版）。

:::

在您的 JavaScript 入口檔案（預設 React Native 應用模板中的 index.js）中，需啟用 `shouldEmitW3CPointerEvents` 旗標以使用 Pointer Events，並啟用 `shouldPressibilityUseW3CPointerEventsForHover` 以在 `Pressability` 中使用 Pointer Events。

```js
import ReactNativeFeatureFlags from 'react-native/Libraries/ReactNative/ReactNativeFeatureFlags';

// enable the JS-side of the w3c PointerEvent implementation
ReactNativeFeatureFlags.shouldEmitW3CPointerEvents = () => true;

// enable hover events in Pressibility to be backed by the PointerEvent implementation
ReactNativeFeatureFlags.shouldPressibilityUseW3CPointerEventsForHover =
  () => true;
```

### iOS 專屬設定

為確保指標事件能從 iOS 原生渲染器發送，您需在原生應用初始化程式碼（通常是 `AppDelegate.mm`）中切換一個原生功能旗標。

```objc
#import <React/RCTConstants.h>

// ...

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    RCTSetDispatchW3CPointerEvents(YES);

    // ...
}
```

Note that to ensure that the Pointer Event implementation can distinguish between mouse and touch pointers on iOS you need to add [`UIApplicationSupportsIndirectInputEvents`](https://developer.apple.com/documentation/bundleresources/information_property_list/uiapplicationsupportsindirectinputevents) to your Xcode project’s `info.plist`.

### Android 專屬設定

與 iOS 類似，Android 也需在應用初始化時啟用功能旗標——通常是根 React Activity 或 Surface 的 `onCreate` 方法中設定。

```java
import com.facebook.react.config.ReactFeatureFlags;

//... somewhere in initialization

@Override
public void onCreate() {
    ReactFeatureFlags.dispatchPointerEvents = true;
}
```

### JavaScript 設定

```js
function onPointerOver(event) {
  console.log(
    'Over blue box offset: ',
    event.nativeEvent.offsetX,
    event.nativeEvent.offsetY,
  );
}

// ... in some component
<View
  onPointerOver={onPointerOver}
  style={{height: 100, width: 100, backgroundColor: 'blue'}}
/>;
```

## 歡迎提供意見回饋

目前 Pointer Events 已應用於我們的 VR 平台並驅動 Oculus Store，但我們也期待獲得社群對現有實作方式與進度的早期反饋。我們樂於分享後續進展，若您對此工作有任何疑問或想法，歡迎加入[指標事件專屬討論](https://github.com/react-native-community/discussions-and-proposals/discussions/557)。