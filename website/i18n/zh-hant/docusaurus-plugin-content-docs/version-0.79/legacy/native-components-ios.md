---
id: native-components-ios
title: iOS Native UI Components
---

import NativeDeprecated from '@site/versioned_docs/version-0.79/the-new-architecture/\_markdown_native_deprecation.mdx'

<NativeDeprecated />

現有大量原生UI元件可供最新應用程式使用——有些是平台內建元件，有些來自第三方函式庫，還有一些可能是您過去開發的專屬元件。React Native已封裝了部分核心平台元件（如`ScrollView`和`TextInput`），但並未涵蓋所有元件，特別是您先前為其他應用開發的自訂元件。幸運的是，我們可以將這些現有元件封裝起來，無縫整合到React Native應用中。

與原生模組指南類似，本篇屬於進階指南，假設您已具備iOS開發基礎知識。我們將透過實作核心React Native函式庫中現有`MapView`元件的部分功能，示範如何建構原生UI元件。

## iOS MapView範例

假設我們需要在應用中加入互動式地圖——與其從頭開發，不如直接使用[`MKMapView`](https://developer.apple.com/library/prerelease/mac/documentation/MapKit/Reference/MKMapView_Class/index.html)，只需讓它能透過JavaScript呼叫即可。

原生視圖由`RCTViewManager`的子類別創建和管理。這些子類別功能類似視圖控制器，但實際上是單例模式——橋接器只會為每個管理器創建一個實例。它們將原生視圖暴露給`RCTUIManager`，由後者委派設置和更新視圖屬性。`RCTViewManager`通常也作為視圖的代理，透過橋接器將事件傳回JavaScript。

要暴露視圖，您可以：

- 繼承`RCTViewManager`來創建元件管理器
- 添加`RCT_EXPORT_MODULE()`標記宏
- 實作`-(UIView *)view`方法

```objectivec title='RNTMapManager.m'
#import <MapKit/MapKit.h>

#import <React/RCTViewManager.h>

@interface RNTMapManager : RCTViewManager
@end

@implementation RNTMapManager

RCT_EXPORT_MODULE(RNTMap)

- (UIView *)view
{
  return [[MKMapView alloc] init];
}

@end
```

:::note
Do not attempt to set the `frame` or `backgroundColor` properties on the `UIView` instance that you expose through the `-view` method.
React Native will overwrite the values set by your custom class in order to match your JavaScript component's layout props.
If you need this granularity of control it might be better to wrap the `UIView` instance you want to style in another `UIView` and return the wrapper `UIView` instead.
See [Issue 2948](https://github.com/facebook/react-native/issues/2948) for more context.
:::

:::info
上例中，我們使用`RNT`作為類別名稱前綴。前綴用於避免與其他框架命名衝突。
Apple框架使用雙字母前綴，React Native使用`RCT`前綴。為避免衝突，建議在自訂類別中使用非`RCT`的三字母前綴。
:::

接著需要少量JavaScript程式碼使其成為可用的React元件：

```tsx {3} title="MapView.tsx"
import {requireNativeComponent} from 'react-native';

export default requireNativeComponent('RNTMap');
```

The `requireNativeComponent` function automatically resolves `RNTMap` to `RNTMapManager` and exports our native view for use in JavaScript.

```tsx title="MyApp.tsx"
import MapView from './MapView.tsx';

export default function MyApp() {
  return <MapView style={{flex: 1}} />;
}
```

:::note
渲染時請記得拉伸視圖，否則您將只會看到空白畫面。
:::

現在這已是功能完整的原生地圖視圖元件，支援雙指縮放等原生手勢操作。但目前還無法透過JavaScript控制它。

## 屬性

增強元件可用性的第一步是橋接部分原生屬性。假設我們要實現禁用縮放和指定可見區域功能。禁用縮放是布林值屬性，只需添加這行程式碼：

```objectivec title='RNTMapManager.m'
RCT_EXPORT_VIEW_PROPERTY(zoomEnabled, BOOL)
```

請注意我們明確指定類型為 `BOOL`——React Native 在橋接通信時會使用 `RCTConvert` 來轉換各種數據類型，錯誤值會顯示方便的「RedBox」錯誤以便您立即發現問題。當邏輯如此簡單時，這個宏會自動處理整個實現過程。

現在要實際禁用縮放功能，我們在 JavaScript 中設置屬性：

```tsx {4} title="MyApp.tsx"
import MapView from './MapView.tsx';

export default function MyApp() {
  return <MapView zoomEnabled={false} style={{flex: 1}} />;
}
```

為了記錄 MapView 元件的屬性（及其接受的數值），我們將添加一個包裝元件並使用 TypeScript 定義介面：

```tsx {6-9} title="MapView.tsx"
import {requireNativeComponent} from 'react-native';

const RNTMap = requireNativeComponent('RNTMap');

export default function MapView(props: {
  /**
   * Whether the user may use pinch gestures to zoom in and out.
   */
  zoomEnabled?: boolean;
}) {
  return <RNTMap {...props} />;
}
```

現在我們有了一個文檔完善的包裝元件可供使用。

接著添加更複雜的 `region` 屬性。首先添加原生代碼：

```objectivec title='RNTMapManager.m'
RCT_CUSTOM_VIEW_PROPERTY(region, MKCoordinateRegion, MKMapView)
{
  [view setRegion:json ? [RCTConvert MKCoordinateRegion:json] : defaultView.region animated:YES];
}
```

這比之前的 `BOOL` 情況更複雜。現在需要為 `MKCoordinateRegion` 類型編寫轉換函數，並添加自定義代碼使視圖在從 JS 設置區域時產生動畫效果。在提供的函數體內，`json` 指向從 JS 傳遞的原始值，`view` 變量可訪問管理器視圖實例，而 `defaultView` 用於在 JS 傳送 null 時將屬性重置為默認值。

You could write any conversion function you want for your view - here is the implementation for `MKCoordinateRegion` via a category on `RCTConvert`. It uses an already existing category of ReactNative `RCTConvert+CoreLocation`:

```objectivec title='RNTMapManager.m'
#import "RCTConvert+Mapkit.h"
```

```objectivec title='RCTConvert+Mapkit.h'
#import <MapKit/MapKit.h>
#import <React/RCTConvert.h>
#import <CoreLocation/CoreLocation.h>
#import <React/RCTConvert+CoreLocation.h>

@interface RCTConvert (Mapkit)

+ (MKCoordinateSpan)MKCoordinateSpan:(id)json;
+ (MKCoordinateRegion)MKCoordinateRegion:(id)json;

@end

@implementation RCTConvert(MapKit)

+ (MKCoordinateSpan)MKCoordinateSpan:(id)json
{
  json = [self NSDictionary:json];
  return (MKCoordinateSpan){
    [self CLLocationDegrees:json[@"latitudeDelta"]],
    [self CLLocationDegrees:json[@"longitudeDelta"]]
  };
}

+ (MKCoordinateRegion)MKCoordinateRegion:(id)json
{
  return (MKCoordinateRegion){
    [self CLLocationCoordinate2D:json],
    [self MKCoordinateSpan:json]
  };
}

@end
```

這些轉換函數會安全處理 JS 可能傳送的任何 JSON，當遇到缺失鍵值或其他開發錯誤時，會顯示「RedBox」錯誤並返回標準初始化值。

最後為 `region` 屬性添加 TypeScript 文檔：

```tsx {6-25} title="MapView.tsx"
import {requireNativeComponent} from 'react-native';

const RNTMap = requireNativeComponent('RNTMap');

export default function MapView(props: {
  /**
   * The region to be displayed by the map.
   *
   * The region is defined by the center coordinates and the span of
   * coordinates to display.
   */
  region?: {
    /**
     * Coordinates for the center of the map.
     */
    latitude: number;
    longitude: number;

    /**
     * Distance between the minimum and the maximum latitude/longitude
     * to be displayed.
     */
    latitudeDelta: number;
    longitudeDelta: number;
  };
  /**
   * Whether the user may use pinch gestures to zoom in and out.
   */
  zoomEnabled?: boolean;
}) {
  return <RNTMap {...props} />;
}
```

We can now supply the `region` prop to `MapView`:

```tsx {4-9,12} title="MyApp.tsx"
import MapView from './MapView.tsx';

export default function MyApp() {
  const region = {
    latitude: 37.48,
    longitude: -122.16,
    latitudeDelta: 0.1,
    longitudeDelta: 0.1,
  };
  return (
    <MapView
      region={region}
      zoomEnabled={false}
      style={{flex: 1}}
    />
  );
}
```

## 事件處理

現在我們有了可從 JS 自由控制的原生地圖元件，但如何處理用戶事件（如縮放或平移改變可見區域）？

Until now we've only returned a `MKMapView` instance from our manager's `-(UIView *)view` method. We can't add new properties to `MKMapView` so we have to create a new subclass from `MKMapView` which we use for our View. We can then add a `onRegionChange` callback on this subclass:

```objectivec title='RNTMapView.h'
#import <MapKit/MapKit.h>

#import <React/RCTComponent.h>

@interface RNTMapView: MKMapView

@property (nonatomic, copy) RCTBubblingEventBlock onRegionChange;

@end
```

```objectivec title='RNTMapView.m'
#import "RNTMapView.h"

@implementation RNTMapView

@end
```

注意所有 `RCTBubblingEventBlock` 必須以 `on` 為前綴。接著在 `RNTMapManager` 上聲明事件處理屬性，使其成為所有暴露視圖的委託，並通過從原生視圖調用事件處理塊來將事件轉發至 JS。

```objectivec {9,17,31-48} title='RNTMapManager.m'
#import <MapKit/MapKit.h>
#import <React/RCTViewManager.h>

#import "RNTMapView.h"
#import "RCTConvert+Mapkit.h"

@interface RNTMapManager : RCTViewManager <MKMapViewDelegate>
@end

@implementation RNTMapManager

RCT_EXPORT_MODULE()

RCT_EXPORT_VIEW_PROPERTY(zoomEnabled, BOOL)
RCT_EXPORT_VIEW_PROPERTY(onRegionChange, RCTBubblingEventBlock)

RCT_CUSTOM_VIEW_PROPERTY(region, MKCoordinateRegion, MKMapView)
{
  [view setRegion:json ? [RCTConvert MKCoordinateRegion:json] : defaultView.region animated:YES];
}

- (UIView *)view
{
  RNTMapView *map = [RNTMapView new];
  map.delegate = self;
  return map;
}

#pragma mark MKMapViewDelegate

- (void)mapView:(RNTMapView *)mapView regionDidChangeAnimated:(BOOL)animated
{
  if (!mapView.onRegionChange) {
    return;
  }

  MKCoordinateRegion region = mapView.region;
  mapView.onRegionChange(@{
    @"region": @{
      @"latitude": @(region.center.latitude),
      @"longitude": @(region.center.longitude),
      @"latitudeDelta": @(region.span.latitudeDelta),
      @"longitudeDelta": @(region.span.longitudeDelta),
    }
  });
}
@end
```

在委託方法 `-mapView:regionDidChangeAnimated:` 中，會使用區域數據調用對應視圖的事件處理塊。調用 `onRegionChange` 事件處理塊會觸發 JavaScript 中相同的回調屬性。此回調會接收原始事件，我們通常在包裝元件中處理以簡化 API：

```tsx {3-10,14-17,19} title="MapView.tsx"
// ...

type RegionChangeEvent = {
  nativeEvent: {
    latitude: number;
    longitude: number;
    latitudeDelta: number;
    longitudeDelta: number;
  };
};

export default function MapView(props: {
  // ...
  /**
   * Callback that is called continuously when the user is dragging the map.
   */
  onRegionChange: (event: RegionChangeEvent) => unknown;
}) {
  return <RNTMap {...props} onRegionChange={onRegionChange} />;
}
```

```tsx {6-9,14} title="MyApp.tsx"
import MapView from './MapView.tsx';

export default function MyApp() {
  // ...

  const onRegionChange = useCallback(event => {
    const {region} = event.nativeEvent;
    // Do something with `region.latitude`, etc.
  });

  return (
    <MapView
      // ...
      onRegionChange={onRegionChange}
    />
  );
}
```

## 處理多個原生視圖

React Native 視圖在視圖樹中可以擁有多個子視圖，例如：

```tsx
<View>
  <MyNativeView />
  <MyNativeView />
  <Button />
</View>
```

此範例中，`MyNativeView` 類是 `NativeComponent` 的包裝器，並暴露將在 iOS 平台調用的方法。`MyNativeView` 定義於 `MyNativeView.ios.js` 文件中，包含 `NativeComponent` 的代理方法。

When the user interacts with the component, like clicking the button, the `backgroundColor` of `MyNativeView` changes. In this case `UIManager` would not know which `MyNativeView` should be handled and which one should change `backgroundColor`. Below you will find a solution to this problem:

```tsx
<View>
  <MyNativeView ref={this.myNativeReference} />
  <MyNativeView ref={this.myNativeReference2} />
  <Button
    onPress={() => {
      this.myNativeReference.callNativeMethod();
    }}
  />
</View>
```

現在，上述組件具有對特定 `MyNativeView` 的引用，這使我們能夠使用 `MyNativeView` 的特定實例。現在按鈕可以控制哪一個 `MyNativeView` 應該改變其 `backgroundColor`。在此範例中，假設 `callNativeMethod` 會改變 `backgroundColor`。

```tsx title="MyNativeView.ios.tsx"
class MyNativeView extends React.Component {
  callNativeMethod = () => {
    UIManager.dispatchViewManagerCommand(
      ReactNative.findNodeHandle(this),
      UIManager.getViewManagerConfig('RNCMyNativeView').Commands
        .callNativeMethod,
      [],
    );
  };

  render() {
    return <NativeComponent ref={NATIVE_COMPONENT_REF} />;
  }
}
```

`callNativeMethod` is our custom iOS method which for example changes the `backgroundColor` which is exposed through `MyNativeView`. This method uses `UIManager.dispatchViewManagerCommand` which needs 3 parameters:

- `(nonnull NSNumber \*)reactTag`  -  react 視圖的 id。
- `commandID:(NSInteger)commandID`  -  應該調用的原生方法的 Id。
- `commandArgs:(NSArray<id> \*)commandArgs`  -  我們可以從 JS 傳遞到原生的原生方法的參數。

```objectivec title='RNCMyNativeViewManager.m'
#import <React/RCTViewManager.h>
#import <React/RCTUIManager.h>
#import <React/RCTLog.h>

RCT_EXPORT_METHOD(callNativeMethod:(nonnull NSNumber*) reactTag) {
    [self.bridge.uiManager addUIBlock:^(RCTUIManager *uiManager, NSDictionary<NSNumber *,UIView *> *viewRegistry) {
        NativeView *view = viewRegistry[reactTag];
        if (!view || ![view isKindOfClass:[NativeView class]]) {
            RCTLogError(@"Cannot find NativeView with tag #%@", reactTag);
            return;
        }
        [view callNativeMethod];
    }];

}
```

Here the `callNativeMethod` is defined in the `RNCMyNativeViewManager.m` file and contains only one parameter which is `(nonnull NSNumber*) reactTag`. This exported function will find a particular view using `addUIBlock` which contains the `viewRegistry` parameter and returns the component based on `reactTag` allowing it to call the method on the correct component.

## 樣式

由於我們所有的原生 react 視圖都是 `UIView` 的子類，大多數樣式屬性都會如預期般工作。然而，某些組件可能需要默認樣式，例如固定大小的 `UIDatePicker`。這種默認樣式對於佈局算法正常工作非常重要，但我們也希望在使用組件時能夠覆蓋默認樣式。`DatePickerIOS` 通過將原生組件包裝在一個具有靈活樣式的額外視圖中，並在內部原生組件上使用固定樣式（該樣式是通過從原生傳遞的常量生成的）來實現這一點：

```tsx title="DatePickerIOS.ios.tsx"
import {UIManager} from 'react-native';
const RCTDatePickerIOSConsts = UIManager.RCTDatePicker.Constants;
...
  render: function() {
    return (
      <View style={this.props.style}>
        <RCTDatePickerIOS
          ref={DATEPICKER}
          style={styles.rkDatePickerIOS}
          ...
        />
      </View>
    );
  }
});

const styles = StyleSheet.create({
  rkDatePickerIOS: {
    height: RCTDatePickerIOSConsts.ComponentHeight,
    width: RCTDatePickerIOSConsts.ComponentWidth,
  },
});
```

The `RCTDatePickerIOSConsts` constants are exported from native by grabbing the actual frame of the native component like so:

```objectivec title='RCTDatePickerManager.m'
- (NSDictionary *)constantsToExport
{
  UIDatePicker *dp = [[UIDatePicker alloc] init];
  [dp layoutIfNeeded];

  return @{
    @"ComponentHeight": @(CGRectGetHeight(dp.frame)),
    @"ComponentWidth": @(CGRectGetWidth(dp.frame)),
    @"DatePickerModes": @{
      @"time": @(UIDatePickerModeTime),
      @"date": @(UIDatePickerModeDate),
      @"datetime": @(UIDatePickerModeDateAndTime),
    }
  };
}
```

本指南涵蓋了橋接自定義原生組件的許多方面，但還有更多可能需要考慮的內容，例如用於插入和佈局子視圖的自定義鉤子。如果你想更深入，可以查看一些已實現組件的[源代碼](https://github.com/facebook/react-native/tree/main/packages/react-native/React/Views)。