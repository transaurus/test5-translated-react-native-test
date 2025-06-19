---
id: images
title: Images
---

## 靜態圖片資源

React Native 提供統一的方式來管理 Android 和 iOS 應用中的圖片及其他媒體資源。若要添加靜態圖片至您的應用，請將其置於原始碼樹中的某處，並如下引用：

```tsx
<Image source={require('./my-icon.png')} />
```

圖片名稱的解析方式與 JS 模組相同。在上例中，打包工具會尋找與引用該圖片的組件同目錄下的 `my-icon.png`。

您可使用 `@2x` 和 `@3x` 後綴來提供適用不同螢幕密度的圖片。若您的檔案結構如下：

```
.
├── button.js
└── img
    ├── check.png
    ├── check@2x.png
    └── check@3x.png
```

...且 `button.js` 程式碼中包含：

```tsx
<Image source={require('./img/check.png')} />
```

...打包工具將根據裝置螢幕密度打包並提供對應圖片。例如，iPhone 7 會使用 `check@2x.png`，而 iPhone 7 Plus 或 Nexus 5 則會使用 `check@3x.png`。若無完全匹配的圖片，則會選擇最接近的選項。

在 Windows 上，若您向專案添加了新圖片，可能需要重啟打包工具。

您將獲得以下優勢：

1. Android 和 iOS 使用相同系統。
2. 圖片與 JavaScript 程式碼位於同一目錄，組件自包含。
3. 無全域命名空間，無需擔心名稱衝突。
4. 僅實際使用的圖片會被打包進應用。
5. 添加或變更圖片無需重新編譯應用，可如常刷新模擬器。
6. 打包工具知曉圖片尺寸，無需在程式碼中重複指定。
7. 圖片可透過 [npm](https://www.npmjs.com/) 套件分發。

為使此機制生效，`require` 中的圖片名稱需為靜態已知。

```tsx
// GOOD
<Image source={require('./my-icon.png')} />;

// BAD
const icon = this.props.active
  ? 'my-icon-active'
  : 'my-icon-inactive';
<Image source={require('./' + icon + '.png')} />;

// GOOD
const icon = this.props.active
  ? require('./my-icon-active.png')
  : require('./my-icon-inactive.png');
<Image source={icon} />;
```

請注意，以此方式引入的圖片源包含尺寸（寬度、高度）資訊。若需動態縮放圖片（例如透過 flex），您可能需手動在樣式屬性中設定 `{width: undefined, height: undefined}`。

## 靜態非圖片資源

上述 `require` 語法亦可用於靜態引入音訊、視訊或文件檔案至專案中。支援大多數常見檔案類型，包括 `.mp3`、`.wav`、`.mp4`、`.mov`、`.html` 和 `.pdf`。完整清單請參閱 [打包工具預設值](https://github.com/facebook/metro/blob/master/packages/metro-config/src/defaults/defaults.js#L14-L44)。

You can add support for other types by adding an [`assetExts` resolver option](https://metrobundler.dev/docs/configuration#resolver-options) in your [Metro configuration](https://metrobundler.dev/docs/configuration).

需注意的是，視訊必須使用絕對定位而非 `flexGrow`，因目前非圖片資源未傳遞尺寸資訊。此限制不適用於直接連結至 Xcode 或 Android Assets 資料夾中的視訊。

## 混合應用程式資源中的圖片

若您正在建構混合應用（部分 UI 使用 React Native，部分使用平台原生程式碼），仍可使用已打包至應用中的圖片。

對於透過 Xcode 資產目錄或 Android drawable 資料夾引入的圖片，請使用不含副檔名的圖片名稱：

```tsx
<Image
  source={{uri: 'app_icon'}}
  style={{width: 40, height: 40}}
/>
```

對於 Android assets 資料夾中的圖片，請使用 `asset:/` 方案：

```tsx
<Image
  source={{uri: 'asset:/app_icon.png'}}
  style={{width: 40, height: 40}}
/>
```

這些方法不提供安全性檢查。您需自行確保這些圖片在應用中可用，且需手動指定圖片尺寸。

## 網路圖片

您應用中顯示的許多圖片在編譯時並不可用，或者您可能希望動態加載某些圖片以減少二進制文件大小。與靜態資源不同，_您需要手動指定圖片的尺寸_。強烈建議您同時使用 https 以滿足 iOS 上 [App Transport Security](publishing-to-app-store.md#1-enable-app-transport-security) 的要求。

```tsx
// GOOD
<Image source={{uri: 'https://reactjs.org/logo-og.png'}}
       style={{width: 400, height: 400}} />

// BAD
<Image source={{uri: 'https://reactjs.org/logo-og.png'}} />
```

### 圖片的網路請求

如果您希望在圖片請求中設置 HTTP 方法、標頭或主體等內容，可以通過在源對象上定義這些屬性來實現：

```tsx
<Image
  source={{
    uri: 'https://reactjs.org/logo-og.png',
    method: 'POST',
    headers: {
      Pragma: 'no-cache',
    },
    body: 'Your Body goes here',
  }}
  style={{width: 400, height: 400}}
/>
```

## URI 數據圖片

有時，您可能會從 REST API 調用中獲取編碼的圖片數據。您可以使用 `'data:'` URI 方案來使用這些圖片。與網路資源一樣，_您需要手動指定圖片的尺寸_。

:::info
這僅適用於非常小且動態的圖片，例如來自數據庫的列表中的圖標。
:::

```tsx
// include at least width and height!
<Image
  style={{
    width: 51,
    height: 51,
    resizeMode: 'contain',
  }}
  source={{
    uri: 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADMAAAAzCAYAAAA6oTAqAAAAEXRFWHRTb2Z0d2FyZQBwbmdjcnVzaEB1SfMAAABQSURBVGje7dSxCQBACARB+2/ab8BEeQNhFi6WSYzYLYudDQYGBgYGBgYGBgYGBgYGBgZmcvDqYGBgmhivGQYGBgYGBgYGBgYGBgYGBgbmQw+P/eMrC5UTVAAAAABJRU5ErkJggg==',
  }}
/>
```

### 緩存控制（僅限 iOS）

在某些情況下，您可能只希望在本地緩存中已有圖片時顯示它，例如在更高分辨率的圖片可用之前顯示低分辨率的佔位圖。在其他情況下，您不關心圖片是否過時，並願意顯示過時的圖片以節省帶寬。`cache` 源屬性讓您可以控制網路層如何與緩存交互。

- `default`：使用原生平台的默認策略。
- `reload`：URL 的數據將從原始源加載。不應使用現有的緩存數據來滿足 URL 加載請求。
- `force-cache`：將使用現有的緩存數據來滿足請求，無論其年齡或過期日期如何。如果緩存中沒有與請求相對應的現有數據，則從原始源加載數據。
- `only-if-cached`：將使用現有的緩存數據來滿足請求，無論其年齡或過期日期如何。如果緩存中沒有與 URL 加載請求相對應的現有數據，則不會嘗試從原始源加載數據，並且加載被視為失敗。

```tsx
<Image
  source={{
    uri: 'https://reactjs.org/logo-og.png',
    cache: 'only-if-cached',
  }}
  style={{width: 400, height: 400}}
/>
```

## 本地文件系統圖片

請參閱 [CameraRoll](https://github.com/react-native-community/react-native-cameraroll) 以獲取使用 `Images.xcassets` 之外的本地資源的示例。

### 最佳相機膠卷圖片

iOS 會為相機膠卷中的同一張圖片保存多個尺寸，為了性能考慮，選擇最接近的尺寸非常重要。您不會希望使用全質量 3264x2448 的圖片作為顯示 200x200 縮略圖的源。如果有完全匹配的尺寸，React Native 會選擇它，否則它會使用至少大 50% 的第一個尺寸，以避免從接近的尺寸調整大小時出現模糊。所有這些都是默認完成的，因此您無需編寫繁瑣（且容易出錯）的代碼來自行實現。

## 為什麼不自動調整所有內容的大小？

_在瀏覽器中_，如果您不給圖片指定大小，瀏覽器將渲染一個 0x0 的元素，下載圖片，然後根據正確的尺寸渲染圖片。這種行為的主要問題是，您的 UI 會在圖片加載時四處跳動，這會導致非常糟糕的用戶體驗。這稱為 [累積佈局偏移](https://web.dev/cls/)。

_在 React Native 中_，這種行為是故意不實現的。開發人員需要提前知道遠程圖片的尺寸（或寬高比），這需要更多的工作，但我們相信這會帶來更好的用戶體驗。通過 `require('./my-icon.png')` 語法從應用程序包加載的靜態圖片 _可以自動調整大小_，因為它們的尺寸在掛載時立即可用。

例如，`require('./my-icon.png')` 的結果可能是：

```tsx
{"__packager_asset":true,"uri":"my-icon.png","width":591,"height":573}
```

## 以物件形式指定來源

在 React Native 中，一個有趣的設計決策是將 `src` 屬性命名為 `source`，並且不接受字串，而是接受一個帶有 `uri` 屬性的物件。

```tsx
<Image source={{uri: 'something.jpg'}} />
```

在基礎架構層面，這樣設計的原因是它允許我們為此物件附加元數據。例如，如果你使用 `require('./my-icon.png')`，我們會加入關於其實際位置和大小的資訊（請勿依賴此特性，未來可能會變更！）。這同時也是為了未來兼容性考量，例如我們可能希望支援精靈圖（sprites），此時輸出的不是 `{uri: ...}`，而是 `{uri: ..., crop: {left: 10, top: 50, width: 20, height: 40}}`，從而透明地支援所有現有呼叫點的圖像拼接。

對使用者而言，這讓你能夠為物件添加有用的屬性，例如圖像的尺寸以計算其顯示大小。你可以自由地將其作為資料結構來儲存關於圖像的更多資訊。

## 透過嵌套實現背景圖像

熟悉網頁開發的開發者常會提出 `background-image` 的功能需求。為了處理這類用例，你可以使用 `<ImageBackground>` 元件，它擁有與 `<Image>` 相同的屬性，並可在其上疊加任何子元件。

在某些情況下，你可能不想使用 `<ImageBackground>`，因為其實現較為基礎。請參考 `<ImageBackground>` 的[文件](imagebackground.md)以獲取更多見解，並在需要時創建自己的自訂元件。

```tsx
return (
  <ImageBackground source={...} style={{width: '100%', height: '100%'}}>
    <Text>Inside</Text>
  </ImageBackground>
);
```

請注意，你必須指定一些寬度和高度的樣式屬性。

## iOS 邊框半徑樣式

請注意，以下特定角落的邊框半徑樣式屬性可能會被 iOS 的圖像元件忽略：

- `borderTopLeftRadius`
- `borderTopRightRadius`
- `borderBottomLeftRadius`
- `borderBottomRightRadius`

## 線程外解碼

圖像解碼可能需要超過一幀的時間。這是網頁上幀率下降的主要原因之一，因為解碼是在主線程中進行的。在 React Native 中，圖像解碼是在不同的線程中完成的。實際上，你已經需要處理圖像尚未下載完成的情況，因此在解碼過程中多顯示幾幀的佔位圖並不需要任何程式碼變更。

## 配置 iOS 圖像快取限制

在 iOS 上，我們提供了一個 API 來覆寫 React Native 的預設圖像快取限制。這應該從你的原生 AppDelegate 程式碼中呼叫（例如在 `didFinishLaunchingWithOptions` 內）。

```objectivec
RCTSetImageCacheLimits(4*1024*1024, 200*1024*1024);
```

**參數：**

| Name           | Type   | Required | Description             |
| -------------- | ------ | -------- | ----------------------- |
| imageSizeLimit | number | Yes      | Image cache size limit. |
| totalCostLimit | number | Yes      | Total cache cost limit. |

在上述程式碼範例中，圖像大小限制設為 4 MB，總成本限制設為 200 MB。