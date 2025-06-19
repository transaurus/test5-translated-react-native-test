---
id: communication-ios
title: Communication between native and React Native
---

在[與現有應用整合指南](integration-with-existing-apps)和[原生UI元件指南](native-components-ios)中，我們學習了如何將React Native嵌入原生元件，反之亦然。當我們混合使用原生和React Native元件時，最終會需要在這兩個世界之間進行通信。其他指南中已經提到了一些實現方法。本文總結了可用的技術。

## 簡介

React Native的靈感來自React，因此信息流的基本概念是相似的。React中的流動是單向的。我們維護一個元件層次結構，其中每個元件僅依賴於其父元件和自身的內部狀態。我們通過屬性來實現這一點：數據以自上而下的方式從父元件傳遞給其子元件。如果祖先元件依賴於其後代的狀態，則應向下傳遞一個回調函數，供後代用來更新祖先。

相同的概念適用於React Native。只要我們純粹在框架內構建應用程序，就可以通過屬性和回調來驅動我們的應用。但是，當我們混合使用React Native和原生元件時，需要一些特定的跨語言機制來在它們之間傳遞信息。

## 屬性

屬性是跨元件通信最直接的方式。因此，我們需要一種方法來將屬性從原生傳遞到React Native，以及從React Native傳遞到原生。

### 從原生傳遞屬性到React Native

為了將React Native視圖嵌入原生元件，我們使用`RCTRootView`。`RCTRootView`是一個包含React Native應用的`UIView`。它還提供了原生端與託管應用之間的接口。

`RCTRootView`有一個初始化器，允許你將任意屬性傳遞給React Native應用。`initialProperties`參數必須是`NSDictionary`的實例。字典在內部被轉換為頂層JS元件可以引用的JSON對象。

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

`RCTRootView`還提供了一個讀寫屬性`appProperties`。設置`appProperties`後，React Native應用會使用新屬性重新渲染。僅當新更新的屬性與之前的屬性不同時，才會執行更新。

```objectivec
NSArray *imageList = @[@"https://dummyimage.com/600x400/ff0000/000000.png",
                       @"https://dummyimage.com/600x400/ffffff/ff0000.png"];

rootView.appProperties = @{@"images" : imageList};
```

隨時更新屬性是可以的。但是，更新必須在主線程上執行。你可以在任何線程上使用getter。

:::note
Currently, there is a known issue where setting appProperties during the bridge startup, the change can be lost. See https://github.com/facebook/react-native/issues/20115 for more information.
:::

無法一次只更新部分屬性。我們建議你將其構建到自己的包裝器中。

### 從React Native傳遞屬性到原生

原生元件屬性的暴露問題在[這篇文章](native-components-ios#properties)中有詳細介紹。簡而言之，在你的自定義原生元件中使用`RCT_CUSTOM_VIEW_PROPERTY`宏導出屬性，然後在React Native中使用它們，就像該元件是普通的React Native元件一樣。

### 屬性的限制

跨語言屬性的主要缺點是它們不支持回調，而回調可以讓我們處理自下而上的數據綁定。想像一下，你有一個小的RN視圖，你想作為JS操作的結果從原生父視圖中移除。使用屬性無法做到這一點，因為信息需要自下而上傳遞。

儘管我們有一種跨語言回調的變體（[在此描述](native-modules-ios#callbacks)），但這些回調並不總是我們需要的。主要問題是它們不打算作為屬性傳遞。相反，這種機制允許我們從JS觸發原生操作，並在JS中處理該操作的結果。

## 其他跨語言交互方式（事件和原生模塊）

如前一章所述，使用屬性存在一些限制。有時屬性不足以驅動應用程式的邏輯，我們需要一個提供更大靈活性的解決方案。本章涵蓋 React Native 中可用的其他通訊技術，這些技術可用於內部通訊（RN 中 JS 與原生層之間的溝通），也可用於外部通訊（RN 與應用程式中「純原生」部分之間的溝通）。

React Native 允許您執行跨語言函數呼叫。您可以從 JS 呼叫自訂原生程式碼，反之亦然。遺憾的是，根據我們所處的端別，實現相同目標的方式有所不同。對於原生端，我們使用事件機制來安排在 JS 中執行處理函數；而對於 React Native 端，我們直接呼叫原生模組匯出的方法。

### 從原生端呼叫 React Native 函數（事件）

事件在[這篇文章](native-components-ios#events)中有詳細說明。請注意，使用事件無法保證執行時間，因為事件是在單獨的執行緒上處理的。

事件功能強大，因為它們允許我們變更 React Native 元件，而無需持有其參考。然而，使用時可能會遇到一些陷阱：

- 由於事件可以從任何地方發送，它們可能會在專案中引入義大利麵條式的依賴關係。
- 事件共享命名空間，這意味著您可能會遇到名稱衝突。衝突不會被靜態檢測，因此難以除錯。
- 如果您使用多個相同 React Native 元件的實例，並希望從事件的角度區分它們，您可能需要引入識別符並隨事件傳遞（可以使用原生視圖的 `reactTag` 作為識別符）。

我們在將原生元件嵌入 React Native 時常用的模式是讓原生元件的 RCTViewManager 作為視圖的委託，透過橋接器將事件發送回 JavaScript。這將相關的事件呼叫集中在一個地方。

### 從 React Native 呼叫原生函數（原生模組）

原生模組是可在 JS 中使用的 Objective-C 類別。通常每個模組在每個 JS 橋接器中會創建一個實例。它們可以向 React Native 匯出任意函數和常數。[這篇文章](native-modules-ios#content)對此有詳細說明。

原生模組是單例的事實限制了在嵌入情境下的機制。假設我們有一個嵌入原生視圖中的 React Native 元件，我們希望更新原生父視圖。使用原生模組機制，我們需要匯出一個不僅接受預期參數，還接受父原生視圖識別符的函數。該識別符將用於獲取父視圖的參考以進行更新。也就是說，我們需要在模組中維護一個從識別符到原生視圖的映射。

儘管這個解決方案複雜，但它被用於 `RCTUIManager` 中，這是 React Native 內部管理所有 React Native 視圖的類別。

原生模組也可用於向 JS 公開現有的原生函式庫。[Geolocation 函式庫](https://github.com/michalchudziak/react-native-geolocation)就是這個概念的實際範例。

:::caution
所有原生模組共享相同的命名空間。創建新模組時請注意名稱衝突。
:::

## 佈局計算流程

在整合原生與 React Native 時，我們還需要一種方法來協調兩種不同的佈局系統。本節涵蓋常見的佈局問題，並提供解決這些問題的機制簡介。

### 嵌入 React Native 中的原生元件佈局

This case is covered in [this article](native-components-ios#styles). To summarize, since all our native react views are subclasses of `UIView`, most style and size attributes will work like you would expect out of the box.

### 嵌入原生端中的 React Native 元件佈局

#### 固定尺寸的 React Native 內容

一般情境是當我們有一個固定尺寸的 React Native 應用程式，且原生端已知其尺寸時。特別是，全螢幕的 React Native 視圖就屬於這種情況。如果我們想要一個較小的根視圖，可以明確設定 RCTRootView 的框架。

舉例來說，若要讓一個 RN 應用程式高度為 200（邏輯）像素，並與宿主視圖同寬，我們可以這樣做：

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

當我們有一個固定尺寸的根視圖時，需要在 JS 端尊重其邊界。換句話說，我們需要確保 React Native 內容能被包含在固定尺寸的根視圖內。最簡單的方法是使用 Flexbox 佈局。如果使用絕對定位，且 React 元件在根視圖邊界外可見，將會與原生視圖產生重疊，導致某些功能出現非預期行為。例如，'TouchableHighlight' 將不會在根視圖邊界外高亮觸控區域。

動態調整根視圖尺寸（透過重新設定其 frame 屬性）是完全可行的。React Native 會自動處理內容的佈局。

#### 彈性尺寸的 React Native 內容

某些情況下，我們需要渲染初始尺寸未知的內容。假設尺寸將由 JS 動態決定，我們有兩種解決方案：

1. 可用 `ScrollView` 元件包裹 React Native 視圖，確保內容始終可存取且不會與原生視圖重疊。
2. React Native 允許在 JS 端計算 RN 應用尺寸並傳遞給宿主 `RCTRootView` 的擁有者。擁有者需負責重新佈局子視圖以維持 UI 一致性，這是透過 `RCTRootView` 的彈性模式實現的。

`RCTRootView` 支援 4 種尺寸彈性模式：

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

以下為範例：

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

此範例中，`FlexibleSizeExampleView` 視圖持有根視圖。我們建立並初始化根視圖後設定委派來處理尺寸更新。將根視圖的尺寸彈性設為 `RCTRootViewSizeFlexibilityHeight`，意味著每次 React Native 內容高度變化時，`rootViewDidChangeIntrinsicSize:` 方法會被呼叫。最後設定根視圖的寬度與位置（注意高度設定在此無效，因為高度已交由 RN 決定）。

完整範例程式碼可參閱[此處](https://github.com/facebook/react-native/blob/main/packages/rn-tester/RNTester/NativeExampleViews/FlexibleSizeExampleView.mm)。

動態變更根視圖的尺寸彈性模式是可行的。變更後會觸發佈局重計算，並在內容尺寸確定後呼叫委派的 `rootViewDidChangeIntrinsicSize:` 方法。

:::note
React Native 的佈局計算在獨立線程執行，而原生 UI 更新則在主線程進行。這可能導致原生與 React Native 間暫時的 UI 不一致性。此為已知問題，團隊正致力於同步不同來源的 UI 更新。
:::

:::note
React Native 在根視圖成為其他視圖的子視圖之前，不會執行任何佈局計算。
如果您想在知道尺寸前隱藏 React Native 視圖，請將根視圖添加為子視圖並使其初始狀態為隱藏（使用 `UIView` 的 `hidden` 屬性）。然後在委派方法中更改其可見性。
:::