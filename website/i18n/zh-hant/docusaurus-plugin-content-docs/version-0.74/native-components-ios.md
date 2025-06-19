---
id: native-components-ios
title: iOS Native UI Components
---

import NativeDeprecated from './the-new-architecture/\_markdown_native_deprecation.mdx'

<NativeDeprecated />

現有大量原生UI元件可供最新應用程式使用——有些是平台內建元件，有些來自第三方函式庫，還有些可能是您過去開發的專屬元件。React Native已封裝了部分核心平台元件（如`ScrollView`和`TextInput`），但並未涵蓋所有元件，特別是您過往應用中自行開發的元件。幸運的是，我們能將這些現有元件封裝整合至React Native應用中。

與原生模組指南類似，本篇同屬進階指南，假設您已具備iOS開發基礎知識。本文將透過實作React Native核心函式庫中現有`MapView`元件的部分功能，示範如何建構原生UI元件。

## iOS MapView範例

假設我們需要在應用中加入互動式地圖——與其重新造輪子，不如直接使用[`MKMapView`](https://developer.apple.com/library/prerelease/mac/documentation/MapKit/Reference/MKMapView_Class/index.html)，我們只需讓它能透過JavaScript操作即可。

原生視圖由`RCTViewManager`的子類別創建和管理。這些子類別功能類似視圖控制器，但實際上是單例模式——橋接器只會創建每個子類別的單一實例。它們將原生視圖暴露給`RCTUIManager`，由後者委派設置和更新視圖屬性。`RCTViewManager`通常也作為視圖的委託，透過橋接器將事件回傳至JavaScript。

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
上述範例中，我們使用`RNT`作為類別名稱前綴。前綴用於避免與其他框架命名衝突。
Apple框架使用雙字母前綴，React Native使用`RCT`前綴。為避免命名衝突，建議在自訂類別中使用非`RCT`的三字母前綴。
:::

接著您需要少量JavaScript程式碼使其成為可用的React元件：

```tsx title="MapView.tsx"
import {requireNativeComponent} from 'react-native';

// requireNativeComponent automatically resolves 'RNTMap' to 'RNTMapManager'
module.exports = requireNativeComponent('RNTMap');
```

```tsx title="MyApp.tsx"
import MapView from './MapView.tsx';

...

render() {
  return <MapView style={{flex: 1}} />;
}
```

請確保此處使用`RNTMap`。我們需要在此引入管理器，它會將管理器的視圖暴露給JavaScript使用。

:::note
渲染時請記得拉伸視圖，否則您將只看到空白畫面。
:::

```tsx
  render() {
    return <MapView style={{flex: 1}} />;
  }
```

現在這已是功能完整的原生地圖視圖元件，支援雙指縮放等原生手勢操作。但我們還無法透過JavaScript控制它 :(

## 屬性

增強元件可用性的第一步是橋接部分原生屬性。假設我們想禁用縮放並指定可見區域。禁用縮放是布林值，只需添加這行程式碼：

```objectivec title='RNTMapManager.m'
RCT_EXPORT_VIEW_PROPERTY(zoomEnabled, BOOL)
```

請注意我們明確指定類型為 `BOOL`——React Native 在橋接通信時會使用 `RCTConvert` 來轉換各種數據類型，若傳入錯誤值會立即顯示「RedBox」錯誤提示。當參數類型如此簡單時，這個宏會自動處理整個實現過程。

現在要實際禁用縮放功能，我們在 JS 中設置屬性：

```tsx title="MyApp.tsx"
<MapView zoomEnabled={false} style={{flex: 1}} />
```

為了記錄 MapView 組件的屬性（及其可接受值），我們將添加一個包裝組件並使用 React 的 `PropTypes` 來定義接口：

```tsx title="MapView.tsx"
import PropTypes from 'prop-types';
import React from 'react';
import {requireNativeComponent} from 'react-native';

class MapView extends React.Component {
  render() {
    return <RNTMap {...this.props} />;
  }
}

MapView.propTypes = {
  /**
   * A Boolean value that determines whether the user may use pinch
   * gestures to zoom in and out of the map.
   */
  zoomEnabled: PropTypes.bool,
};

const RNTMap = requireNativeComponent('RNTMap');

module.exports = MapView;
```

現在我們有了一個文檔完善的包裝組件可供使用。

接著添加更複雜的 `region` 屬性。首先添加原生代碼：

```objectivec title='RNTMapManager.m'
RCT_CUSTOM_VIEW_PROPERTY(region, MKCoordinateRegion, MKMapView)
{
  [view setRegion:json ? [RCTConvert MKCoordinateRegion:json] : defaultView.region animated:YES];
}
```

這比之前的 `BOOL` 情況複雜許多。現在需要處理 `MKCoordinateRegion` 類型的轉換函數，並通過自定義代碼實現從 JS 設置區域時的動畫效果。在提供的函數體內，`json` 指向從 JS 傳入的原始值，`view` 變量可訪問管理器視圖實例，而 `defaultView` 用於在 JS 傳入空值時將屬性重置為默認值。

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

這些轉換函數會安全處理 JS 可能傳入的任何 JSON 數據，當遇到缺失鍵值或其他開發錯誤時，會顯示「RedBox」錯誤並返回標準初始化值。

要完成對 `region` 屬性的支持，需在 `propTypes` 中進行文檔說明：

```tsx title="MapView.tsx"
MapView.propTypes = {
  /**
   * A Boolean value that determines whether the user may use pinch
   * gestures to zoom in and out of the map.
   */
  zoomEnabled: PropTypes.bool,

  /**
   * The region to be displayed by the map.
   *
   * The region is defined by the center coordinates and the span of
   * coordinates to display.
   */
  region: PropTypes.shape({
    /**
     * Coordinates for the center of the map.
     */
    latitude: PropTypes.number.isRequired,
    longitude: PropTypes.number.isRequired,

    /**
     * Distance between the minimum and the maximum latitude/longitude
     * to be displayed.
     */
    latitudeDelta: PropTypes.number.isRequired,
    longitudeDelta: PropTypes.number.isRequired,
  }),
};
```

```tsx title="MyApp.tsx"
render() {
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

此處可清晰看到 region 的結構已在 JS 文檔中明確定義。

## 事件處理

現在我們有了可從 JS 自由控制的原生地圖組件，但如何處理用戶事件（如縮放手勢或平移改變可見區域）？

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

注意所有 `RCTBubblingEventBlock` 必須以 `on` 為前綴。接著在 `RNTMapManager` 聲明事件處理屬性，使其成為所有暴露視圖的代理，並通過從原生視圖調用事件處理塊來將事件傳遞至 JS。

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

在代理方法 `-mapView:regionDidChangeAnimated:` 中，會使用區域數據在對應視圖上調用事件處理塊。調用 `onRegionChange` 事件處理塊會觸發 JS 中相同的回調 prop。該回調會接收原始事件，通常我們會在包裝組件中處理以簡化 API：

```tsx title="MapView.tsx"
class MapView extends React.Component {
  _onRegionChange = event => {
    if (!this.props.onRegionChange) {
      return;
    }

    // process raw event...
    this.props.onRegionChange(event.nativeEvent);
  };
  render() {
    return (
      <RNTMap
        {...this.props}
        onRegionChange={this._onRegionChange}
      />
    );
  }
}
MapView.propTypes = {
  /**
   * Callback that is called continuously when the user is dragging the map.
   */
  onRegionChange: PropTypes.func,
  ...
};
```

```tsx title="MyApp.tsx"
class MyApp extends React.Component {
  onRegionChange(event) {
    // Do stuff with event.region.latitude, etc.
  }

  render() {
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
        onRegionChange={this.onRegionChange}
      />
    );
  }
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

在這個範例中，`MyNativeView` 類別是 `NativeComponent` 的封裝器，並公開了將在 iOS 平台上呼叫的方法。`MyNativeView` 定義於 `MyNativeView.ios.js` 檔案中，包含了 `NativeComponent` 的代理方法。

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

現在上述元件具有對特定 `MyNativeView` 的參考，這使我們能夠使用 `MyNativeView` 的特定實例。現在按鈕可以控制哪個 `MyNativeView` 應該改變其 `backgroundColor`。在此範例中，我們假設 `callNativeMethod` 會改變 `backgroundColor`。

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

- `(nonnull NSNumber \*)reactTag`  -  React 視圖的 ID。
- `commandID:(NSInteger)commandID`  -  應呼叫的原生方法 ID。
- `commandArgs:(NSArray<id> \*)commandArgs`  -  我們可以從 JS 傳遞到原生的方法參數。

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

由於我們所有的原生 React 視圖都是 `UIView` 的子類別，大多數樣式屬性都能如預期般直接使用。然而，某些元件可能需要預設樣式，例如固定大小的 `UIDatePicker`。這種預設樣式對於佈局演算法正常運作非常重要，但我們也希望在使用元件時能夠覆寫預設樣式。`DatePickerIOS` 透過將原生元件包裝在一個具有彈性樣式的額外視圖中，並在內部原生元件上使用固定樣式（由原生傳遞的常數生成）來實現這一點：

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

本指南涵蓋了橋接自訂原生元件的許多面向，但您可能還需要考慮更多因素，例如用於插入和佈局子視圖的自訂鉤子。如果您想更深入瞭解，可以查看一些已實作元件的[原始碼](https://github.com/facebook/react-native/tree/main/packages/react-native/React/Views)。