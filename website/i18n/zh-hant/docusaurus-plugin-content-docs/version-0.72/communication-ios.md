---
id: communication-ios
title: Communication between native and React Native
---

在[整合既有應用程式指南](integration-with-existing-apps)和[原生UI元件指南](native-components-ios)中，我們學習了如何將React Native嵌入原生元件，反之亦然。當我們混合使用原生與React Native元件時，最終會需要在這兩個世界之間建立溝通管道。其他指南已提及部分實現方法，本文將統整現有技術。

## 簡介

React Native is inspired by React, so the basic idea of the information flow is similar. The flow in React is one-directional. We maintain a hierarchy of components, in which each component depends only on its parent and its own internal state. We do this with properties: data is passed from a parent to its children in a top-down manner. If an ancestor component relies on the state of its descendant, one should pass down a callback to be used by the descendant to update the ancestor.

The same concept applies to React Native. As long as we are building our application purely within the framework, we can drive our app with properties and callbacks. But, when we mix React Native and native components, we need some specific, cross-language mechanisms that would allow us to pass information between them.

## Properties

屬性是最直接的跨元件通訊方式，因此我們需要實現雙向屬性傳遞：從原生到React Native，以及從React Native到原生。

### 從原生傳遞屬性至React Native

In order to embed a React Native view in a native component, we use `RCTRootView`. `RCTRootView` is a `UIView` that holds a React Native app. It also provides an interface between native side and the hosted app.

`RCTRootView` has an initializer that allows you to pass arbitrary properties down to the React Native app. The `initialProperties` parameter has to be an instance of `NSDictionary`. The dictionary is internally converted into a JSON object that the top-level JS component can reference.

```objectivec
NSArray *imageList = @[@"http://foo.com/bar1.png",
                       @"http://foo.com/bar2.png"];

NSDictionary *props = @{@"images" : imageList};

RCTRootView *rootView = [[RCTRootView alloc] initWithBridge:bridge
                                                 moduleName:@"ImageBrowserApp"
                                          initialProperties:props];
```

```tsx
import React from 'react';
import {View, Image} from 'react-native';

export default class ImageBrowserApp extends React.Component {
  renderImage(imgURI) {
    return <Image source={{uri: imgURI}} />;
  }
  render() {
    return <View>{this.props.images.map(this.renderImage)}</View>;
  }
}
```

`RCTRootView` also provides a read-write property `appProperties`. After `appProperties` is set, the React Native app is re-rendered with new properties. The update is only performed when the new updated properties differ from the previous ones.

```objectivec
NSArray *imageList = @[@"http://foo.com/bar3.png",
                       @"http://foo.com/bar4.png"];

rootView.appProperties = @{@"images" : imageList};
```

屬性可隨時更新，但更新操作必須在主線程執行（讀取操作則可在任意線程進行）。

:::note
Currently, there is a known issue where setting appProperties during the bridge startup, the change can be lost. See https://github.com/facebook/react-native/issues/20115 for more information.
:::

無法單獨更新部分屬性，建議自行建立封裝器(wrapper)實現此功能。

### 從React Native傳遞屬性至原生

The problem exposing properties of native components is covered in detail in [this article](native-components-ios#properties). In short, export properties with `RCT_CUSTOM_VIEW_PROPERTY` macro in your custom native component, then use them in React Native as if the component was an ordinary React Native component.

### 屬性的限制

The main drawback of cross-language properties is that they do not support callbacks, which would allow us to handle bottom-up data bindings. Imagine you have a small RN view that you want to be removed from the native parent view as a result of a JS action. There is no way to do that with props, as the information would need to go bottom-up.

Although we have a flavor of cross-language callbacks ([described here](native-modules-ios#callbacks)), these callbacks are not always the thing we need. The main problem is that they are not intended to be passed as properties. Rather, this mechanism allows us to trigger a native action from JS, and handle the result of that action in JS.

## 其他跨語言互動方式（事件與原生模組）

如前一章所述，使用屬性存在一些限制。有時屬性不足以驅動應用程式的邏輯，我們需要一個更具彈性的解決方案。本章涵蓋了 React Native 中可用的其他通訊技術，這些技術既可用於內部通訊（RN 中的 JS 與原生層之間），也可用於外部通訊（RN 與應用程式的「純原生」部分之間）。

React Native 允許您執行跨語言函數呼叫。您可以從 JS 執行自訂原生程式碼，反之亦然。遺憾的是，根據我們所處的端，我們以不同的方式實現相同的目標。對於原生端，我們使用事件機制來安排在 JS 中執行處理函數，而對於 React Native，我們直接呼叫原生模組匯出的方法。

### 從原生端呼叫 React Native 函數（事件）

事件在[這篇文章](native-components-ios#events)中有詳細描述。請注意，使用事件無法保證執行時間，因為事件是在單獨的執行緒上處理的。

事件功能強大，因為它們允許我們在不需引用 React Native 元件的情況下對其進行修改。然而，使用時可能會遇到一些陷阱：

- 由於事件可以從任何地方發送，它們可能會在專案中引入義大利麵條式的依賴關係。
- 事件共享命名空間，這意味著您可能會遇到名稱衝突。衝突不會被靜態檢測，因此難以調試。
- 如果您使用多個相同 React Native 元件的實例，並希望從事件的角度區分它們，您可能需要引入標識符並與事件一起傳遞（可以使用原生視圖的 `reactTag` 作為標識符）。

我們在將原生元件嵌入 React Native 時常用的模式是讓原生元件的 RCTViewManager 作為視圖的委託，透過橋接將事件發送回 JavaScript。這將相關的事件呼叫集中在一個地方。

### 從 React Native 呼叫原生函數（原生模組）

原生模組是在 JS 中可用的 Objective-C 類別。通常每個 JS 橋接會創建每個模組的一個實例。它們可以向 React Native 匯出任意函數和常數。[這篇文章](native-modules-ios#content)對此有詳細說明。

原生模組是單例的這一事實限制了在嵌入情境下的機制。假設我們有一個嵌入原生視圖中的 React Native 元件，我們希望更新原生的父視圖。使用原生模組機制，我們將匯出一個函數，該函數不僅接受預期的參數，還接受父原生視圖的標識符。該標識符將用於獲取父視圖的引用以進行更新。也就是說，我們需要在模組中維護從標識符到原生視圖的映射。

儘管這個解決方案很複雜，但它被用於 `RCTUIManager` 中，這是 React Native 內部管理所有 React Native 視圖的類別。

原生模組還可用於向 JS 公開現有的原生函式庫。[Geolocation 函式庫](https://github.com/michalchudziak/react-native-geolocation)就是這個想法的一個實例。

:::caution
所有原生模組共享相同的命名空間。在創建新模組時請注意名稱衝突。
:::

## 佈局計算流程

在整合原生與 React Native 時，我們還需要一種方法來統一兩種不同的佈局系統。本節涵蓋常見的佈局問題，並提供解決這些問題的機制簡要說明。

### 嵌入 React Native 中的原生元件佈局

This case is covered in [this article](native-components-ios#styles). To summarize, since all our native react views are subclasses of `UIView`, most style and size attributes will work like you would expect out of the box.

### 嵌入原生中的 React Native 元件佈局

#### 固定大小的 React Native 內容

一般情境是當我們有一個固定大小的 React Native 應用程式，且原生端已知其大小。特別是，全螢幕的 React Native 視圖屬於這種情況。如果我們想要一個較小的根視圖，可以明確設置 RCTRootView 的框架。

舉例來說，若要讓一個 RN 應用程式的高度為 200（邏輯）像素，並與宿主視圖的寬度相同，我們可以這樣做：

```objectivec title='SomeViewController.m'
- (void)viewDidLoad
{
  [...]
  RCTRootView *rootView = [[RCTRootView alloc] initWithBridge:bridge
                                                   moduleName:appName
                                            initialProperties:props];
  rootView.frame = CGRectMake(0, 0, self.view.width, 200);
  [self.view addSubview:rootView];
}
```

當我們有一個固定大小的根視圖時，我們需要在 JS 端尊重其邊界。換句話說，我們需要確保 React Native 內容能夠被包含在固定大小的根視圖內。最簡單的方法是使用 Flexbox 佈局。如果你使用絕對定位，且 React 元件在根視圖邊界外可見，將會與原生視圖重疊，導致某些功能行為異常。例如，'TouchableHighlight' 將不會在根視圖邊界外的高亮觸摸區域顯示效果。

動態更新根視圖的大小是完全可行的，只需重新設定其 frame 屬性。React Native 會自動處理內容的佈局。

#### 彈性大小的 React Native 內容

在某些情況下，我們希望渲染初始大小未知的內容。假設大小將在 JS 中動態定義。我們有兩種解決方案：

1. 你可以將 React Native 視圖包裹在 `ScrollView` 元件中。這保證了內容始終可見且不會與原生視圖重疊。
2. React Native 允許你在 JS 中確定 RN 應用的大小，並將其提供給宿主 `RCTRootView` 的所有者。所有者負責重新佈局子視圖並保持 UI 一致性。我們通過 `RCTRootView` 的彈性模式來實現這一點。

`RCTRootView` 支援四種不同的尺寸彈性模式：

```objectivec title='RCTRootView.h'
typedef NS_ENUM(NSInteger, RCTRootViewSizeFlexibility) {
  RCTRootViewSizeFlexibilityNone = 0,
  RCTRootViewSizeFlexibilityWidth,
  RCTRootViewSizeFlexibilityHeight,
  RCTRootViewSizeFlexibilityWidthAndHeight,
};
```

`RCTRootViewSizeFlexibilityNone` 是預設值，使根視圖的大小固定（但仍可通過 `setFrame:` 更新）。其他三種模式允許我們追蹤 React Native 內容的尺寸更新。例如，將模式設為 `RCTRootViewSizeFlexibilityHeight` 會讓 React Native 測量內容高度並將該資訊傳回給 `RCTRootView` 的委託。委託中可以執行任意操作，包括設定根視圖的 frame 以使內容適配。僅當內容大小變更時才會呼叫委託。

:::caution
Making a dimension flexible in both JS and native leads to undefined behavior. For example - don't make a top-level React component's width flexible (with `flexbox`) while you're using `RCTRootViewSizeFlexibilityWidth` on the hosting `RCTRootView`.
:::

讓我們看一個例子。

```objectivec title='FlexibleSizeExampleView.m'
- (instancetype)initWithFrame:(CGRect)frame
{
  [...]

  _rootView = [[RCTRootView alloc] initWithBridge:bridge
  moduleName:@"FlexibilityExampleApp"
  initialProperties:@{}];

  _rootView.delegate = self;
  _rootView.sizeFlexibility = RCTRootViewSizeFlexibilityHeight;
  _rootView.frame = CGRectMake(0, 0, self.frame.size.width, 0);
}

#pragma mark - RCTRootViewDelegate
- (void)rootViewDidChangeIntrinsicSize:(RCTRootView *)rootView
{
  CGRect newFrame = rootView.frame;
  newFrame.size = rootView.intrinsicContentSize;

  rootView.frame = newFrame;
}
```

在此範例中，我們有一個 `FlexibleSizeExampleView` 視圖，其中包含一個根視圖。我們創建根視圖、初始化它並設定委託。委託將處理尺寸更新。接著，我們將根視圖的尺寸彈性設為 `RCTRootViewSizeFlexibilityHeight`，這意味著每次 React Native 內容的高度變更時，都會呼叫 `rootViewDidChangeIntrinsicSize:` 方法。最後，我們設定根視圖的寬度和位置。注意，我們也設定了高度，但由於高度依賴 RN，此設定不會生效。

你可以在此查看完整的範例原始碼[這裡](https://github.com/facebook/react-native/blob/main/packages/rn-tester/RNTester/NativeExampleViews/FlexibleSizeExampleView.mm)。

動態變更根視圖的尺寸彈性模式是可行的。變更根視圖的彈性模式會觸發佈局重新計算，一旦內容大小確定，委託的 `rootViewDidChangeIntrinsicSize:` 方法將被呼叫。

:::note
React Native 的佈局計算在單獨的線程中執行，而原生 UI 視圖更新則在主線程中進行。
這可能導致原生與 React Native 之間的暫時性 UI 不一致。這是一個已知問題，我們的團隊正在努力同步來自不同來源的 UI 更新。
:::

:::note
React Native 在根視圖成為其他視圖的子視圖之前，不會執行任何佈局計算。
如果您希望在知道尺寸前隱藏 React Native 視圖，請先將根視圖添加為子視圖並將其設為初始隱藏（使用 `UIView` 的 `hidden` 屬性）。然後在委派方法中更改其可見性。
:::