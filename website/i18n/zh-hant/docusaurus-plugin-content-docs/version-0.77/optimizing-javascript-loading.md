---
id: optimizing-javascript-loading
title: Optimizing JavaScript loading
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

解析和執行 JavaScript 代碼需要消耗記憶體與時間。因此隨著應用程式增長，延遲載入代碼直到首次需要時使用通常很有幫助。React Native 預設啟用了一些標準優化功能，您也可以在自己的代碼中採用特定技巧來幫助 React 更有效率地載入應用程式。對於超大型應用程式，還有一些進階的自動化優化方案（各有利弊）可供選擇。

## 推薦方案：使用 Hermes 引擎

Hermes 是新版 React Native 應用的預設引擎，針對代碼載入效率進行了高度優化。在正式版建置中，JavaScript 代碼會預先完整編譯為位元組碼。位元組碼會按需載入記憶體，無需像普通 JavaScript 那樣進行解析。

:::info
詳細了解如何在 React Native 中使用 Hermes [請參閱此文件](./hermes)。
:::

## 推薦方案：延遲載入大型元件

若某個包含大量代碼/依賴項的元件在應用初始渲染時不太可能被使用，您可以使用 React 的 [`lazy`](https://react.dev/reference/react/lazy) API 來延遲載入其代碼，直到首次渲染時才載入。通常建議對應用中的畫面級元件實施延遲載入，這樣新增畫面時就不會增加應用啟動時間。

:::info
詳細了解如何[使用 Suspense 延遲載入元件](https://react.dev/reference/react/lazy#suspense-for-code-splitting)（含代碼範例），請參閱 React 官方文件。
:::

### 技巧：避免模組副作用

Lazy-loading components can change the behavior of your app if your component modules (or their dependencies) have _side effects_, such as modifying global variables or subscribing to events outside of a component. Most modules in React apps should not have any side effects.

```tsx title="SideEffects.tsx"
import Logger from './utils/Logger';

//  🚩 🚩 🚩 Side effect! This must be executed before React can even begin to
// render the SplashScreen component, and can unexpectedly break code elsewhere
// in your app if you later decide to lazy-load SplashScreen.
global.logger = new Logger();

export function SplashScreen() {
  // ...
}
```

## Advanced: Call `require` inline

Sometimes you may want to defer loading some code until you use it for the first time, without using `lazy` or an asynchronous `import()`. You can do this by using the [`require()`](https://metrobundler.dev/docs/module-api/#require) function where you would otherwise use a static `import` at the top of the file.

```tsx title="VeryExpensive.tsx"
import {Component} from 'react';
import {Text} from 'react-native';
// ... import some very expensive modules

export default function VeryExpensive() {
  // ... lots and lots of rendering logic
  return <Text>Very Expensive Component</Text>;
}
```

```tsx title="Optimized.tsx"
import {useCallback, useState} from 'react';
import {TouchableOpacity, View, Text} from 'react-native';
// Usually we would write a static import:
// import VeryExpensive from './VeryExpensive';

let VeryExpensive = null;

export default function Optimize() {
  const [needsExpensive, setNeedsExpensive] = useState(false);
  const didPress = useCallback(() => {
    if (VeryExpensive == null) {
      VeryExpensive = require('./VeryExpensive').default;
    }

    setNeedsExpensive(true);
  }, []);

  return (
    <View style={{marginTop: 20}}>
      <TouchableOpacity onPress={didPress}>
        <Text>Load</Text>
      </TouchableOpacity>
      {needsExpensive ? <VeryExpensive /> : null}
    </View>
  );
}
```

## 進階技巧：自動內聯 `require` 呼叫

若使用 React Native CLI 建置應用，系統會自動為您內聯所有 `require` 呼叫（不包括 `import`），此優化同時作用於您的代碼和使用的第三方套件（`node_modules`）。

```tsx
import {useCallback, useState} from 'react';
import {TouchableOpacity, View, Text} from 'react-native';

// This top-level require call will be evaluated lazily as part of the component below.
const VeryExpensive = require('./VeryExpensive').default;

export default function Optimize() {
  const [needsExpensive, setNeedsExpensive] = useState(false);
  const didPress = useCallback(() => {
    setNeedsExpensive(true);
  }, []);

  return (
    <View style={{marginTop: 20}}>
      <TouchableOpacity onPress={didPress}>
        <Text>Load</Text>
      </TouchableOpacity>
      {needsExpensive ? <VeryExpensive /> : null}
    </View>
  );
}
```

:::info
Some React Native frameworks disable this behavior. In particular, in Expo projects, `require` calls are not inlined by default. You can enable this optimization by editing your project's Metro config and setting `inlineRequires: true` in [`getTransformOptions`](https://metrobundler.dev/docs/configuration#gettransformoptions).
:::

### 內聯 `require` 的潛在問題

Inlining `require` calls changes the order in which modules are evaluated, and can even cause some modules to _never_ be evaluated. This is usually safe to do automatically, because JavaScript modules are often written to be side-effect-free.

若您的某個模組確實具有副作用（例如初始化日誌機制或修補全域 API），則可能會出現意外行為甚至崩潰。此時您可能需要將特定模組排除在此優化外，或完全停用該功能。

To **disable all automatic inlining of `require` calls:**

更新您的 `metro.config.js` 文件，將轉換器選項 `inlineRequires` 設為 `false`：

```tsx title="metro.config.js"
module.exports = {
  transformer: {
    async getTransformOptions() {
      return {
        transform: {
          inlineRequires: false,
        },
      };
    },
  },
};
```

To only **exclude certain modules from `require` inlining:**

有兩個相關的轉換器選項：`inlineRequires.blockList` 和 `nonInlinedRequires`。請參閱程式碼片段以了解如何使用每個選項的範例。

```tsx title="metro.config.js"
module.exports = {
  transformer: {
    async getTransformOptions() {
      return {
        transform: {
          inlineRequires: {
            blockList: {
              // require() calls in `DoNotInlineHere.js` will not be inlined.
              [require.resolve('./src/DoNotInlineHere.js')]: true,

              // require() calls anywhere else will be inlined, unless they
              // match any entry nonInlinedRequires (see below).
            },
          },
          nonInlinedRequires: [
            // require('react') calls will not be inlined anywhere
            'react',
          ],
        },
      };
    },
  },
};
```

See the documentation for [`getTransformOptions` in Metro](https://metrobundler.dev/docs/configuration#gettransformoptions) for more details on setting up and fine-tuning your inline `require`s.

## 進階：使用隨機存取模組包（非 Hermes）

:::info
**Not supported when [using Hermes](#use-hermes).** Hermes bytecode is not compatible with the RAM bundle format, and provides the same (or better) performance in all use cases.
:::

隨機存取模組包（亦稱為 RAM 包）與上述技術協同工作，以限制需要解析並載入記憶體的 JavaScript 程式碼量。每個模組儲存為獨立的字串（或檔案），僅在模組需要執行時才進行解析。

RAM 包可以物理上分割為多個檔案，或使用 _索引_ 格式，即在單一檔案中包含多個模組的查找表。

<Tabs groupId="platform" queryString defaultValue={constants.defaultPlatform} values={constants.platforms}>
<TabItem value="android">

On Android enable the RAM format by editing your `android/app/build.gradle` file. Before the line `apply from: "../../node_modules/react-native/react.gradle"` add or amend the `project.ext.react` block:

```
project.ext.react = [
  bundleCommand: "ram-bundle",
]
```

Use the following lines on Android if you want to use a single indexed file:

```
project.ext.react = [
  bundleCommand: "ram-bundle",
  extraPackagerArgs: ["--indexed-ram-bundle"]
]
```

</TabItem>
<TabItem value="ios">

On iOS, RAM bundles are always indexed ( = single file).

Enable the RAM format in Xcode by editing the build phase "Bundle React Native code and images". Before `../node_modules/react-native/scripts/react-native-xcode.sh` add `export BUNDLE_COMMAND="ram-bundle"`:

```
export BUNDLE_COMMAND="ram-bundle"
export NODE_BINARY=node
../node_modules/react-native/scripts/react-native-xcode.sh
```

</TabItem>
</Tabs>

See the documentation for [`getTransformOptions` in Metro](https://metrobundler.dev/docs/configuration#gettransformoptions) for more details on setting up and fine-tuning your RAM bundle build.