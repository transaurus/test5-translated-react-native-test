---
id: communication-ios
title: Communication between native and React Native
---

在[整合既有應用程式指南](integration-with-existing-apps)和[原生UI元件指南](native-components-ios)中，我們學習了如何將React Native嵌入原生元件，反之亦然。當我們混合使用原生與React Native元件時，最終會需要在這兩個世界之間建立溝通管道。其他指南已提及部分實現方法，本文將總結現有技術方案。

## 簡介

React Native的設計靈感來自React，因此資訊流的基本概念相似。React中的數據流是單向的——我們維護元件層級結構，每個元件僅依賴其父元件和自身內部狀態。這通過屬性(properties)實現：數據以自上而下的方式從父元件傳遞給子元件。若祖先元件需要依賴後代元件的狀態，則應向下傳遞回調函數供後代元件更新祖先狀態。

此概念同樣適用於React Native。只要我們完全在框架內構建應用程式，就可以通過屬性和回調驅動應用。但當混合使用React Native與原生元件時，就需要特定的跨語言機制來實現它們之間的資訊傳遞。

## 屬性

屬性是最直接的跨元件通訊方式。因此我們需要實現雙向傳遞：從原生到React Native，以及從React Native到原生。

### 從原生向React Native傳遞屬性

In order to embed a React Native view in a native component, we use `RCTRootView`. `RCTRootView` is a `UIView` that holds a React Native app. It also provides an interface between native side and the hosted app.

`RCTRootView`的初始化方法允許向下傳遞任意屬性，`initialProperties`參數必須是`NSDictionary`實例。該字典會在內部轉換為JSON對象，供頂層JS元件引用。

```objectivec
NSArray *imageList = @[@"https://dummyimage.com/600x400/ffffff/000000.png",
                       @"https://dummyimage.com/600x400/000000/ffffff.png"];

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
NSArray *imageList = @[@"https://dummyimage.com/600x400/ff0000/000000.png",
                       @"https://dummyimage.com/600x400/ffffff/ff0000.png"];

rootView.appProperties = @{@"images" : imageList};
```

隨時更新屬性都是可行的，但更新操作必須在主線程執行。讀取操作則可在任意線程進行。

:::note
Currently, there is a known issue where setting appProperties during the bridge startup, the change can be lost. See https://github.com/facebook/react-native/issues/20115 for more information.
:::

無法實現部分屬性更新，建議自行封裝此功能。

### 從React Native向原生傳遞屬性

The problem exposing properties of native components is covered in detail in [this article](native-components-ios#properties). In short, export properties with `RCT_CUSTOM_VIEW_PROPERTY` macro in your custom native component, then use them in React Native as if the component was an ordinary React Native component.

### 屬性的局限性

跨語言屬性的主要缺陷是不支持回調機制——這正是實現自下而上數據綁定所需的。例如當您需要通過JS操作移除原生父視圖中的RN子視圖時，純屬性方案無法實現這種自下而上的資訊傳遞。

雖然存在跨語言回調的變體方案([參見此處](native-modules-ios#callbacks))，但這些回調並非萬能解藥。核心問題在於它們不適合作為屬性傳遞，該機制實際實現的是：從JS觸發原生操作後，仍在JS端處理操作結果。

## 其他跨語言交互方式（事件與原生模組）

如前一章所述，使用屬性存在一些限制。有時屬性不足以驅動應用程式的邏輯，我們需要一個提供更大靈活性的解決方案。本章涵蓋 React Native 中可用的其他通訊技術，這些技術既可用於內部通訊（RN 中 JS 與原生層之間），也可用於外部通訊（RN 與應用程式的「純原生」部分之間）。

React Native 允許您執行跨語言函數呼叫。您可以從 JS 執行自訂原生程式碼，反之亦然。遺憾的是，根據我們所處的端，我們會以不同的方式實現相同的目標。對於原生端，我們使用事件機制來排程在 JS 中執行處理函數；而對於 React Native 端，我們直接呼叫由原生模組匯出的方法。

### 從原生端呼叫 React Native 函數（事件）

事件在[這篇文章](native-components-ios#events)中有詳細描述。請注意，使用事件無法保證執行時間，因為事件是在單獨的執行緒上處理的。

事件功能強大，因為它們允許我們在不需引用 React Native 元件的情況下對其進行修改。然而，使用時可能會遇到一些陷阱：

- 由於事件可以從任何地方發送，它們可能會在專案中引入義大利麵條式的依賴關係。
- 事件共享命名空間，這意味著您可能會遇到名稱衝突。這些衝突無法靜態檢測，因此難以除錯。
- 如果您使用多個相同 React Native 元件的實例，並希望從事件的角度區分它們，則可能需要引入標識符並與事件一起傳遞（您可以使用原生視圖的 `reactTag` 作為標識符）。

我們在將原生元件嵌入 React Native 時常用的模式是讓原生元件的 RCTViewManager 作為視圖的委託，透過橋接器將事件發送回 JavaScript。這將相關的事件呼叫集中在一個地方。

### 從 React Native 呼叫原生函數（原生模組）

原生模組是在 JS 中可用的 Objective-C 類別。通常每個 JS 橋接器會為每個模組創建一個實例。它們可以向 React Native 匯出任意函數和常數。[這篇文章](native-modules-ios#content)對此有詳細說明。

原生模組是單例的這一事實限制了在嵌入情境下的機制。假設我們有一個嵌入原生視圖的 React Native 元件，我們希望更新原生父視圖。使用原生模組機制，我們將匯出一個函數，該函數不僅接受預期的參數，還接受父原生視圖的標識符。該標識符將用於獲取父視圖的引用以進行更新。也就是說，我們需要在模組中維護從標識符到原生視圖的映射。

儘管這個解決方案複雜，但它被用於 `RCTUIManager` 中，這是 React Native 內部管理所有 React Native 視圖的類別。

原生模組還可用於向 JS 公開現有的原生函式庫。[Geolocation 函式庫](https://github.com/michalchudziak/react-native-geolocation)就是這個想法的一個實例。

:::caution
所有原生模組共享相同的命名空間。在創建新模組時請注意名稱衝突。
:::

## 佈局計算流程

在整合原生與 React Native 時，我們還需要一種方法來合併兩種不同的佈局系統。本節涵蓋常見的佈局問題，並提供解決這些問題的機制簡要說明。

### 嵌入 React Native 的原生元件佈局

This case is covered in [this article](native-components-ios#styles). To summarize, since all our native react views are subclasses of `UIView`, most style and size attributes will work like you would expect out of the box.

### 嵌入原生端的 React Native 元件佈局

#### 固定大小的 React Native 內容

一般情境是當我們有一個固定大小的 React Native 應用程式，且原生端知道其大小時。特別是，全螢幕的 React Native 視圖就屬於這種情況。如果我們想要一個較小的根視圖，可以明確設定 RCTRootView 的框架。

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

當我們有一個固定大小的根視圖時，需要在 JS 端尊重其邊界。換句話說，我們需要確保 React Native 內容能夠被包含在固定大小的根視圖內。最簡單的方法是使用 Flexbox 佈局。如果使用絕對定位，且 React 元件在根視圖邊界外可見，將會與原生視圖重疊，導致某些功能行為異常。例如，'TouchableHighlight' 將不會在根視圖邊界外的高亮觸摸區域生效。

動態更新根視圖的大小（通過重新設定其 frame 屬性）是完全可行的。React Native 會自動處理內容的佈局調整。

#### 彈性大小的 React Native 內容

某些情況下，我們需要渲染初始大小未知的內容。假設大小將在 JS 中動態定義，我們有兩種解決方案：

1. 可以將 React Native 視圖包裹在 `ScrollView` 元件中。這能確保內容始終可見且不會與原生視圖重疊。
2. React Native 允許在 JS 中計算 RN 應用的大小，並將結果提供給宿主 `RCTRootView` 的所有者。所有者需負責重新佈局子視圖以保持 UI 一致性。這通過 `RCTRootView` 的彈性模式實現。

`RCTRootView` 支援 4 種不同的尺寸彈性模式：

```objectivec title='RCTRootView.h'
typedef NS_ENUM(NSInteger, RCTRootViewSizeFlexibility) {
  RCTRootViewSizeFlexibilityNone = 0,
  RCTRootViewSizeFlexibilityWidth,
  RCTRootViewSizeFlexibilityHeight,
  RCTRootViewSizeFlexibilityWidthAndHeight,
};
```

`RCTRootViewSizeFlexibilityNone` is the default value, which makes a root view's size fixed (but it still can be updated with `setFrame:`). The other three modes allow us to track React Native content's size updates. For instance, setting mode to `RCTRootViewSizeFlexibilityHeight` will cause React Native to measure the content's height and pass that information back to `RCTRootView`'s delegate. An arbitrary action can be performed within the delegate, including setting the root view's frame, so the content fits. The delegate is called only when the size of the content has changed.

:::caution
Making a dimension flexible in both JS and native leads to undefined behavior. For example - don't make a top-level React component's width flexible (with `flexbox`) while you're using `RCTRootViewSizeFlexibilityWidth` on the hosting `RCTRootView`.
:::

以下是一個範例：

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

範例中的 `FlexibleSizeExampleView` 視圖包含一個根視圖。我們建立並初始化根視圖後設定其委派，委派將處理尺寸更新。接著將根視圖的尺寸彈性設為 `RCTRootViewSizeFlexibilityHeight`，這意味著每次 React Native 內容高度變化時，`rootViewDidChangeIntrinsicSize:` 方法會被呼叫。最後設定根視圖的寬度和位置（注意高度設定此時無效，因為高度已交由 RN 控制）。

完整範例程式碼可參閱[此處](https://github.com/facebook/react-native/blob/main/packages/rn-tester/RNTester/NativeExampleViews/FlexibleSizeExampleView.mm)。

動態變更根視圖的尺寸彈性模式是可行的。變更後會觸發佈局重新計算，並在內容尺寸確定後呼叫委派方法 `rootViewDidChangeIntrinsicSize:`。

:::note
React Native 的佈局計算在獨立線程執行，而原生 UI 更新則在主線程進行。
這可能導致原生與 React Native 間的暫時性 UI 不一致。此為已知問題，團隊正在努力同步不同來源的 UI 更新。
:::

:::note
React Native 在根視圖成為其他視圖的子視圖之前，不會執行任何佈局計算。
若您希望在知道尺寸前隱藏 React Native 視圖，請先將根視圖添加為子視圖並設定為初始隱藏（使用 `UIView` 的 `hidden` 屬性）。然後在委派方法中變更其可見性。
:::