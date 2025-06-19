---
id: communication-ios
title: Communication between native and React Native
---

在[與現有應用整合指南](integration-with-existing-apps)和[原生UI元件指南](native-components-ios)中，我們學習了如何在原生元件中嵌入React Native，反之亦然。當我們混合使用原生和React Native元件時，最終會需要在這兩個世界之間進行通信。其他指南中已經提到了一些實現方法。本文總結了可用的技術。

## 簡介

React Native的靈感來自React，因此信息流的基本概念是相似的。React中的流動是單向的。我們維護一個元件層次結構，其中每個元件僅依賴於其父元件和自身的內部狀態。我們通過屬性來實現這一點：數據以自上而下的方式從父元件傳遞給其子元件。如果祖先元件依賴於其後代的狀態，則應向下傳遞一個回調函數，供後代用來更新祖先。

相同的概念適用於React Native。只要我們純粹在框架內構建應用程序，就可以通過屬性和回調來驅動我們的應用。但是，當我們混合使用React Native和原生元件時，需要一些特定的跨語言機制來在它們之間傳遞信息。

## 屬性

屬性是跨元件通信最直接的方式。因此，我們需要一種方法來將屬性從原生傳遞到React Native，以及從React Native傳遞到原生。

### 從原生傳遞屬性到React Native

為了在原生元件中嵌入React Native視圖，我們使用`RCTRootView`。`RCTRootView`是一個包含React Native應用的`UIView`。它還提供了原生端與託管應用之間的接口。

`RCTRootView`有一個初始化器，允許你將任意屬性傳遞給React Native應用。`initialProperties`參數必須是`NSDictionary`的實例。該字典在內部被轉換為頂層JS元件可以引用的JSON對象。

```objectivec
NSArray *imageList = @[@"http://foo.com/bar1.png",
                       @"http://foo.com/bar2.png"];

NSDictionary *props = @{@"images" : imageList};

RCTRootView *rootView = [[RCTRootView alloc] initWithBridge:bridge
                                                 moduleName:@"ImageBrowserApp"
                                          initialProperties:props];
```

```jsx
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

`RCTRootView`還提供了一個可讀寫的屬性`appProperties`。設置`appProperties`後，React Native應用會使用新屬性重新渲染。僅當新更新的屬性與之前的屬性不同時才會執行更新。

```objectivec
NSArray *imageList = @[@"http://foo.com/bar3.png",
                       @"http://foo.com/bar4.png"];

rootView.appProperties = @{@"images" : imageList};
```

隨時更新屬性是可以的。但是，更新必須在主線程上執行。你可以在任何線程上使用getter。

:::note
Currently, there is a known issue where setting appProperties during the bridge startup, the change can be lost. See https://github.com/facebook/react-native/issues/20115 for more information.
:::

無法一次只更新部分屬性。我們建議你將其構建到自己的包裝器中。

### 從React Native傳遞屬性到原生

原生元件屬性的暴露問題在[這篇文章](native-components-ios#properties)中有詳細介紹。簡而言之，在你的自定義原生元件中使用`RCT_CUSTOM_VIEW_PROPERTY`宏導出屬性，然後在React Native中使用它們，就像該元件是一個普通的React Native元件一樣。

### 屬性的限制

跨語言屬性的主要缺點是它們不支持回調，而回調可以讓我們處理自下而上的數據綁定。想像一下，你有一個小的RN視圖，你希望由於JS操作而將其從原生父視圖中移除。使用屬性無法實現這一點，因為信息需要自下而上傳遞。

儘管我們有一種跨語言回調的變體（[在此描述](native-modules-ios#callbacks)），但這些回調並不總是我們需要的。主要問題是它們不打算作為屬性傳遞。相反，這種機制允許我們從JS觸發原生操作，並在JS中處理該操作的結果。

## 其他跨語言交互方式（事件和原生模塊）

如前一章所述，使用屬性存在一些限制。有時屬性不足以驅動應用程式的邏輯，我們需要一個更靈活的解決方案。本章涵蓋 React Native 中可用的其他通訊技術，這些技術可用於內部通訊（RN 中 JS 與原生層之間的溝通）以及外部通訊（RN 與應用程式中「純原生」部分之間的溝通）。

React Native 允許您執行跨語言函數呼叫。您可以從 JS 執行自訂原生程式碼，反之亦然。遺憾的是，根據我們所處的端點，我們會以不同的方式實現相同的目標。對於原生端，我們使用事件機制來排程在 JS 中執行處理函數；而對於 React Native 端，我們直接呼叫由原生模組匯出的方法。

### 從原生端呼叫 React Native 函數（事件）

事件在[這篇文章](native-components-ios#events)中有詳細說明。請注意，使用事件無法保證執行時間，因為事件是在單獨的執行緒上處理的。

事件非常強大，因為它們允許我們在不需要引用 React Native 元件的情況下對其進行修改。然而，使用時可能會遇到一些陷阱：

- 由於事件可以從任何地方發送，它們可能會在專案中引入義大利麵條式的依賴關係。
- 事件共享命名空間，這意味著您可能會遇到名稱衝突。這些衝突無法靜態檢測，因此難以除錯。
- 如果您使用多個相同 React Native 元件的實例，並且希望從事件的角度區分它們，您可能需要引入標識符並與事件一起傳遞（可以使用原生視圖的 `reactTag` 作為標識符）。

我們在將原生元件嵌入 React Native 時常用的模式是讓原生元件的 RCTViewManager 成為視圖的委託，透過橋接器將事件發送回 JavaScript。這將相關的事件呼叫集中在一個地方。

### 從 React Native 呼叫原生函數（原生模組）

原生模組是在 JS 中可用的 Objective-C 類別。通常每個模組在每個 JS 橋接器中會創建一個實例。它們可以向 React Native 匯出任意函數和常數。這些已在[這篇文章](native-modules-ios#content)中詳細介紹。

原生模組是單例的這一事實限制了在嵌入情境下的機制。假設我們有一個嵌入原生視圖中的 React Native 元件，並且我們希望更新原生父視圖。使用原生模組機制，我們需要匯出一個不僅接受預期參數，還接受父原生視圖標識符的函數。該標識符將用於獲取對父視圖的引用以進行更新。也就是說，我們需要在模組中維護一個從標識符到原生視圖的映射。

儘管這個解決方案很複雜，但它被用於 `RCTUIManager` 中，這是一個內部 React Native 類別，用於管理所有 React Native 視圖。

原生模組還可以用於向 JS 公開現有的原生函式庫。[Geolocation 函式庫](https://github.com/michalchudziak/react-native-geolocation)就是這一理念的實際範例。

:::caution
所有原生模組共享相同的命名空間。在創建新模組時請注意名稱衝突。
:::

## 佈局計算流程

在整合原生與 React Native 時，我們還需要一種方法來統合兩種不同的佈局系統。本節涵蓋常見的佈局問題，並提供解決這些問題的機制簡要說明。

### 嵌入 React Native 中的原生元件佈局

這種情況在[這篇文章](native-components-ios#styles)中有詳細說明。總結來說，由於我們所有的原生 React 視圖都是 `UIView` 的子類別，大多數樣式和尺寸屬性都會如預期般工作。

### 嵌入原生端中的 React Native 元件佈局

#### 固定尺寸的 React Native 內容

一般情境是當我們有一個固定尺寸的 React Native 應用程式，且該尺寸對原生端已知。特別是全螢幕的 React Native 視圖就屬於這種情況。如果我們想要一個較小的根視圖，可以明確設定 RCTRootView 的框架。

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

當我們有一個固定大小的根視圖時，我們需要在 JS 端尊重其邊界。換句話說，我們需要確保 React Native 內容能夠被包含在固定大小的根視圖內。最簡單的方法是使用 Flexbox 佈局。如果你使用絕對定位，且 React 元件在根視圖邊界外可見，將會與原生視圖產生重疊，導致某些功能行為異常。例如，'TouchableHighlight' 將不會在根視圖邊界外高亮你的觸控操作。

動態更新根視圖的大小是完全可行的，只需重新設定其 frame 屬性。React Native 會負責內容的佈局調整。

#### 彈性大小的 React Native 內容

在某些情況下，我們希望渲染初始大小未知的內容。假設大小將在 JS 中動態定義。我們有兩種解決方案：

1. 你可以將 React Native 視圖包裹在 `ScrollView` 元件中。這能確保內容始終可見且不會與原生視圖重疊。
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

`RCTRootViewSizeFlexibilityNone` 是預設值，使根視圖的大小固定（但仍可通過 `setFrame:` 更新）。其他三種模式允許我們追蹤 React Native 內容的尺寸更新。例如，將模式設為 `RCTRootViewSizeFlexibilityHeight` 會讓 React Native 測量內容高度並將資訊回傳給 `RCTRootView` 的委派。委派中可以執行任意操作，包括設定根視圖的 frame 以使內容適配。僅當內容尺寸變化時才會呼叫委派。

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

在此範例中，我們有一個 `FlexibleSizeExampleView` 視圖，其中包含一個根視圖。我們創建根視圖、初始化並設定委派。委派將處理尺寸更新。接著，我們將根視圖的尺寸彈性設為 `RCTRootViewSizeFlexibilityHeight`，這意味著每次 React Native 內容高度變化時，都會呼叫 `rootViewDidChangeIntrinsicSize:` 方法。最後，我們設定根視圖的寬度和位置。注意我們也設定了高度，但由於高度依賴 RN，因此該設定無效。

你可以在此查看完整的範例原始碼[這裡](https://github.com/facebook/react-native/blob/0.70-stable/packages/rn-tester/RNTester/NativeExampleViews/FlexibleSizeExampleView.m)。

動態變更根視圖的尺寸彈性模式是可行的。變更彈性模式會觸發佈局重新計算，一旦內容尺寸確定，委派的 `rootViewDidChangeIntrinsicSize:` 方法將被呼叫。

:::note
React Native 的佈局計算在獨立線程執行，而原生 UI 視圖更新則在主線程進行。
這可能導致原生與 React Native 之間暫時的 UI 不一致。這是已知問題，我們的團隊正在努力同步來自不同來源的 UI 更新。
:::

:::note
React Native 在根視圖成為其他視圖的子視圖之前，不會執行任何佈局計算。
如果您希望在知道尺寸前隱藏 React Native 視圖，請先將根視圖添加為子視圖並將其設為初始隱藏狀態（使用 `UIView` 的 `hidden` 屬性）。然後在委派方法中更改其可見性。
:::