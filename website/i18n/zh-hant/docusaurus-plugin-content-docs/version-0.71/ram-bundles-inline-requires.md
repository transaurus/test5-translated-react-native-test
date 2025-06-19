---
id: ram-bundles-inline-requires
title: RAM Bundles and Inline Requires
---

若您的應用程式規模龐大，可考慮採用隨機存取模組（RAM）套件格式並搭配行內引用（inline requires）。此方式特別適用於具有大量畫面、但典型使用情境中未必會開啟的應用程式。通常這類應用程式在啟動後相當時間內並不需要執行大量程式碼，例如包含複雜個人檔案頁面或較少使用功能的應用程式，但多數使用者會話僅涉及主畫面更新。透過RAM格式與行內引用（實際使用時才載入），我們能優化套件載入效率。

## JavaScript載入機制

在React Native執行JS程式碼前，必須先將程式碼載入記憶體並解析。傳統套件若載入50MB的套件，必須完整載入並解析全部50MB後才能執行。RAM套件的優化原理在於：啟動時僅載入實際需要的部分，後續再按需漸進式載入其他模組。

## 行內引用

行內引用會延遲模組或檔案的載入時機，直到實際需要時才執行。基礎範例如下：

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

即使未使用RAM格式，行內引用仍能改善啟動時間，因為VeryExpensive.js內的程式碼僅會在首次被引用時執行。

## 啟用RAM格式

在iOS平台，RAM格式會建立單一索引檔案，React Native將逐個模組載入。Android平台預設會為每個模組建立獨立檔案，您可強制Android像iOS一樣建立單一檔案，但使用多檔案模式通常更具效能且記憶體消耗更低。

在Xcode中編輯「Bundle React Native code and images」建置階段以啟用RAM格式。於`../node_modules/react-native/scripts/react-native-xcode.sh`前添加`export BUNDLE_COMMAND="ram-bundle"`：

```
export BUNDLE_COMMAND="ram-bundle"
export NODE_BINARY=node
../node_modules/react-native/scripts/react-native-xcode.sh
```

在Android平台編輯`android/app/build.gradle`檔案啟用RAM格式。於`apply from: "../../node_modules/react-native/react.gradle"`前添加或修改`project.ext.react`區塊：

```
project.ext.react = [
  bundleCommand: "ram-bundle",
]
```

若要在Android平台使用單一索引檔案，請添加以下設定：

```
project.ext.react = [
  bundleCommand: "ram-bundle",
  extraPackagerArgs: ["--indexed-ram-bundle"]
]
```

:::info
若您使用[Hermes JS引擎](https://github.com/facebook/hermes)，則**不應**啟用RAM套件功能。Hermes載入位元碼時，`mmap`會確保不會載入整個檔案。Hermes與RAM套件機制不相容，同時使用可能導致問題。
:::

## 預載與行內引用配置

使用RAM套件後，呼叫`require`會產生額外開銷。當遇到尚未載入的模組時，`require`需透過橋接器發送訊息。這對啟動階段影響最大，因為此時需載入初始模組而產生大量require呼叫。我們可透過預載部分模組來優化，這需要實作某種形式的行內引用。

## 模組載入分析

在根檔案（index.(ios|android).js）的初始導入後添加以下程式碼：

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

執行應用程式時，可於控制台查看已載入與待載入模組數量。建議檢查moduleNames內容是否有異常模組。請注意行內引用會在首次參照時觸發，可能需要重構程式碼以確保啟動時僅載入必要模組。您可透過修改require上的Systrace物件來協助除錯有問題的引用。

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

```js
const modulePaths = require('./packager/modulePaths');
const resolve = require('path').resolve;
const fs = require('fs');

// Update the following line if the root folder of your app is somewhere else.
const ROOT_FOLDER = resolve(__dirname, '..');

const config = {
  transformer: {
    getTransformOptions: () => {
      const moduleMap = {};
      modulePaths.forEach(path => {
        if (fs.existsSync(path)) {
          moduleMap[resolve(path)] = true;
        }
      });
      return {
        preloadedModules: moduleMap,
        transform: {inlineRequires: {blockList: moduleMap}},
      };
    },
  },
  projectRoot: ROOT_FOLDER,
};

module.exports = config;
```

設定檔中的 `preloadedModules` 參數用於指定建構 RAM bundle 時應標記為預載的模組。當 bundle 載入時，這些模組會立即載入，甚至在執行任何 require 之前。而 `blockList` 參數則標示這些模組不應使用 inline require。由於它們已被預載，使用 inline require 並不會帶來效能優勢。實際上，生成的 JavaScript 程式碼每次參照這些匯入時，反而會額外耗費時間解析 inline require。

## 測試與效能改善評估

現在您應該已準備好使用 RAM 格式與 inline requires 來建置應用程式。請務必測量並比較啟用前後的啟動時間差異。