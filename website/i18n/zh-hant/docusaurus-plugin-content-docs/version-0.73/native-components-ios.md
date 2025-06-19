---
id: native-components-ios
title: iOS Native UI Components
---

import NativeDeprecated from './the-new-architecture/\_markdown_native_deprecation.mdx'

<NativeDeprecated />

現有大量原生UI元件可供最新應用程式使用——有些是平台內建元件，有些來自第三方函式庫，還有一些可能是您過去開發專案時自行建構的。React Native 已封裝了部分核心平台元件（如 `ScrollView` 和 `TextInput`），但並未涵蓋所有元件，特別是您可能為前一個應用程式自訂的元件。幸運的是，我們可以將這些現有元件封裝起來，無縫整合至您的 React Native 應用中。

與原生模組指南類似，這也是一份進階指南，假設您已具備iOS開發基礎知識。本指南將透過實作React Native核心函式庫中現有 `MapView` 元件的部分功能，示範如何建構原生UI元件。

## iOS MapView 範例

假設我們想在應用程式中加入互動式地圖——不妨直接使用 [`MKMapView`](https://developer.apple.com/library/prerelease/mac/documentation/MapKit/Reference/MKMapView_Class/index.html)，我們只需讓它能透過JavaScript操作即可。

Native views are created and manipulated by subclasses of `RCTViewManager`. These subclasses are similar in function to view controllers, but are essentially singletons - only one instance of each is created by the bridge. They expose native views to the `RCTUIManager`, which delegates back to them to set and update the properties of the views as necessary. The `RCTViewManager`s are also typically the delegates for the views, sending events back to JavaScript via the bridge.

要暴露一個視圖，您可以：

- 繼承 `RCTViewManager` 來為您的元件建立管理器
- 添加 `RCT_EXPORT_MODULE()` 標記宏
- 實作 `-(UIView *)view` 方法

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
在上面的範例中，我們為類別名稱添加了 `RNT` 前綴。前綴用於避免與其他框架的名稱衝突。
Apple框架使用兩個字母的前綴，React Native 使用 `RCT` 作為前綴。為避免名稱衝突，我們建議在您自己的類別中使用非 `RCT` 的三字母前綴。
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

請確保此處使用 `RNTMap`。我們需要在此引入管理器，它會將我們管理器的視圖暴露給JavaScript使用。

:::note
渲染時請記得拉伸視圖，否則您將只會看到空白畫面。
:::

```tsx
  render() {
    return <MapView style={{flex: 1}} />;
  }
```

現在這已是一個功能完整的JavaScript原生地圖視圖元件，支援雙指縮放等原生手勢操作。不過我們還無法真正從JavaScript控制它 :(

## 屬性

首先要讓元件更實用的方法是橋接一些原生屬性。假設我們想禁用縮放並指定可見區域。禁用縮放是布林值，因此我們添加這行程式碼：

```objectivec title='RNTMapManager.m'
RCT_EXPORT_VIEW_PROPERTY(zoomEnabled, BOOL)
```

請注意，我們明確將類型指定為 `BOOL`——React Native 在橋接通信時會使用 `RCTConvert` 來轉換各種不同的數據類型，錯誤的值會顯示方便的「RedBox」錯誤，讓您立即知道問題所在。當情況如此簡單時，整個實現過程都會由這個宏自動處理。

現在要實際禁用縮放功能，我們在 JS 中設置屬性：

```tsx title="MyApp.tsx"
<MapView zoomEnabled={false} style={{flex: 1}} />
```

To document the properties (and which values they accept) of our MapView component we'll add a wrapper component and document the interface with React `PropTypes`:

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

接下來，讓我們添加更複雜的 `region` 屬性。我們首先添加原生代碼：

```objectivec title='RNTMapManager.m'
RCT_CUSTOM_VIEW_PROPERTY(region, MKCoordinateRegion, MKMapView)
{
  [view setRegion:json ? [RCTConvert MKCoordinateRegion:json] : defaultView.region animated:YES];
}
```

好的，這比之前處理的 `BOOL` 情況要複雜得多。現在我們有一個需要轉換函數的 `MKCoordinateRegion` 類型，並且我們有自定義代碼，以便在從 JS 設置區域時視圖會動畫過渡。在我們提供的函數體內，`json` 指的是從 JS 傳遞過來的原始值。還有一個 `view` 變量，它讓我們可以訪問管理器的視圖實例，以及一個 `defaultView`，我們用它來在 JS 發送空值時將屬性重置為默認值。

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

這些轉換函數設計用於安全處理 JS 可能拋出的任何 JSON，當遇到缺失鍵或其他開發者錯誤時，會顯示「RedBox」錯誤並返回標準的初始化值。

為了完成對 `region` 屬性的支持，我們需要在 `propTypes` 中記錄它：

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

在這裡，您可以看到區域的形狀在 JS 文檔中是明確的。

## 事件處理

現在我們有一個可以從 JS 自由控制的原生地圖組件，但我們如何處理來自用戶的事件，例如捏合縮放或平移以改變可見區域？

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

請注意，所有 `RCTBubblingEventBlock` 都必須以 `on` 為前綴。接下來，在 `RNTMapManager` 上聲明一個事件處理程序屬性，使其成為所有暴露視圖的委託，並通過從原生視圖調用事件處理程序塊來將事件轉發到 JS。

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

在委託方法 `-mapView:regionDidChangeAnimated:` 中，事件處理程序塊會在相應的視圖上被調用，並帶有區域數據。調用 `onRegionChange` 事件處理程序塊會導致在 JavaScript 中調用相同的回調屬性。這個回調會以原始事件的形式被調用，我們通常在包裝組件中處理它以簡化 API：

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

一個 React Native 視圖在視圖樹中可以擁有多個子視圖，例如：

```tsx
<View>
  <MyNativeView />
  <MyNativeView />
  <Button />
</View>
```

在此範例中，`MyNativeView` 類別是 `NativeComponent` 的封裝器，並公開將在 iOS 平台上呼叫的方法。`MyNativeView` 定義於 `MyNativeView.ios.js` 檔案中，包含 `NativeComponent` 的代理方法。

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

現在上述元件已參照特定的 `MyNativeView`，這讓我們能使用 `MyNativeView` 的特定實例。按鈕現在可控制哪個 `MyNativeView` 應變更其 `backgroundColor`。在此範例中，我們假設 `callNativeMethod` 會改變 `backgroundColor`。

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
- `commandID:(NSInteger)commandID`  -  應呼叫的原生方法 id。
- `commandArgs:(NSArray<id> \*)commandArgs`  -  我們可從 JS 傳遞至原生的方法參數。

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

由於我們所有的原生 react 視圖都是 `UIView` 的子類別，大多數樣式屬性都能如預期般直接運作。然而某些元件會需要預設樣式，例如固定尺寸的 `UIDatePicker`。這個預設樣式對佈局演算法正常運作很重要，但我們也希望能在使用元件時覆寫預設樣式。`DatePickerIOS` 透過將原生元件包裝在具有彈性樣式的額外視圖中，並對內部原生元件使用固定樣式（此樣式由從原生傳入的常數產生）來實現此功能：

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

本指南涵蓋了橋接自訂原生元件的許多面向，但您可能還需考慮其他事項，例如用於插入和佈局子視圖的自訂鉤子。若想更深入瞭解，可參考一些已實作元件的[原始碼](https://github.com/facebook/react-native/tree/main/packages/react-native/React/Views)。