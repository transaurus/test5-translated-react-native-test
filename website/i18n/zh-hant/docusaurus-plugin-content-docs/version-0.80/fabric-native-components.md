---
id: fabric-native-components-introduction
title: Fabric Native Components Introduction
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import constants from '@site/core/TabsConstants';
import {FabricNativeComponentsAndroid,FabricNativeComponentsIOS} from './\_fabric-native-components';

# 原生元件

If you want to build _new_ React Native Components that wrap around a [Host Component](https://reactnative.dev/architecture/glossary#host-view-tree-and-host-view) like a unique kind of [CheckBox](https://developer.android.com/reference/androidx/appcompat/widget/AppCompatCheckBox) on Android, or a [UIButton](https://developer.apple.com/documentation/uikit/uibutton?language=objc) on iOS, you should use a Fabric Native Component.

本指南將透過實作網頁視圖元件，示範如何建構 Fabric 原生元件。具體步驟如下：

1. 使用 Flow 或 TypeScript 定義 JavaScript 規格
2. 設定依賴管理系統，使其能根據規格自動生成程式碼並完成連結
3. 實作原生平台程式碼
4. 在應用程式中使用該元件

您需要一個基礎模板生成的應用程式來使用此元件：

```bash
npx @react-native-community/cli@latest init Demo --install-pods false
```

## 建立網頁視圖元件

本指南將示範如何建立網頁視圖元件。我們將使用 Android 的[`WebView`](https://developer.android.com/reference/android/webkit/WebView)元件與 iOS 的[`WKWebView`](https://developer.apple.com/documentation/webkit/wkwebview?language=objc)元件來實作。

首先建立以下目錄結構來存放元件程式碼：

```bash
mkdir -p Demo/{specs,android/app/src/main/java/com/webview}
```

您將在此目錄結構下進行開發：

```
Demo
├── android/app/src/main/java/com/webview
└── ios
└── specs
```

- The `android/app/src/main/java/com/webview` folder is the folder that will contain our Android code.
- The `ios` folder is the folder that will contain our iOS code.
- The `specs` folder is the folder that will contain the Codegen's specification file.

## 1. 為 Codegen 定義規格

您的規格必須使用 [TypeScript](https://www.typescriptlang.org/) 或 [Flow](https://flow.org/) 定義（詳見 [Codegen](the-new-architecture/what-is-codegen) 文件）。這些規格將被 Codegen 用來生成 C++、Objective-C++ 和 Java 程式碼，以連接您的平台程式碼與 React 運行的 JavaScript 執行環境。

The specification file must be named `<MODULE_NAME>NativeComponent.{ts|js}` to work with Codegen. The suffix `NativeComponent` is not only a convention, it is actually used by Codegen to detect a spec file.

以下是我們網頁視圖元件的規格範例：

<Tabs groupId="language" queryString defaultValue={constants.defaultJavaScriptSpecLanguage} values={constants.javaScriptSpecLanguages}>
<TabItem value="typescript">

```typescript title="Demo/specs/WebViewNativeComponent.ts"
import type {
  CodegenTypes,
  HostComponent,
  ViewProps,
} from 'react-native';
import {codegenNativeComponent} from 'react-native';

type WebViewScriptLoadedEvent = {
  result: 'success' | 'error';
};

export interface NativeProps extends ViewProps {
  sourceURL?: string;
  onScriptLoaded?: CodegenTypes.BubblingEventHandler<WebViewScriptLoadedEvent> | null;
}

export default codegenNativeComponent<NativeProps>(
  'CustomWebView',
) as HostComponent<NativeProps>;
```

</TabItem>
<TabItem value="flow">

```ts title="Demo/RCTWebView/js/RCTWebViewNativeComponent.js":
// @flow strict-local

import type {CodegenTypes, HostComponent, ViewProps} from 'react-native';
import {codegenNativeComponent} from 'react-native';

type WebViewScriptLoadedEvent = $ReadOnly<{|
  result: "success" | "error",
|}>;

type NativeProps = $ReadOnly<{|
  ...ViewProps,
  sourceURL?: string;
  onScriptLoaded?: CodegenTypes.BubblingEventHandler<WebViewScriptLoadedEvent>?;
|}>;

export default (codegenNativeComponent<NativeProps>(
  'CustomWebView',
): HostComponent<NativeProps>);

```

</TabItem>
</Tabs>

此規格主要包含三個部分（不含導入語句）：

- The `WebViewScriptLoadedEvent` is a supporting data type for the data the event needs to pass from native to JavaScript.
- The `NativeProps` is a definition of the props that we can set on the component.
- The `codegenNativeComponent` statement allows us to codegenerate the code for the custom component and that defines a name for the component used to match the native implementations.

與原生模組相同，您可以在 `specs/` 目錄中放置多個規格文件。有關可用型別及其對應平台型別的詳細資訊，請參閱[附錄](appendix.md#codegen-typings)。

## 2. 設定 Codegen 執行環境

React Native 的 Codegen 工具會根據規格生成平台特定介面與樣板程式碼。為此，Codegen 需要知道規格文件的位置與處理方式。請更新您的 `package.json` 包含以下內容：

```json package.json
    "start": "react-native start",
    "test": "jest"
  },
  // highlight-start
  "codegenConfig": {
    "name": "AppSpec",
    "type": "components",
    "jsSrcsDir": "specs",
    "android": {
      "javaPackageName": "com.webview"
    },
    "ios": {
      "componentProvider": {
        "CustomWebView": "RCTWebView"
      }
    }
  },
  // highlight-end
  "dependencies": {
```

完成 Codegen 的基礎設定後，我們需要準備原生程式碼來對接生成的程式碼。

請注意，在 iOS 平台我們需明確將規格導出的 JS 元件名稱（`CustomWebView`）與實作該元件的原生 iOS 類別進行映射。

## 2. 建置原生程式碼

現在是時候撰寫原生平台程式碼，以便當 React 要求渲染視圖時，平台能建立正確的原生視圖並將其渲染到畫面上。

您需要分別完成 Android 和 iOS 平台的實作。

:::note
本指南展示如何建立僅適用於新架構的原生元件。若需同時支援新架構與舊架構，請參閱我們的[向後相容性指南](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/backwards-compat.md)。

:::

<Tabs groupId="platforms" queryString defaultValue={constants.defaultPlatform}>
    <TabItem value="android" label="Android">
        <FabricNativeComponentsAndroid />
    </TabItem>
    <TabItem value="ios" label="iOS">
        <FabricNativeComponentsIOS />
    </TabItem>
</Tabs>

## 3. 使用您的原生元件

最後，您可以在應用程式中使用新元件。請更新生成的 `App.tsx` 檔案如下：

```javascript title="Demo/App.tsx"
import React from 'react';
import {Alert, StyleSheet, View} from 'react-native';
import WebView from './specs/WebViewNativeComponent';

function App(): React.JSX.Element {
  return (
    <View style={styles.container}>
      <WebView
        sourceURL="https://react.dev/"
        style={styles.webview}
        onScriptLoaded={() => {
          Alert.alert('Page Loaded');
        }}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    alignContent: 'center',
  },
  webview: {
    width: '100%',
    height: '100%',
  },
});

export default App;
```

這段程式碼建立了一個應用程式，使用我們創建的 `WebView` 元件來載入 `react.dev` 網站。

當網頁載入完成時，應用程式還會顯示一個警示框。

## 4. 使用 WebView 元件執行您的應用程式

<Tabs groupId="platforms" queryString defaultValue={constants.defaultPlatform}>
<TabItem value="android" label="Android">
```bash
yarn run android
```
</TabItem>
<TabItem value="ios" label="iOS">
```bash
yarn run ios
```
</TabItem>
</Tabs>

|                                      Android                                      |                                     iOS                                      |
| :-------------------------------------------------------------------------------: | :--------------------------------------------------------------------------: |
| <img style={{ "max-height": "600px" }} src="/docs/assets/webview-android.webp" /> | <img style={{"max-height": "600px" }} src="/docs/assets/webview-ios.webp" /> |