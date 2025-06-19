---
id: platform-specific-code
title: Platform-Specific Code
---

在開發跨平台應用程式時，您會希望盡可能重複使用程式碼。但某些情況下可能需要針對不同平台實作差異化程式碼，例如為 Android 和 iOS 分別設計不同的視覺元件。

React Native 提供兩種組織程式碼並區分平台的方式：

- Using the [`Platform` module](platform-specific-code.md#platform-module).
- Using [platform-specific file extensions](platform-specific-code.md#platform-specific-extensions).

某些元件可能包含僅在特定平台有效的屬性。這些屬性都標註有 `@platform` 註解，並在官方網站上顯示小徽章標記。

## Platform 模組

React Native 提供了一個能檢測應用程式運行平台的模組。您可以使用此檢測邏輯來實作平台特定程式碼。當只有小部分元件需要區分平台時，建議採用此方式。

```tsx
import {Platform, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  height: Platform.OS === 'ios' ? 200 : 100,
});
```

`Platform.OS` 在 iOS 上會返回 `ios`，在 Android 上則返回 `android`。

另提供 `Platform.select` 方法，該方法接收一個鍵值為 `'ios' | 'android' | 'native' | 'default'` 的物件，並返回最符合當前平台的數值。手機平台上會優先匹配 `ios` 和 `android` 鍵值，若未指定則使用 `native` 鍵值，最後才使用 `default` 鍵值。

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

這將使容器在所有平台上都具有 `flex: 1` 樣式，iOS 上顯示紅色背景，Android 上顯示綠色背景，其他平台則顯示藍色背景。

由於該方法接受 `any` 型別參數，您也可以用它來返回平台特定元件，如下所示：

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

### Detecting the Android version <div class="label android" title="This section is related to Android platform">Android</div>

在 Android 上，`Platform` 模組還可用於檢測應用程式運行的 Android 平台版本：

```tsx
import {Platform} from 'react-native';

if (Platform.Version === 25) {
  console.log('Running on Nougat!');
}
```

**注意**：`Version` 對應的是 Android API 版本號而非作業系統版本號。版本對照表請參閱 [Android 版本歷史](https://en.wikipedia.org/wiki/Android_version_history#Overview)。

### Detecting the iOS version <div class="label ios" title="This section is related to iOS platform">iOS</div>

在 iOS 上，`Version` 是透過 `-[UIDevice systemVersion]` 取得的作業系統版本字串，例如 "10.3"。若要檢測 iOS 的主要版本號：

```tsx
import {Platform} from 'react-native';

const majorVersionIOS = parseInt(Platform.Version, 10);
if (majorVersionIOS <= 9) {
  console.log('Work around a change in behavior');
}
```

## 平台特定副檔名

當平台特定程式碼較為複雜時，建議將程式碼拆分至獨立檔案。React Native 會自動識別 `.ios.` 或 `.android.` 副檔名，並在需要時載入對應的平台檔案。

例如，假設您的專案中有以下檔案：

```shell
BigButton.ios.js
BigButton.android.js
```

您可以透過以下方式導入元件：

```tsx
import BigButton from './BigButton';
```

React Native 會根據運行平台自動選擇正確的檔案。

## 原生特定副檔名（與 NodeJS 和 Web 共享程式碼）

當模組需要在 NodeJS/Web 與 React Native 之間共享且無 Android/iOS 差異時，可使用 `.native.js` 副檔名。這對於 React Native 與 ReactJS 共享通用程式碼的專案特別有用。

舉例來說，假設您的專案中包含以下檔案：

```shell
Container.js # picked up by webpack, Rollup or any other Web bundler
Container.native.js # picked up by the React Native bundler for both Android and iOS (Metro)
```

您仍可省略 `.native` 副檔名進行導入，如下所示：

```tsx
import Container from './Container';
```

**專業建議：** 請設定您的網頁打包工具忽略 `.native.js` 副檔名，以避免生產環境套件中包含未使用的程式碼，從而減少最終套件體積。