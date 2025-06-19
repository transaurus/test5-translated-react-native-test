---
id: ram-bundles-inline-requires
title: RAM Bundles and Inline Requires
---

若您的應用程式規模較大，可考慮採用隨機存取模組(RAM)套件格式並搭配行內(inline)引入(require)。這對於擁有大量畫面、但典型使用情境中未必會開啟的應用程式特別有用。通常適用於啟動後短時間內不需載入大量程式碼的場景，例如應用程式包含複雜的個人檔案頁面或較少使用的功能，但多數使用階段僅涉及主畫面更新。透過RAM格式與實際使用時才動態引入的功能模組，我們能有效優化套件載入效率。

## JavaScript載入機制

在react-native執行JS程式碼前，必須先將程式碼載入記憶體並解析。傳統套件若載入50MB的檔案，必須完整載入並解析全部內容後才能開始執行。RAM套件的優化原理在於：啟動時僅載入實際需要的部分內容，後續再按需漸進式載入其他模組。

## 行內引入(Inline Requires)

行內引入可延遲模組或檔案的載入時機，直到真正需要使用時才載入。基礎範例如下：

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

即使未採用RAM格式，行內引入仍能改善啟動時間，因為VeryExpensive.js內的程式碼僅會在首次被引入時執行。

## 啟用RAM格式

在iOS平台啟用RAM格式會建立單一索引檔案，react native將逐個模組載入。Android平台預設會為每個模組建立獨立檔案，您可強制Android像iOS一樣建立單一檔案，但使用多個獨立檔案通常更具效能且記憶體消耗更低。

在Xcode中編輯「Bundle React Native code and images」建置階段以啟用RAM格式。於`../node_modules/react-native/scripts/react-native-xcode.sh`前加入`export BUNDLE_COMMAND="ram-bundle"`：

```
export BUNDLE_COMMAND="ram-bundle"
export NODE_BINARY=node
../node_modules/react-native/scripts/react-native-xcode.sh
```

Android平台請編輯`android/app/build.gradle`檔案，在`apply from: "../../node_modules/react-native/react.gradle"`前新增或修改`project.ext.react`區塊：

```
project.ext.react = [
  bundleCommand: "ram-bundle",
]
```

若要在Android平台使用單一索引檔案，請加入以下設定：

```
project.ext.react = [
  bundleCommand: "ram-bundle",
  extraPackagerArgs: ["--indexed-ram-bundle"]
]
```

:::info
若使用[Hermes JS引擎](https://github.com/facebook/hermes)，**不應**啟用RAM套件功能。Hermes載入位元碼時會透過`mmap`確保不會載入整個檔案，與RAM套件機制不相容，可能導致問題。
:::

## 預載與行內引入配置

Now that we have a RAM bundle, there is overhead for calling `require`. `require` now needs to send a message over the bridge when it encounters a module it has not loaded yet. This will impact startup the most, because that is where the largest number of require calls are likely to take place while the app loads the initial module. Luckily we can configure a portion of the modules to be preloaded. In order to do this, you will need to implement some form of inline require.

## 模組載入分析

在根檔案(index.(ios|android).js)的初始引入(import)後加入以下程式碼：

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

執行應用程式時，可從控制台觀察已載入與待載入模組數量。建議檢查moduleNames內容是否有異常模組。請注意行內引入會在首次參照時觸發，可能需要重構程式碼確保啟動時僅載入必要模組。您可修改require上的Systrace物件協助除錯有問題的引入。

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

設定檔中的 `preloadedModules` 參數用於指定在建置 RAM bundle 時應標記為預先載入的模組。當 bundle 載入時，這些模組會立即載入，甚至在執行任何 require 之前就已準備就緒。而 `blockList` 參數則表示這些模組不應使用內聯 require。由於它們已被預先載入，使用內聯 require 並不會帶來效能優勢。實際上，生成的 JavaScript 程式碼每次解析這些內聯 require 時反而會耗費額外時間。

## 測試與效能改善評估

現在您應該已準備好使用 RAM 格式和內聯 require 來建置應用程式。請務必測量並比較啟用前後的啟動時間差異。