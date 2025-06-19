---
id: images
title: Images
---

## 靜態圖片資源

React Native 提供了一種統一的方式來管理 Android 和 iOS 應用中的圖片及其他媒體資源。若要添加靜態圖片到您的應用，請將其放置在源代碼樹中的某處，並像這樣引用它：

```tsx
<Image source={require('./my-icon.png')} />
```

圖片名稱的解析方式與 JS 模組相同。在上面的例子中，打包工具會在與引用它的組件相同的文件夾中查找 `my-icon.png`。

您可以使用 `@2x` 和 `@3x` 後綴來為不同的屏幕密度提供圖片。如果您有以下文件結構：

```
.
├── button.js
└── img
    ├── check.png
    ├── check@2x.png
    └── check@3x.png
```

...而 `button.js` 代碼中包含：

```tsx
<Image source={require('./img/check.png')} />
```

...打包工具將根據設備的屏幕密度打包並提供相應的圖片。例如，`check@2x.png` 將在 iPhone 7 上使用，而 `check@3x.png` 將在 iPhone 7 Plus 或 Nexus 5 上使用。如果沒有與屏幕密度匹配的圖片，則會選擇最接近的最佳選項。

在 Windows 上，如果您向項目中添加了新圖片，可能需要重新啟動打包工具。

以下是您獲得的一些好處：

1. Android 和 iOS 使用相同的系統。
2. 圖片與您的 JavaScript 代碼位於同一文件夾中。組件是自包含的。
3. 沒有全局命名空間，即您不必擔心名稱衝突。
4. 只有實際使用的圖片才會被打包到您的應用中。
5. 添加和更改圖片不需要重新編譯應用，您可以像平常一樣刷新模擬器。
6. 打包工具知道圖片的尺寸，無需在代碼中重複。
7. 圖片可以通過 [npm](https://www.npmjs.com/) 包分發。

為了使這一切正常工作，`require` 中的圖片名稱必須是靜態已知的。

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

請注意，以這種方式引入的圖片源包含圖片的尺寸（寬度、高度）信息。如果您需要動態縮放圖片（例如通過 flex），您可能需要在樣式屬性中手動設置 `{width: undefined, height: undefined}`。

## 靜態非圖片資源

上述的 `require` 語法也可以用於靜態包含音頻、視頻或文檔文件到您的項目中。支持大多數常見文件類型，包括 `.mp3`、`.wav`、`.mp4`、`.mov`、`.html` 和 `.pdf`。完整列表請參見 [打包工具默認設置](https://github.com/facebook/metro/blob/master/packages/metro-config/src/defaults/defaults.js#L14-L44)。

You can add support for other types by adding an [`assetExts` resolver option](https://metrobundler.dev/docs/configuration#resolver-options) in your [Metro configuration](https://metrobundler.dev/docs/configuration).

需要注意的是，視頻必須使用絕對定位而不是 `flexGrow`，因為目前不會傳遞非圖片資源的尺寸信息。對於直接鏈接到 Xcode 或 Android 的 Assets 文件夾中的視頻，不會出現此限制。

## 來自混合應用資源的圖片

如果您正在構建一個混合應用（部分 UI 使用 React Native，部分 UI 使用平台代碼），您仍然可以使用已經打包到應用中的圖片。

對於通過 Xcode 資源目錄或 Android drawable 文件夾包含的圖片，請使用不帶擴展名的圖片名稱：

```tsx
<Image
  source={{uri: 'app_icon'}}
  style={{width: 40, height: 40}}
/>
```

對於 Android assets 文件夾中的圖片，請使用 `asset:/` 方案：

```tsx
<Image
  source={{uri: 'asset:/app_icon.png'}}
  style={{width: 40, height: 40}}
/>
```

這些方法不提供安全檢查。您需要確保這些圖片在應用中可用。此外，您還需要手動指定圖片的尺寸。

## 網絡圖片

您應用中顯示的許多圖片在編譯時並不可用，或者您可能希望動態加載部分圖片以減少二進位檔案大小。與靜態資源不同，_您需要手動指定圖片的尺寸_。強烈建議同時使用 https 以滿足 iOS 上 [App Transport Security](publishing-to-app-store.md#1-enable-app-transport-security) 的要求。

```tsx
// GOOD
<Image source={{uri: 'https://reactjs.org/logo-og.png'}}
       style={{width: 400, height: 400}} />

// BAD
<Image source={{uri: 'https://reactjs.org/logo-og.png'}} />
```

### 圖片的網路請求

若您想在圖片請求中設定 HTTP 方法、標頭或主體等參數，可以透過在來源物件上定義這些屬性來實現：

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

## URI 資料圖片

有時您可能從 REST API 呼叫中取得編碼的圖片資料。您可以使用 `'data:'` URI 方案來顯示這些圖片。與網路資源相同，_您需要手動指定圖片的尺寸_。

:::info
此方法僅建議用於非常小且動態的圖片，例如資料庫清單中的圖示。
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

### 快取控制

某些情況下，您可能只想顯示本地快取中已有的圖片（例如先顯示低解析度佔位圖，直到高解析度圖片可用）。其他情況下，您可能不介意圖片是否過期，願意顯示過期圖片以節省頻寬。`cache` 來源屬性讓您能控制網路層如何與快取互動。

- `default`: 使用原生平台的預設策略。
- `reload`: 從原始來源載入 URL 資料，不應使用任何現有快取資料來滿足 URL 載入請求。
- `force-cache`: 無論快取資料的時效性或過期日期為何，都會使用現有快取資料來滿足請求。若快取中沒有對應請求的資料，則從原始來源載入。
- `only-if-cached`: 僅使用現有快取資料滿足請求，無論其時效性或過期日期。若快取中沒有對應 URL 載入請求的資料，則不會嘗試從原始來源載入，並視為載入失敗。

```tsx
<Image
  source={{
    uri: 'https://reactjs.org/logo-og.png',
    cache: 'only-if-cached',
  }}
  style={{width: 400, height: 400}}
/>
```

## 本地檔案系統圖片

請參閱 [CameraRoll](https://github.com/react-native-community/react-native-cameraroll) 了解如何使用位於 `Images.xcassets` 外部的本地資源範例。

### 最佳相機膠卷圖片

iOS 會在相機膠卷中為同一張圖片保存多種尺寸，基於效能考量選擇最接近的尺寸至關重要。當顯示 200x200 縮圖時，您不會想使用完整品質的 3264x2448 圖片作為來源。若有完全匹配的尺寸，React Native 會自動選擇；否則會使用至少大 50% 的第一個可用尺寸，以避免從相近尺寸調整時產生模糊效果。這些都是預設行為，您無需編寫繁瑣（且容易出錯）的代碼來實現。

## 為何不自動調整所有內容的尺寸？

_在瀏覽器中_，若您未指定圖片尺寸，瀏覽器會先渲染 0x0 元素，下載圖片後再根據正確尺寸重新渲染。這種行為的最大問題在於圖片載入時會導致 UI 四處跳動，造成極差的用戶體驗。這被稱為 [累積版面偏移 (CLS)](https://web.dev/cls/)。

_In React Native_ this behavior is intentionally not implemented. It is more work for the developer to know the dimensions (or aspect ratio) of the remote image in advance, but we believe that it leads to a better user experience. Static images loaded from the app bundle via the `require('./my-icon.png')` syntax _can be automatically sized_ because their dimensions are available immediately at the time of mounting.

例如，`require('./my-icon.png')` 的結果可能是：

```tsx
{"__packager_asset":true,"uri":"my-icon.png","width":591,"height":573}
```

## 來源作為物件

在 React Native 中，一個有趣的設計決策是將 `src` 屬性命名為 `source`，並且不接受字串，而是接受一個帶有 `uri` 屬性的物件。

```tsx
<Image source={{uri: 'something.jpg'}} />
```

在基礎架構方面，這樣做的原因是它允許我們為這個物件附加元數據。例如，如果你使用 `require('./my-icon.png')`，那麼我們會添加有關其實際位置和大小的信息（請不要依賴這一點，未來可能會改變！）。這也是一種未來證明，例如我們可能希望在某個時候支持精靈圖（sprites），而不是輸出 `{uri: ...}`，我們可以輸出 `{uri: ..., crop: {left: 10, top: 50, width: 20, height: 40}}`，並在所有現有的調用點透明地支持精靈圖。

在用戶端，這讓你可以為物件添加有用的屬性，例如圖像的尺寸，以便計算它將顯示的大小。你可以自由地使用它作為你的數據結構來存儲更多關於圖像的信息。

## 通過嵌套實現背景圖像

熟悉網頁開發的開發者經常會提出的一個常見功能請求是 `background-image`。為了處理這種用例，你可以使用 `<ImageBackground>` 組件，它擁有與 `<Image>` 相同的屬性，並且可以在其上添加任何你想要疊加的內容。

在某些情況下，你可能不想使用 `<ImageBackground>`，因為其實現較為基礎。請參考 `<ImageBackground>` 的[文檔](imagebackground.md)以獲取更多見解，並在需要時創建你自己的自定義組件。

```tsx
return (
  <ImageBackground source={...} style={{width: '100%', height: '100%'}}>
    <Text>Inside</Text>
  </ImageBackground>
);
```

請注意，你必須指定一些寬度和高度的樣式屬性。

## iOS 邊框半徑樣式

請注意，以下特定於角落的邊框半徑樣式屬性可能會被 iOS 的圖像組件忽略：

- `borderTopLeftRadius`
- `borderTopRightRadius`
- `borderBottomLeftRadius`
- `borderBottomRightRadius`

## 線程外解碼

圖像解碼可能需要超過一幀的時間。這是網頁上幀率下降的主要原因之一，因為解碼是在主線程中完成的。在 React Native 中，圖像解碼是在不同的線程中完成的。實際上，你已經需要處理圖像尚未下載的情況，因此在解碼過程中多顯示幾幀的佔位圖不需要任何代碼更改。

## 配置 iOS 圖像緩存限制

在 iOS 上，我們提供了一個 API 來覆蓋 React Native 的默認圖像緩存限制。這應該從你的原生 AppDelegate 代碼中調用（例如在 `didFinishLaunchingWithOptions` 中）。

```objectivec
RCTSetImageCacheLimits(4*1024*1024, 200*1024*1024);
```

**參數：**

| Name           | Type   | Required | Description             |
| -------------- | ------ | -------- | ----------------------- |
| imageSizeLimit | number | Yes      | Image cache size limit. |
| totalCostLimit | number | Yes      | Total cache cost limit. |

在上面的代碼示例中，圖像大小限制設置為 4 MB，總成本限制設置為 200 MB。