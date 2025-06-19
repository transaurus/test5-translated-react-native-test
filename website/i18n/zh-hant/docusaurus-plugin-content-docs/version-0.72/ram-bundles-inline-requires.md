---
id: ram-bundles-inline-requires
title: RAM Bundles and Inline Requires
---

若您的應用程式規模龐大，可考慮採用隨機存取模組（RAM）套件格式並使用內嵌式 require。這對於包含大量畫面但典型使用情境中未必會開啟的應用程式特別有用，尤其適用於啟動後短時間內不需載入大量程式碼的情況。例如應用程式包含複雜的個人檔案頁面或較少使用的功能，但多數使用情境僅涉及主畫面更新。透過 RAM 格式與實際使用時才內嵌載入（inline require）這些功能，可有效優化套件載入效率。

## JavaScript 載入機制

在 react-native 執行 JS 程式碼前，必須先將程式碼載入記憶體並解析。傳統套件若載入 50MB 的套件，必須完整載入並解析全部內容後才能執行。RAM 套件的優化原理在於：啟動時僅載入實際需要的部分程式碼，後續再按需漸進式載入其他模組。

## 內嵌式 require

內嵌式 require 會延遲模組或檔案的載入時機，直到實際需要時才執行。基礎範例如下：

```tsx title="VeryExpensive.tsx"
import React, {Component} from 'react';
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

即使未使用 RAM 格式，內嵌式 require 仍能改善啟動時間，因為 VeryExpensive.js 的程式碼只會在首次被 require 時執行。

## 啟用 RAM 格式

在 iOS 平台，RAM 格式會建立單一索引檔案，react native 將逐個模組載入。Android 平台預設會為每個模組建立獨立檔案，您可強制 Android 建立類似 iOS 的單一檔案，但使用多個獨立檔案通常效能更佳且記憶體消耗更低。

在 Xcode 中啟用 RAM 格式：編輯「Bundle React Native code and images」建置階段，於 `../node_modules/react-native/scripts/react-native-xcode.sh` 前加入 `export BUNDLE_COMMAND="ram-bundle"`：

```
export BUNDLE_COMMAND="ram-bundle"
export NODE_BINARY=node
../node_modules/react-native/scripts/react-native-xcode.sh
```

在 Android 平台啟用 RAM 格式：編輯 `android/app/build.gradle` 檔案，於 `apply from: "../../node_modules/react-native/react.gradle"` 前新增或修改 `project.ext.react` 區塊：

```
project.ext.react = [
  bundleCommand: "ram-bundle",
]
```

若要在 Android 平台使用單一索引檔案，請加入以下設定：

```
project.ext.react = [
  bundleCommand: "ram-bundle",
  extraPackagerArgs: ["--indexed-ram-bundle"]
]
```

:::info
若您使用 [Hermes JS 引擎](https://github.com/facebook/hermes)，則**不應**啟用 RAM 套件功能。Hermes 載入位元碼時會透過 `mmap` 確保不會載入整個檔案，而 RAM 套件機制與此不相容，可能導致問題。
:::

## 預載與內嵌式 require 設定

Now that we have a RAM bundle, there is overhead for calling `require`. `require` now needs to send a message over the bridge when it encounters a module it has not loaded yet. This will impact startup the most, because that is where the largest number of require calls are likely to take place while the app loads the initial module. Luckily we can configure a portion of the modules to be preloaded. In order to do this, you will need to implement some form of inline require.

## 模組載入分析

在根檔案（index.(ios|android).js）的初始導入語句後加入以下程式碼：

```js
const modules = require.getModules();
const moduleIds = Object.keys(modules);
const loadedModuleNames = moduleIds
  .filter(moduleId => modules[moduleId].isInitialized)
  .map(moduleId => modules[moduleId].verboseName);
const waitingModuleNames = moduleIds
  .filter(moduleId => !modules[moduleId].isInitialized)
  .map(moduleId => modules[moduleId].verboseName);

// make sure that the modules you expect to be waiting are actually waiting
console.log(
  'loaded:',
  loadedModuleNames.length,
  'waiting:',
  waitingModuleNames.length,
);

// grab this text blob, and put it in a file named packager/modulePaths.js
console.log(
  `module.exports = ${JSON.stringify(
    loadedModuleNames.sort(),
    null,
    2,
  )};`,
);
```

運行應用程式時，可透過控制台觀察已載入與待載入模組數量。建議檢查 moduleNames 內容是否有異常模組。請注意：內嵌式 require 會在首次引用導入時觸發，可能需要重構程式碼以確保啟動時僅載入必要模組。您可透過修改 require 的 Systrace 物件來協助除錯異常 require 呼叫。

```js
require.Systrace.beginEvent = message => {
  if (message.includes(problematicModule)) {
    throw new Error();
  }
};
```

Every app is different, but it may make sense to only load the modules you need for the very first screen. When you are satisfied, put the output of the loadedModuleNames into a file named `packager/modulePaths.js`.

## 更新 metro.config.js 設定檔

現在我們需要更新專案根目錄下的 `metro.config.js` 檔案，以使用新生成的 `modulePaths.js` 檔案：

<!-- prettier-ignore -->

```js
const {getDefaultConfig, mergeConfig} = require('@react-native/metro-config');
const fs = require('fs');
const path = require('path');
const modulePaths = require('./packager/modulePaths');

const config = {
  transformer: {
    getTransformOptions: () => {
      const moduleMap = {};
      modulePaths.forEach(modulePath => {
        if (fs.existsSync(modulePath)) {
          moduleMap[path.resolve(modulePath)] = true;
        }
      });
      return {
        preloadedModules: moduleMap,
        transform: {inlineRequires: {blockList: moduleMap}},
      };
    },
  },
};

module.exports = mergeConfig(getDefaultConfig(__dirname), config);
```

另請參閱 [**設定 Metro**](/docs/metro#configuring-metro)。

The `preloadedModules` entry in the config indicates which modules should be marked as preloaded when building a RAM bundle. When the bundle is loaded, those modules are immediately loaded, before any requires have even executed. The `blockList` entry indicates that those modules should not be required inline. Because they are preloaded, there is no performance benefit from using an inline require. In fact the generated JavaScript spends extra time resolving the inline require every time the imports are referenced.

## 測試與效能改善評估

現在您應該已準備好使用 RAM 格式和內聯 require 來建置應用程式。請務必測量並比較啟用前後的啟動時間差異。