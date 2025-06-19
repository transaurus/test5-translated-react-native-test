---
id: optimizing-javascript-loading
title: Optimizing JavaScript loading
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

解析和執行 JavaScript 程式碼需要記憶體和時間。因此，隨著應用程式的增長，延遲載入程式碼直到首次需要時通常很有幫助。React Native 預設啟用了一些標準優化，您也可以在自己的程式碼中採用一些技巧來幫助 React 更有效率地載入應用程式。此外，還有一些適合大型應用程式的高階自動優化（各有其權衡取捨）。

## 推薦：使用 Hermes

Hermes 是新版 React Native 應用程式的預設引擎，並針對高效程式碼載入進行了高度優化。在正式版建置中，JavaScript 程式碼會預先完全編譯為位元組碼。位元組碼是按需載入到記憶體中的，不需要像純 JavaScript 那樣進行解析。

:::info
有關在 React Native 中使用 Hermes 的更多資訊，請參閱[這裡](./hermes)。
:::

## 推薦：延遲載入大型元件

如果一個包含大量程式碼/依賴項的元件在應用程式初始渲染時不太可能被使用，您可以使用 React 的 [`lazy`](https://react.dev/reference/react/lazy) API 來延遲載入其程式碼，直到首次渲染時才載入。通常，您應該考慮延遲載入應用程式中的畫面級元件，這樣新增畫面時不會增加應用程式的啟動時間。

:::info
有關[使用 Suspense 延遲載入元件](https://react.dev/reference/react/lazy#suspense-for-code-splitting)的更多資訊，包括程式碼範例，請參閱 React 的文件。
:::

### 提示：避免模組副作用

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

有時您可能希望延遲載入某些程式碼，直到首次使用時才載入，而不使用 `lazy` 或非同步 `import()`。您可以在檔案頂部使用 [`require()`](https://metrobundler.dev/docs/module-api/#require) 函數來實現這一點，而不是使用靜態 `import`。

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

## 進階：自動內聯 `require` 呼叫

如果您使用 React Native CLI 建置應用程式，`require` 呼叫（但不包括 `import`）會自動為您內聯，包括您的程式碼和使用的任何第三方套件（`node_modules`）中的呼叫。

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

### 內聯 `require` 的陷阱

Inlining `require` calls changes the order in which modules are evaluated, and can even cause some modules to _never_ be evaluated. This is usually safe to do automatically, because JavaScript modules are often written to be side-effect-free.

如果您的某個模組確實有副作用（例如初始化某些日誌機制或修補全域 API 供其他程式碼使用），則可能會出現意外行為甚至崩潰。在這些情況下，您可能需要將某些模組排除在此優化之外，或完全禁用此優化。

要**禁用所有 `require` 呼叫的自動內聯：**

更新您的 `metro.config.js`，將 `inlineRequires` 轉換器選項設定為 `false`：

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

要僅**排除某些模組不進行 `require` 內聯：**

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
**不支援 [使用 Hermes 時](#use-hermes)。** Hermes 位元組碼與 RAM 包格式不相容，但在所有使用情境下提供相同（或更好）的性能。
:::

隨機存取模組包（亦稱為 RAM 包）與上述技術協同工作，以限制需要解析並載入記憶體的 JavaScript 程式碼量。每個模組儲存為獨立的字串（或檔案），僅在需要執行該模組時才進行解析。

RAM 包可以物理上分割為多個檔案，或者使用 _索引_ 格式，即在單一檔案中包含多個模組的查找表。

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