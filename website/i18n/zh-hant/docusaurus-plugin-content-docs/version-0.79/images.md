---
id: images
title: Images
---

## 靜態圖片資源

React Native 提供了一種統一的方式來管理 Android 和 iOS 應用中的圖片及其他媒體資源。若要添加靜態圖片到您的應用中，請將其放置在源代碼樹的某處，並像這樣引用它：

```tsx
<Image source={require('./my-icon.png')} />
```

圖片名稱的解析方式與 JS 模組相同。在上面的例子中，打包器會在需要它的組件所在的文件夾中查找 `my-icon.png`。

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

...打包器將根據設備的屏幕密度打包並提供相應的圖片。例如，`check@2x.png` 將在 iPhone 7 上使用，而 `check@3x.png` 將在 iPhone 7 Plus 或 Nexus 5 上使用。如果沒有與屏幕密度匹配的圖片，則會選擇最接近的最佳選項。

在 Windows 上，如果您向項目中添加了新圖片，可能需要重新啟動打包器。

以下是您獲得的一些好處：

1. Android 和 iOS 使用相同的系統。
2. 圖片與您的 JavaScript 代碼位於同一文件夾中。組件是自包含的。
3. 沒有全局命名空間，即您不必擔心名稱衝突。
4. 只有實際使用的圖片才會被打包到您的應用中。
5. 添加和更改圖片不需要重新編譯應用，您可以像平常一樣刷新模擬器。
6. 打包器知道圖片的尺寸，無需在代碼中重複。
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

請注意，以這種方式所需的圖片源包括圖片的尺寸（寬度、高度）信息。如果您需要動態縮放圖片（例如通過 flex），您可能需要在樣式屬性中手動設置 `{width: undefined, height: undefined}`。

## 靜態非圖片資源

上述的 `require` 語法也可以用於靜態包含音頻、視頻或文檔文件到您的項目中。支持大多數常見文件類型，包括 `.mp3`、`.wav`、`.mp4`、`.mov`、`.html` 和 `.pdf`。完整列表請參見 [打包器默認設置](https://github.com/facebook/metro/blob/master/packages/metro-config/src/defaults/defaults.js#L14-L44)。

You can add support for other types by adding an [`assetExts` resolver option](https://metrobundler.dev/docs/configuration#resolver-options) in your [Metro configuration](https://metrobundler.dev/docs/configuration).

需要注意的是，視頻必須使用絕對定位而不是 `flexGrow`，因為目前沒有為非圖片資源傳遞尺寸信息。對於直接鏈接到 Xcode 或 Android 的 Assets 文件夾中的視頻，不會出現此限制。

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

您應用中顯示的許多圖片在編譯時並不可用，或者您可能希望動態加載一些圖片以減少二進制文件大小。與靜態資源不同，_您需要手動指定圖片的尺寸_。強烈建議您同時使用 https，以滿足 iOS 上 [App Transport Security](publishing-to-app-store.md#1-enable-app-transport-security) 的要求。

```tsx
// GOOD
<Image source={{uri: 'https://reactjs.org/logo-og.png'}}
       style={{width: 400, height: 400}} />

// BAD
<Image source={{uri: 'https://reactjs.org/logo-og.png'}} />
```

### 圖片的網絡請求

如果您希望在圖片請求中設置 HTTP 動詞、標頭或請求體，可以通過在源對象上定義這些屬性來實現：

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

有時，您可能從 REST API 調用中獲取編碼的圖片數據。您可以使用 `'data:'` URI 方案來使用這些圖片。與網絡資源一樣，_您需要手動指定圖片的尺寸_。

:::info
建議僅用於非常小且動態的圖片，例如來自數據庫的列表中的圖標。
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

### 緩存控制

在某些情況下，您可能只希望在本地緩存中已有圖片時顯示它，例如在更高分辨率的圖片可用之前顯示一個低分辨率的佔位圖。在其他情況下，您不介意圖片是否過時，並願意顯示過時的圖片以節省帶寬。`cache` 源屬性讓您可以控制網絡層如何與緩存交互。

- `default`：使用原生平台的默認策略。
- `reload`：從原始源加載 URL 的數據。不應使用現有的緩存數據來滿足 URL 加載請求。
- `force-cache`：無論其年齡或過期日期如何，都將使用現有的緩存數據來滿足請求。如果緩存中沒有與請求相對應的現有數據，則從原始源加載數據。
- `only-if-cached`：無論其年齡或過期日期如何，都將使用現有的緩存數據來滿足請求。如果緩存中沒有與 URL 加載請求相對應的現有數據，則不會嘗試從原始源加載數據，並且加載被視為失敗。

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

請參閱 [CameraRoll](https://github.com/react-native-community/react-native-cameraroll) 以了解如何使用位於 `Images.xcassets` 之外的本地資源的示例。

### Drawable 資源

Android supports loading [drawable resources](https://developer.android.com/guide/topics/resources/drawable-resource) via the `xml` file type. This means you can use [vector drawables](https://developer.android.com/develop/ui/views/graphics/vector-drawable-resources) for rendering icons or [shape drawables](https://developer.android.com/guide/topics/resources/drawable-resource#Shape) for, well, drawing shapes! You can import and use these resource types the same as any other [static resource](#static-image-resources) or [hybrid resource](#images-from-hybrid-apps-resources). You have to specify image dimensions manually.

對於與您的 JS 代碼位於同一位置的靜態 drawables，使用 `require` 或 `import` 語法（兩者效果相同）：

```tsx
<Image
  source={require('./img/my_icon.xml')}
  style={{width: 40, height: 40}}
/>
```

對於包含在 Android drawable 文件夾（即 `res/drawable`）中的 drawables，使用不帶擴展名的資源名稱：

```tsx
<Image
  source={{uri: 'my_icon'}}
  style={{width: 40, height: 40}}
/>
```

drawable 資源與其他圖片類型的一個關鍵區別是，必須在 Android 應用程序的編譯時引用該資源，因為 Android 需要運行 [Android Asset Packaging Tool (AAPT)](https://developer.android.com/tools/aapt2) 來打包資源。AAPT 創建的二進制 XML 文件格式無法通過 Metro 從網絡加載。如果您更改資源的目錄或名稱，則每次都需要重新構建 Android 應用程序。

#### 創建 XML drawable 資源

Android 在其[可繪製資源指南](https://developer.android.com/guide/topics/resources/drawable-resource)中提供了每種支援的可繪製資源類型的完整文件，並附有原始 XML 檔案的範例。你可以使用 Android Studio 的工具，如[向量資源工作室](https://developer.android.com/studio/write/vector-asset-studio)，從可縮放向量圖形 (SVG) 和 Adobe Photoshop 文件 (PSD) 建立向量可繪製資源。

:::info
如果你希望將 XML 檔案視為靜態圖片資源（即使用 `import` 或 `require` 語句），應盡量避免在建立的 XML 檔案中引用其他資源。如果你想利用對其他可繪製資源或屬性的引用，如[顏色狀態列表](https://developer.android.com/guide/topics/resources/color-list-resource)或[尺寸資源](https://developer.android.com/guide/topics/resources/more-resources#Dimension)，則應將可繪製資源作為[混合資源](#images-from-hybrid-apps-resources)包含，並按名稱導入。
:::

### 最佳相機膠卷圖片

iOS 會在相機膠卷中為同一張圖片保存多種尺寸，出於性能考慮，選擇最接近的尺寸非常重要。當顯示 200x200 的縮略圖時，你不會希望使用全品質的 3264x2448 圖片作為來源。如果有完全匹配的尺寸，React Native 會選擇它，否則會使用至少大 50% 的第一個尺寸，以避免從接近尺寸調整大小時出現模糊。這一切都是默認完成的，因此你無需擔心編寫繁瑣（且容易出錯）的代碼來自行處理。

## 為什麼不自動調整所有內容的大小？

_在瀏覽器中_，如果你不給圖片指定大小，瀏覽器會渲染一個 0x0 的元素，下載圖片，然後根據正確的尺寸渲染圖片。這種行為的最大問題是，隨著圖片的加載，你的 UI 會到處跳動，這會導致非常糟糕的用戶體驗。這被稱為[累積佈局偏移](https://web.dev/cls/)。

_In React Native_ this behavior is intentionally not implemented. It is more work for the developer to know the dimensions (or aspect ratio) of the remote image in advance, but we believe that it leads to a better user experience. Static images loaded from the app bundle via the `require('./my-icon.png')` syntax _can be automatically sized_ because their dimensions are available immediately at the time of mounting.

例如，`require('./my-icon.png')` 的結果可能是：

```tsx
{"__packager_asset":true,"uri":"my-icon.png","width":591,"height":573}
```

## 來源作為物件

在 React Native 中，一個有趣的決定是 `src` 屬性被命名為 `source`，並且不接受字串，而是接受一個具有 `uri` 屬性的物件。

```tsx
<Image source={{uri: 'something.jpg'}} />
```

在基礎設施方面，這樣做的原因是它允許我們將元數據附加到這個物件上。例如，如果你使用 `require('./my-icon.png')`，那麼我們會添加有關其實際位置和大小的信息（不要依賴這個事實，它可能在未來改變！）。這也是為了未來兼容性，例如我們可能希望在某個時候支援精靈圖，而不是輸出 `{uri: ...}`，我們可以輸出 `{uri: ..., crop: {left: 10, top: 50, width: 20, height: 40}}`，並透明地支援所有現有呼叫站點的精靈圖。

在用戶方面，這讓你可以用有用的屬性（如圖片的尺寸）來註解物件，以計算它將顯示的大小。可以自由地將其用作存儲有關圖片的更多信息的數據結構。

## 通過嵌套實現背景圖片

熟悉網頁開發的開發者常見的一個功能請求是 `background-image`。為了處理這種用例，你可以使用 `<ImageBackground>` 組件，它具有與 `<Image>` 相同的屬性，並可以在其上添加任何你希望疊加的子元素。

在某些情況下，您可能不想使用 `<ImageBackground>`，因為其實作較為基礎。如需更多資訊，請參考 `<ImageBackground>` 的[文件](imagebackground.md)，並在需要時建立您自己的自訂元件。

```tsx
return (
  <ImageBackground source={...} style={{width: '100%', height: '100%'}}>
    <Text>Inside</Text>
  </ImageBackground>
);
```

請注意，您必須指定某些寬度和高度的樣式屬性。

## iOS 邊框半徑樣式

請注意，iOS 的圖片元件可能會忽略以下特定角落的邊框半徑樣式屬性：

- `borderTopLeftRadius`
- `borderTopRightRadius`
- `borderBottomLeftRadius`
- `borderBottomRightRadius`

## 離線程解碼

圖片解碼可能需要超過一幀的時間。這是網頁上幀率下降的主要原因之一，因為解碼是在主線程中進行的。在 React Native 中，圖片解碼是在不同的線程中進行的。實際上，您已經需要處理圖片尚未下載完成的情況，因此在解碼過程中多顯示幾幀的佔位圖並不需要任何程式碼變更。

## 設定 iOS 圖片快取限制

在 iOS 上，我們提供了一個 API 來覆寫 React Native 的預設圖片快取限制。這應該從您的原生 AppDelegate 程式碼中呼叫（例如在 `didFinishLaunchingWithOptions` 中）。

```objectivec
RCTSetImageCacheLimits(4*1024*1024, 200*1024*1024);
```

**參數：**

| Name           | Type   | Required | Description             |
| -------------- | ------ | -------- | ----------------------- |
| imageSizeLimit | number | Yes      | Image cache size limit. |
| totalCostLimit | number | Yes      | Total cache cost limit. |

在上述程式碼範例中，圖片大小限制設定為 4 MB，總成本限制設定為 200 MB。