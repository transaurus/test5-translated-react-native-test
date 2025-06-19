---
id: platform-specific-code
title: Platform Specific Code
---

在開發跨平台應用程式時，你會希望盡可能重複使用程式碼。但在某些情況下，可能需要針對不同平台撰寫不同的程式碼，例如你可能需要為 Android 和 iOS 分別實作視覺元件。

React Native 提供了兩種組織程式碼並按平台區分的方式：

- Using the [`Platform` module](platform-specific-code.md#platform-module).
- Using [platform-specific file extensions](platform-specific-code.md#platform-specific-extensions).

某些元件可能具有僅在單一平台有效的屬性。所有這類屬性都會標註 `@platform` 並在官方網站上顯示小徽章標記。

## Platform 模組

React Native 提供了一個能檢測應用程式運行平台的模組。你可以利用此檢測邏輯來實作平台特定程式碼。當只有元件的小部分需要區分平台時，建議使用此方式。

```tsx
import {Platform, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  height: Platform.OS === 'ios' ? 200 : 100,
});
```

`Platform.OS` 在 iOS 上會回傳 `ios`，在 Android 上則會回傳 `android`。

另提供 `Platform.select` 方法，當傳入一個鍵值為 `'ios' | 'android' | 'native' | 'default'` 的物件時，會根據當前運行平台返回最匹配的值。具體來說，在手機上運行時會優先採用 `ios` 和 `android` 鍵值；若未指定，則會使用 `native` 鍵值，最後才是 `default` 鍵值。

```tsx
import {Platform, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    ...Platform.select({
      ios: {
        backgroundColor: 'red',
      },
      android: {
        backgroundColor: 'green',
      },
      default: {
        // other platforms, web for example
        backgroundColor: 'blue',
      },
    }),
  },
});
```

這會讓容器在所有平台都具有 `flex: 1` 樣式，在 iOS 上顯示紅色背景，Android 上顯示綠色背景，其他平台則顯示藍色背景。

由於該方法接受 `any` 型別值，你也可以用它來返回平台特定元件，如下所示：

```tsx
const Component = Platform.select({
  ios: () => require('ComponentIOS'),
  android: () => require('ComponentAndroid'),
})();

<Component />;
```

```tsx
const Component = Platform.select({
  native: () => require('ComponentForNative'),
  default: () => require('ComponentForWeb'),
})();

<Component />;
```

### 檢測 Android 版本

在 Android 上，`Platform` 模組還可用於檢測應用程式運行的 Android 平台版本：

```tsx
import {Platform} from 'react-native';

if (Platform.Version === 25) {
  console.log('Running on Nougat!');
}
```

**注意**：`Version` 對應的是 Android API 版本號，而非 Android 作業系統版本號。版本對照表請參考 [Android 版本歷史](https://en.wikipedia.org/wiki/Android_version_history#Overview)。

### 檢測 iOS 版本

在 iOS 上，`Version` 是透過 `-[UIDevice systemVersion]` 取得的字串，表示當前作業系統版本（例如 "10.3"）。若要檢測 iOS 的主要版本號：

```tsx
import {Platform} from 'react-native';

const majorVersionIOS = parseInt(Platform.Version, 10);
if (majorVersionIOS <= 9) {
  console.log('Work around a change in behavior');
}
```

## 平台特定副檔名

當平台特定程式碼較為複雜時，建議將程式碼拆分至獨立檔案。React Native 會自動識別 `.ios.` 或 `.android.` 副檔名，並在需要時從其他元件載入對應的平台檔案。

例如，假設你的專案中有以下檔案：

```shell
BigButton.ios.js
BigButton.android.js
```

接著你可以這樣引入元件：

```tsx
import BigButton from './BigButton';
```

React Native 會根據運行平台自動選擇正確的檔案。

## 原生特定副檔名（與 NodeJS 和 Web 共享程式碼）

當模組需要在 NodeJS/Web 和 React Native 之間共享且無 Android/iOS 差異時，可使用 `.native.js` 副檔名。這對於需要在 React Native 和 ReactJS 之間共享程式碼的專案特別有用。

例如，假設你的專案中有以下檔案：

```shell
Container.js # picked up by webpack, Rollup or any other Web bundler
Container.native.js # picked up by the React Native bundler for both Android and iOS (Metro)
```

你仍可省略 `.native` 副檔名來引入該檔案：

```tsx
import Container from './Container';
```

**專業建議：** 配置您的 Web 打包工具忽略 `.native.js` 副檔名，以避免在生產環境包中包含未使用的代碼，從而減少最終包的大小。