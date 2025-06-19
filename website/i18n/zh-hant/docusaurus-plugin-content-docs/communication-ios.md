---
id: communication-ios
title: Communication between native and React Native
---

在[整合既有應用程式指南](integration-with-existing-apps)和[原生UI元件指南](legacy/native-components-ios)中，我們學習了如何將React Native嵌入原生元件，反之亦然。當我們混合使用原生與React Native元件時，最終會需要在這兩個世界之間建立溝通管道。其他指南中已提及部分實現方法，本文將總結現有技術方案。

## 簡介

React Native的設計靈感來自React，因此資訊流的基本概念相似。React中的資料流是單向的，我們維護著元件層級結構，其中每個元件僅依賴其父元件和自身內部狀態。這通過屬性(properties)實現：資料以自上而下的方式從父元件傳遞給子元件。若祖先元件需要依賴後代元件的狀態，則應向下傳遞回調函數供後代元件更新祖先狀態。

相同概念適用於React Native。只要我們完全在框架內建構應用程式，就可以透過屬性和回調驅動應用。但當混合使用React Native與原生元件時，我們需要特定的跨語言機制來實現它們之間的資訊傳遞。

## 屬性

屬性是跨元件通訊最直接的方式。因此我們需要實現雙向屬性傳遞：從原生到React Native，以及從React Native到原生。

### 從原生傳遞屬性到React Native

為了將React Native視圖嵌入原生元件，我們使用`RCTRootView`。`RCTRootView`是承載React Native應用的`UIView`，同時也提供原生端與宿主應用之間的介面。

`RCTRootView`的初始化方法允許您向下傳遞任意屬性到React Native應用。`initialProperties`參數必須是`NSDictionary`實例，該字典會在內部轉換為頂層JS元件可引用的JSON物件。

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

`RCTRootView`還提供可讀寫屬性`appProperties`。設置`appProperties`後，React Native應用會使用新屬性重新渲染。僅當新屬性與先前值不同時才會執行更新。

```objectivec
NSArray *imageList = @[@"https://dummyimage.com/600x400/ff0000/000000.png",
                       @"https://dummyimage.com/600x400/ffffff/ff0000.png"];

rootView.appProperties = @{@"images" : imageList};
```

隨時更新屬性是被允許的，但更新操作必須在主線程執行。而讀取操作可在任意線程進行。

:::note
Currently, there is a known issue where setting appProperties during the bridge startup, the change can be lost. See https://github.com/facebook/react-native/issues/20115 for more information.
:::

目前無法實現部分屬性更新。建議您自行建立封裝層來處理此需求。

### 從React Native傳遞屬性到原生

原生元件屬性的暴露問題在[這篇文章](legacy/native-components-ios#properties)中有詳細說明。簡而言之，在自訂原生元件中使用`RCT_CUSTOM_VIEW_PROPERTY`宏導出屬性，即可在React Native中像使用普通React Native元件一樣使用這些屬性。

### 屬性的限制

跨語言屬性的主要缺點是不支援回調機制，這使得我們無法實現自下而上的資料綁定。例如當您需要透過JS操作移除原生父視圖中的RN子視圖時，僅靠屬性無法實現，因為這需要資訊自下而上傳遞。

雖然我們有跨語言回調的變體方案（[參見此處](legacy/native-modules-ios#callbacks)），但這些回調並非總是適用。主要問題在於它們不適合作為屬性傳遞，該機制實際是讓我們從JS觸發原生操作，並在JS中處理操作結果。

## 其他跨語言互動方式（事件與原生模組）

如前一章所述，使用屬性存在一些限制。有時屬性不足以驅動應用程式的邏輯，我們需要一個提供更多靈活性的解決方案。本章涵蓋 React Native 中可用的其他通訊技術，這些技術可用於內部通訊（RN 中的 JS 與原生層之間）以及外部通訊（RN 與應用程式的「純原生」部分之間）。

React Native 允許您執行跨語言函數呼叫。您可以從 JS 執行自訂原生程式碼，反之亦然。不幸的是，根據我們所處的環境，我們會以不同的方式實現相同的目標。對於原生端，我們使用事件機制來安排在 JS 中執行處理函數，而對於 React Native，我們直接呼叫原生模組匯出的方法。

### 從原生端呼叫 React Native 函數（事件）

事件在[這篇文章](legacy/native-components-ios#events)中有詳細描述。請注意，使用事件無法保證執行時間，因為事件是在單獨的執行緒上處理的。

事件非常強大，因為它們允許我們在不需引用 React Native 元件的情況下對其進行更改。然而，在使用時可能會遇到一些陷阱：

- 由於事件可以從任何地方發送，它們可能會在專案中引入混亂的依賴關係。
- 事件共享命名空間，這意味著您可能會遇到名稱衝突。衝突不會被靜態檢測，因此難以調試。
- 如果您使用多個相同 React Native 元件的實例，並且希望從事件的角度區分它們，您可能需要引入標識符並與事件一起傳遞（可以使用原生視圖的 `reactTag` 作為標識符）。

我們在將原生元件嵌入 React Native 時常用的模式是讓原生元件的 RCTViewManager 作為視圖的委託，通過橋接將事件發送回 JavaScript。這將相關的事件呼叫集中在一個地方。

### 從 React Native 呼叫原生函數（原生模組）

原生模組是可在 JS 中使用的 Objective-C 類別。通常每個 JS 橋接會創建每個模組的一個實例。它們可以向 React Native 匯出任意函數和常數。[這篇文章](legacy/native-modules-ios#content)已詳細介紹了它們。

原生模組是單例的事實在嵌入情境中限制了此機制。假設我們有一個嵌入原生視圖的 React Native 元件，我們希望更新原生父視圖。使用原生模組機制，我們將匯出一個不僅接受預期參數，還接受父原生視圖標識符的函數。該標識符將用於檢索父視圖的引用以進行更新。也就是說，我們需要在模組中維護從標識符到原生視圖的映射。

儘管此解決方案複雜，但它被用於 `RCTUIManager` 中，這是管理所有 React Native 視圖的內部 React Native 類別。

原生模組還可用於向 JS 公開現有的原生函式庫。[Geolocation 函式庫](https://github.com/michalchudziak/react-native-geolocation)就是此概念的一個實際例子。

:::caution
所有原生模組共享相同的命名空間。在創建新模組時請注意名稱衝突。
:::

## 佈局計算流程

在整合原生與 React Native 時，我們還需要一種方法來協調兩種不同的佈局系統。本節涵蓋常見的佈局問題，並提供解決這些問題的機制簡要描述。

### 嵌入 React Native 的原生元件佈局

This case is covered in [this article](legacy/native-components-ios#styles). To summarize, since all our native react views are subclasses of `UIView`, most style and size attributes will work like you would expect out of the box.

### 嵌入原生端的 React Native 元件佈局

#### 固定大小的 React Native 內容

一般情境是當我們有一個固定大小的 React Native 應用程式，且原生端已知此大小。特別是，全螢幕的 React Native 視圖屬於此情況。如果我們想要一個較小的根視圖，可以明確設置 RCTRootView 的框架。

舉例來說，若要讓一個 RN 應用程式高度為 200（邏輯）像素，寬度與宿主視圖同寬，我們可以這樣做：

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

當我們有一個固定尺寸的根視圖時，需要在 JS 端尊重其邊界。換句話說，我們需要確保 React Native 內容能被包含在固定尺寸的根視圖內。最簡單的方法是使用 Flexbox 佈局。如果使用絕對定位，且 React 元件在根視圖邊界外可見，將會與原生視圖產生重疊，導致某些功能出現非預期行為。例如，'TouchableHighlight' 將不會在根視圖邊界外的高亮觸碰區域顯示效果。

動態更新根視圖尺寸（透過重新設定其 frame 屬性）是完全可行的，React Native 會自動處理內容的佈局調整。

#### 彈性尺寸的 React Native 內容

某些情況下，我們需要渲染初始尺寸未知的內容。假設尺寸將由 JS 動態決定，我們有兩種解決方案：

1. 可將 React Native 視圖包裹在 `ScrollView` 元件中，這能確保內容始終可存取且不會與原生視圖重疊。
2. React Native 允許在 JS 中計算 RN 應用的尺寸，並將結果提供給宿主 `RCTRootView` 的擁有者。擁有者需負責重新佈局子視圖以維持 UI 一致性，這是透過 `RCTRootView` 的彈性模式實現的。

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

以下為範例說明：

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

此範例中的 `FlexibleSizeExampleView` 視圖包含一個根視圖。我們建立並初始化根視圖後設定代理，代理將處理尺寸更新。接著將根視圖的尺寸彈性設為 `RCTRootViewSizeFlexibilityHeight`，這表示每當 React Native 內容高度變化時，`rootViewDidChangeIntrinsicSize:` 方法會被呼叫。最後設定根視圖的寬度與位置（注意高度設定在此無效，因為高度已交由 RN 控制）。

完整範例程式碼可參閱[此處](https://github.com/facebook/react-native/blob/main/packages/rn-tester/RNTester/NativeExampleViews/FlexibleSizeExampleView.mm)。

動態變更根視圖的尺寸彈性模式是可行的。變更後會觸發佈局重新計算，待內容尺寸確定後，代理的 `rootViewDidChangeIntrinsicSize:` 方法將被呼叫。

:::note
React Native 的佈局計算在獨立線程執行，而原生 UI 更新則在主線程進行。
這可能導致原生與 React Native 之間出現短暫的 UI 不一致，此為已知問題，團隊正致力於同步不同來源的 UI 更新。
:::

:::note
React Native 在根視圖成為其他視圖的子視圖之前，不會執行任何佈局計算。
若您希望在知道尺寸前隱藏 React Native 視圖，請將根視圖添加為子視圖並使其初始狀態為隱藏（使用 `UIView` 的 `hidden` 屬性）。然後在委派方法中變更其可見性。
:::