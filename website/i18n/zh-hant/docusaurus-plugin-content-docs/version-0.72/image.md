---
id: image
title: Image
---

一個用於顯示不同類型圖片的 React 元件，包括網路圖片、靜態資源、臨時本地圖片以及來自本地磁碟的圖片（例如相機膠卷中的圖片）。

此範例展示如何從本地儲存空間、網路甚至使用 `'data:'` URI 方案提供的數據中獲取並顯示圖片。

> 請注意，對於網路和數據圖片，您需要手動指定圖片的尺寸！

## 範例

```SnackPlayer name=Example
import React from 'react';
import {View, Image, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  container: {
    paddingTop: 50,
  },
  tinyLogo: {
    width: 50,
    height: 50,
  },
  logo: {
    width: 66,
    height: 58,
  },
});

const DisplayAnImage = () => {
  return (
    <View style={styles.container}>
      <Image
        style={styles.tinyLogo}
        source={require('@expo/snack-static/react-native-logo.png')}
      />
      <Image
        style={styles.tinyLogo}
        source={{
          uri: 'https://reactnative.dev/img/tiny_logo.png',
        }}
      />
      <Image
        style={styles.logo}
        source={{
          uri: 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADMAAAAzCAYAAAA6oTAqAAAAEXRFWHRTb2Z0d2FyZQBwbmdjcnVzaEB1SfMAAABQSURBVGje7dSxCQBACARB+2/ab8BEeQNhFi6WSYzYLYudDQYGBgYGBgYGBgYGBgYGBgZmcvDqYGBgmhivGQYGBgYGBgYGBgYGBgYGBgbmQw+P/eMrC5UTVAAAAABJRU5ErkJggg==',
        }}
      />
    </View>
  );
};

export default DisplayAnImage;
```

您也可以為圖片添加 `style`：

```SnackPlayer name=Example
import React from 'react';
import {View, Image, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  container: {
    paddingTop: 50,
  },
  stretch: {
    width: 50,
    height: 200,
    resizeMode: 'stretch',
  },
});

const DisplayAnImageWithStyle = () => {
  return (
    <View style={styles.container}>
      <Image
        style={styles.stretch}
        source={require('@expo/snack-static/react-native-logo.png')}
      />
    </View>
  );
};

export default DisplayAnImageWithStyle;
```

## Android 上的 GIF 和 WebP 支援

在自行構建原生代碼時，Android 預設不支援 GIF 和 WebP 格式。

您需要根據應用需求，在 `android/app/build.gradle` 中添加一些可選模組。

```groovy
dependencies {
  // If your app supports Android versions before Ice Cream Sandwich (API level 14)
  implementation 'com.facebook.fresco:animated-base-support:1.3.0'

  // For animated GIF support
  implementation 'com.facebook.fresco:animated-gif:2.5.0'

  // For WebP support, including animated WebP
  implementation 'com.facebook.fresco:animated-webp:2.5.0'
  implementation 'com.facebook.fresco:webpsupport:2.5.0'

  // For WebP support, without animations
  implementation 'com.facebook.fresco:webpsupport:2.5.0'
}
```

> 注意：上述列出的版本可能未及時更新。請查閱主倉庫中的 [`ReactAndroid/gradle.properties`](https://github.com/facebook/react-native/blob/main/packages/react-native/ReactAndroid/gradle.properties) 以查看特定標記版本中使用的 fresco 版本。

---

# 參考文檔

## 屬性

### [View 屬性](view.md#props)

繼承 [View 屬性](view#props)。

---

### `accessible`

當設為 true 時，表示該圖片是一個無障礙元素。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `accessibilityLabel`

當用戶與圖片互動時，屏幕閱讀器會朗讀的文字。

| Type   |
| ------ |
| string |

---

### `alt`

定義圖片替代文字描述的字符串，當用戶與圖片互動時，屏幕閱讀器會朗讀此文字。使用此屬性會自動將該元素標記為無障礙元素。

| Type   |
| ------ |
| string |

---

### `blurRadius`

blurRadius：添加到圖片的模糊濾鏡的模糊半徑。

| Type   |
| ------ |
| number |

> 提示：在 iOS 上，您需要將 `blurRadius` 設為大於 `5` 的值。

---

### `capInsets` <div class="label ios">iOS</div>

當圖片調整大小時，由 `capInsets` 指定的尺寸的角落將保持固定大小，但圖片的中心內容和邊框將被拉伸。這對於創建可調整大小的圓角按鈕、陰影和其他可調整大小的資源非常有用。更多資訊請參閱 [Apple 官方文檔](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIImage_Class/index.html#//apple_ref/occ/instm/UIImage/resizableImageWithCapInsets)。

| Type         |
| ------------ |
| [Rect](rect) |

---

### `crossOrigin`

一個指定在獲取圖片資源時使用的 CORS 模式的關鍵字字符串。其功能類似於 HTML 中的 crossorigin 屬性。

- `anonymous`：圖片請求中不交換用戶憑證。
- `use-credentials`：在圖片請求中將 `Access-Control-Allow-Credentials` 標頭值設為 `true`。

| Type                                     | Default       |
| ---------------------------------------- | ------------- |
| enum(`'anonymous'`, `'use-credentials'`) | `'anonymous'` |

---

### `defaultSource`

在加載圖片來源時顯示的靜態圖片。

| Type                             |
| -------------------------------- |
| [ImageSource](image#imagesource) |

> **注意：** 在 Android 上，debug 版本會忽略預設的 source 屬性。

---

### `fadeDuration` <div class="label android">Android</div>

淡入淡出動畫的持續時間（毫秒）。

| Type   | Default |
| ------ | ------- |
| number | `300`   |

---

### `height`

圖片元件的高度。

| Type   |
| ------ |
| number |

---

### `loadingIndicatorSource`

與 `source` 類似，此屬性代表用於渲染圖片載入指示器的資源。載入指示器會在圖片下載完成並準備顯示前持續顯示。

| Type                                                  |
| ----------------------------------------------------- |
| [ImageSource](image#imagesource) (`uri` only), number |

---

### `onError`

載入錯誤時觸發。

| Type                                |
| ----------------------------------- |
| (`{nativeEvent: {error} }`) => void |

---

### `onLayout`

在元件掛載或佈局變更時觸發。

| Type                                                    |
| ------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)} => void` |

---

### `onLoad`

載入成功完成時觸發。

**範例：** `onLoad={({nativeEvent: {source: {width, height}}}) => setImageRealSize({width, height})}`

| Type                                                                |
| ------------------------------------------------------------------- |
| `md ({nativeEvent: [ImageLoadEvent](image#imageloadevent)} => void` |

---

### `onLoadEnd`

無論載入成功或失敗時皆會觸發。

| Type       |
| ---------- |
| () => void |

---

### `onLoadStart`

載入開始時觸發。

**範例：** `onLoadStart={() => this.setState({loading: true})}`

| Type       |
| ---------- |
| () => void |

---

### `onPartialLoad` <div class="label ios">iOS</div>

當圖片部分載入完成時觸發。「部分載入」的定義取決於載入器，但此事件主要用於漸進式 JPEG 載入。

| Type       |
| ---------- |
| () => void |

---

### `onProgress`

下載進度更新時觸發。

| Type                                        |
| ------------------------------------------- |
| (`{nativeEvent: {loaded, total} }`) => void |

---

### `progressiveRenderingEnabled` <div class="label android">Android</div>

When `true`, enables progressive jpeg streaming - https://frescolib.org/docs/progressive-jpegs.

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `resizeMethod` <div class="label android">Android</div>

當圖片尺寸與圖片視圖尺寸不符時，用於調整圖片大小的機制。預設為 `auto`。

- `auto`：根據啟發式方法在 `resize` 和 `scale` 之間選擇。

- `resize`：一種軟體操作，會在解碼前於記憶體中修改編碼後的圖片。當圖片遠大於視圖時應使用此方法而非 `scale`。

- `scale`：圖片會被縮小或放大繪製。與 `resize` 相比，`scale` 速度更快（通常由硬體加速）且產生更高品質的圖片。若圖片小於視圖應使用此方法；若圖片略大於視圖時也建議使用。

More details about `resize` and `scale` can be found at http://frescolib.org/docs/resizing.

| Type                                  | Default  |
| ------------------------------------- | -------- |
| enum(`'auto'`, `'resize'`, `'scale'`) | `'auto'` |

---

### `referrerPolicy`

一個字串，用於指定獲取資源時使用的引用來源。設定圖片請求中 `Referrer-Policy` 標頭的值，功能類似於 HTML 中的 `referrerpolicy` 屬性。

| Type                                                                                                                                                                                     | Default                             |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| enum(`'no-referrer'`, `'no-referrer-when-downgrade'`, `'origin'`, `'origin-when-cross-origin'`, `'same-origin'`, `'strict-origin'`, `'strict-origin-when-cross-origin'`, `'unsafe-url'`) | `'strict-origin-when-cross-origin'` |

---

### `resizeMode`

決定當框架與原始圖片尺寸不符時如何調整圖片大小。預設為 `cover`。

- `cover`: 均勻縮放圖片（保持圖片長寬比），使：
  - 圖片的寬度和高度至少有一個維度等於或大於視圖的對應維度（減去內邊距）
  - 縮放後的圖片至少有一個維度與視圖的對應維度（減去內邊距）相等

- `contain`: 均勻縮放圖片（保持圖片長寬比），使圖片的寬度和高度均等於或小於視圖的對應維度（減去內邊距）。

- `stretch`: 獨立縮放寬度和高度，這可能會改變原始圖片的長寬比。

- `repeat`: 重複圖片以填滿視圖框架。圖片將保持其尺寸和長寬比，除非它比視圖大，在這種情況下它將被均勻縮小以適應視圖。

- `center`: 在視圖中沿兩個維度居中圖片。如果圖片比視圖大，則均勻縮小以適應視圖。

| Type                                                              | Default   |
| ----------------------------------------------------------------- | --------- |
| enum(`'cover'`, `'contain'`, `'stretch'`, `'repeat'`, `'center'`) | `'cover'` |

---

### `source`

圖片來源（可以是遠程 URL 或本地文件資源）。

此屬性也可以包含多個遠程 URL，並指定它們的寬度和高度，以及可能的縮放比例/其他 URI 參數。原生端將根據圖片容器的測量尺寸選擇最佳的 `uri` 來顯示。可以添加 `cache` 屬性來控制網絡請求如何與本地緩存交互。（更多信息請參見[圖片的緩存控制（僅限 iOS）](images#cache-control-ios-only)）。

The currently supported formats are `png`, `jpg`, `jpeg`, `bmp`, `gif`, `webp`, `psd` (iOS only). In addition, iOS supports several RAW image formats. Refer to Apple's documentation for the current list of supported camera models (for iOS 12, see https://support.apple.com/en-ca/HT208967).

| Type                             |
| -------------------------------- |
| [ImageSource](image#imagesource) |

---

### `src`

一個字串，代表圖片的遠程 URL。此屬性優先於 `source` 屬性。

**範例:** `src={'https://reactnative.dev/img/tiny_logo.png'}`

| Type   |
| ------ |
| string |

---

### `srcSet`

一個字串，代表以逗號分隔的潛在候選圖片來源列表。每個圖片來源包含一個圖片的 URL 和一個像素密度描述符。如果未指定描述符，則默認為 `1x` 描述符。

如果 `srcSet` 不包含 `1x` 描述符，則使用 `src` 中的值作為 `1x` 描述符的圖片來源（如果提供）。

此屬性優先於 `src` 和 `source` 屬性。

**範例:** `srcSet={'https://reactnative.dev/img/tiny_logo.png 1x, https://reactnative.dev/img/header_logo.svg 2x'}`

| Type   |
| ------ |
| string |

---

### `style`

| Type                                                                                                                                                 |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Image Style Props](image-style-props#props), [Layout Props](layout-props#props), [Shadow Props](shadow-props#props), [Transforms](transforms#props) |

---

### `testID`

用於 UI 自動化測試腳本的唯一標識符。

| Type   |
| ------ |
| string |

---

### `tintColor`

將所有非透明像素的顏色更改為 `tintColor`。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `width`

圖片元件的寬度。

| Type   |
| ------ |
| number |

## 方法

### `abortPrefetch()` <div class="label android">Android</div>

```tsx
static abortPrefetch(requestId: number);
```

中止預取請求。

**參數：**

| Name                                                       | Type   | Description                             |
| ---------------------------------------------------------- | ------ | --------------------------------------- |
| requestId <div class="label basic required">Required</div> | number | Request id as returned by `prefetch()`. |

---

### `getSize()`

```tsx
static getSize(
  uri: string,
  success: (width: number, height: number) => void,
  failure?: (error: any) => void,
): any;
```

在顯示圖片之前獲取其寬度和高度（以像素為單位）。如果無法找到圖片或下載失敗，此方法可能會失敗。

為了獲取圖片的尺寸，可能需要先加載或下載圖片，之後圖片會被緩存。這意味著原則上你可以使用此方法來預加載圖片，但此方法並非為此目的而優化，未來可能會以不完全加載/下載圖片數據的方式實現。預加載圖片的正式支持方法將作為單獨的API提供。

**參數：**

| <div className="wideColumn">Name</div>                   | Type     | Description                                                                                          |
| -------------------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------- |
| uri <div class="label basic required">Required</div>     | string   | The location of the image.                                                                           |
| success <div class="label basic required">Required</div> | function | The function that will be called if the image was successfully found and width and height retrieved. |
| failure                                                  | function | The function that will be called if there was an error, such as failing to retrieve the image.       |

---

### `getSizeWithHeaders()`

```tsx
static getSizeWithHeaders(
  uri: string,
  headers: {[index: string]: string},
  success: (width: number, height: number) => void,
  failure?: (error: any) => void,
): any;
```

在顯示圖片之前獲取其寬度和高度（以像素為單位），並能夠提供請求的標頭。如果無法找到圖片或下載失敗，此方法可能會失敗。此方法也不適用於靜態圖片資源。

為了獲取圖片的尺寸，可能需要先加載或下載圖片，之後圖片會被緩存。這意味著原則上你可以使用此方法來預加載圖片，但此方法並非為此目的而優化，未來可能會以不完全加載/下載圖片數據的方式實現。預加載圖片的正式支持方法將作為單獨的API提供。

**參數：**

| <div className="wideColumn">Name</div>                   | Type     | Description                                                                                          |
| -------------------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------- |
| uri <div class="label basic required">Required</div>     | string   | The location of the image.                                                                           |
| headers <div class="label basic required">Required</div> | object   | The headers for the request.                                                                         |
| success <div class="label basic required">Required</div> | function | The function that will be called if the image was successfully found and width and height retrieved. |
| failure                                                  | function | The function that will be called if there was an error, such as failing to retrieve the image.       |

---

### `prefetch()`

```tsx
await Image.prefetch(url);
```

預取遠程圖片以供後續使用，將其下載到磁盤緩存中。返回一個解析為布爾值的Promise。

**參數：**

| Name                                                 | Type                                              | Description                                            |
| ---------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------ |
| url <div class="label basic required">Required</div> | string                                            | The remote location of the image.                      |
| callback                                             | function <div class="label android">Android</div> | The function that will be called with the `requestId`. |

---

### `queryCache()`

```tsx
static queryCache(
  urls: string[],
): Promise<Record<string, 'memory' | 'disk' | 'disk/memory'>>;
```

執行緩存查詢。返回一個解析為從URL到緩存狀態（如"disk"、"memory"或"disk/memory"）的映射的Promise。如果請求的URL不在映射中，則表示其不在緩存中。

**參數：**

| Name                                                  | Type  | Description                                |
| ----------------------------------------------------- | ----- | ------------------------------------------ |
| urls <div class="label basic required">Required</div> | array | List of image URLs to check the cache for. |

---

### `resolveAssetSource()`

```tsx
static resolveAssetSource(source: ImageSourcePropType): {
  height: number;
  width: number;
  scale: number;
  uri: string;
};
```

將資源引用解析為一個具有`uri`、`scale`、`width`和`height`屬性的對象。

**參數：**

| <div className="wideColumn">Name</div>                  | Type                                     | Description                                                                  |
| ------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------- |
| source <div class="label basic required">Required</div> | [ImageSource](image#imagesource), number | A number (opaque type returned by `require('./foo.png')`) or an ImageSource. |

## 類型定義

### ImageCacheEnum <div class="label ios">iOS</div>

用於設置緩存處理或策略的枚舉，適用於可能被緩存的響應。

| Type                                                               | Default     |
| ------------------------------------------------------------------ | ----------- |
| enum(`'default'`, `'reload'`, `'force-cache'`, `'only-if-cached'`) | `'default'` |

- `default`：使用原生平台的預設策略。
- `reload`：URL 的資料將從原始來源載入。不應使用任何現有的快取資料來滿足 URL 載入請求。
- `force-cache`：無論快取資料的新舊或過期日期為何，都將使用現有的快取資料來滿足請求。如果快取中沒有對應請求的資料，則從原始來源載入資料。
- `only-if-cached`：無論快取資料的新舊或過期日期為何，都將使用現有的快取資料來滿足請求。如果快取中沒有對應 URL 載入請求的資料，則不會嘗試從原始來源載入資料，且該載入將被視為失敗。

### ImageLoadEvent

在 `onLoad` 回調中返回的物件。

| Type   |
| ------ |
| object |

**屬性：**

| Name   | Type   | Description                         |
| ------ | ------ | ----------------------------------- |
| source | object | The [source object](#source-object) |

#### 來源物件

**屬性：**

| Name   | Type   | Description                                                  |
| ------ | ------ | ------------------------------------------------------------ |
| width  | number | The width of loaded image.                                   |
| height | number | The height of loaded image.                                  |
| uri    | string | A string representing the resource identifier for the image. |

### ImageSource

| Type                             |
| -------------------------------- |
| object, array of objects, number |

**屬性（如果傳遞物件或物件陣列）：**

| <div className="wideColumn">Name</div> | Type                                       | Description                                                                                                                                                                          |
| -------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| uri                                    | string                                     | A string representing the resource identifier for the image, which could be an http address, a local file path, or the name of a static image resource.                              |
| width                                  | number                                     | Can be specified if known at build time, in which case the value will be used to set the default `<Image/>` component dimension.                                                     |
| height                                 | number                                     | Can be specified if known at build time, in which case the value will be used to set the default `<Image/>` component dimension.                                                     |
| scale                                  | number                                     | Used to indicate the scale factor of the image. Defaults to `1.0` if unspecified, meaning that one image pixel equates to one display point / DIP.                                   |
| bundle<div class="label ios">iOS</div> | string                                     | The iOS asset bundle which the image is included in. This will default to `[NSBundle mainBundle]` if not set.                                                                        |
| method                                 | string                                     | The HTTP Method to use. Defaults to `'GET'` if not specified.                                                                                                                        |
| headers                                | object                                     | An object representing the HTTP headers to send along with the request for a remote image.                                                                                           |
| body                                   | string                                     | The HTTP body to send with the request. This must be a valid UTF-8 string, and will be sent exactly as specified, with no additional encoding (e.g. URL-escaping or base64) applied. |
| cache<div class="label ios">iOS</div>  | [ImageCacheEnum](image#imagecacheenum-ios) | Determines how the requests handles potentially cached responses.                                                                                                                    |

**如果傳遞數字：**

- `number` - 由類似 `require('./image.jpg')` 返回的不透明類型。