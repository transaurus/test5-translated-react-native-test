---
id: ram-bundles-inline-requires
title: RAM Bundles and Inline Requires
---

若您的應用程式規模較大，可考慮採用隨機存取模組（RAM）套件格式並搭配行內引用（inline requires）。此方式特別適合具有大量畫面、但典型使用情境中未必會開啟的應用程式。通常這類應用程式在啟動後較長時間內並不需要執行大量程式碼，例如包含複雜個人檔案頁面或較少使用功能的應用程式，但多數使用階段僅會造訪應用程式主畫面進行更新。透過RAM格式與行內引用（實際使用時才載入這些功能與畫面），我們能優化套件的載入效率。

## JavaScript載入機制

在react-native執行JS程式碼前，必須先將程式碼載入記憶體並解析。若使用標準套件，當載入50MB的套件時，必須完整載入並解析全部50MB後才能開始執行。RAM套件的優化原理在於：啟動時僅載入您實際需要的部分內容，後續再根據需求逐步載入套件的其他區段。

## 行內引用

行內引用會延遲模組或檔案的載入時機，直到實際需要該檔案時才執行。基礎範例如下：

```tsx title="VeryExpensive.tsx"
import React, {Component} from 'react';
import {Text} from 'react-native';
// ... import some very expensive modules

// You may want to log at the file level to verify when this is happening
console.log('VeryExpensive component loaded');

export default class VeryExpensive extends Component {
  // lots and lots of code
  render() {
    return <Text>Very Expensive Component</Text>;
  }
}
```

```tsx title="Optimized.tsx"
import React, {Component} from 'react';
import {TouchableOpacity, View, Text} from 'react-native';

let VeryExpensive = null;

export default class Optimized extends Component {
  state = {needsExpensive: false};

  didPress = () => {
    if (VeryExpensive == null) {
      VeryExpensive = require('./VeryExpensive').default;
    }

    this.setState(() => ({
      needsExpensive: true,
    }));
  };

  render() {
    return (
      <View style={{marginTop: 20}}>
        <TouchableOpacity onPress={this.didPress}>
          <Text>Load</Text>
        </TouchableOpacity>
        {this.state.needsExpensive ? <VeryExpensive /> : null}
      </View>
    );
  }
}
```

即使未採用RAM格式，行內引用仍能改善啟動時間，因為VeryExpensive.js內的程式碼僅會在首次被引用時才會執行。

## 啟用RAM格式

在iOS平台使用RAM格式會建立單一索引檔案，react native將逐個模組載入。Android平台預設會為每個模組建立獨立檔案，您可強制Android像iOS一樣建立單一檔案，但使用多個獨立檔案通常效能更佳且記憶體消耗更低。

在Xcode中編輯「Bundle React Native code and images」建置階段以啟用RAM格式。於`../node_modules/react-native/scripts/react-native-xcode.sh`前加入`export BUNDLE_COMMAND="ram-bundle"`：

```
export BUNDLE_COMMAND="ram-bundle"
export NODE_BINARY=node
../node_modules/react-native/scripts/react-native-xcode.sh
```

在Android平台編輯`android/app/build.gradle`檔案以啟用RAM格式。於`apply from: "../../node_modules/react-native/react.gradle"`前加入或修改`project.ext.react`區塊：

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
若您使用[Hermes JS引擎](https://github.com/facebook/hermes)，則**不應**啟用RAM套件功能。Hermes載入位元碼時，`mmap`機制能確保不會載入整個檔案。Hermes與RAM套件併用可能導致問題，因兩者的運作機制互不相容。
:::

## 預載與行內引用設定

採用RAM套件後，呼叫`require`會產生額外開銷。當遇到尚未載入的模組時，`require`需透過橋接器發送訊息。這對啟動階段影響最大，因為此時應用程式載入初始模組時可能需呼叫大量require。所幸我們可設定部分模組進行預載，為此您需實作某種形式的行內引用。

## 分析已載入模組

您可在根檔案（index.(ios|android).js）的初始引用後加入以下程式碼：

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

執行應用程式時，可透過控制台觀察已載入與待載入模組數量。建議檢視moduleNames內容確認是否有非預期模組。請注意：行內引用會在首次參照導入時觸發。您可能需要進行調查與重構，確保啟動時僅載入必要模組。您可透過修改require上的Systrace物件來協助除錯有問題的引用。

```js
require.Systrace.beginEvent = message => {
  if (message.includes(problematicModule)) {
    throw new Error();
  }
};
```

Every app is different, but it may make sense to only load the modules you need for the very first screen. When you are satisfied, put the output of the loadedModuleNames into a file named `packager/modulePaths.js`.

## 更新 metro.config.js

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

設定檔中的 `preloadedModules` 項目標記了哪些模組應在建置 RAM bundle 時預先載入。當 bundle 載入時，這些模組會立即載入，甚至在執行任何 require 之前。`blockList` 項目則標記了哪些模組不應使用 inline require。由於這些模組已預先載入，使用 inline require 並不會帶來效能優勢。實際上，生成的 JavaScript 程式碼每次參照這些導入時，反而會花費額外時間解析 inline require。

## 測試與測量改進效果

現在您應該已準備好使用 RAM 格式和 inline requires 來建置應用程式。請務必測量並比較啟用前後的啟動時間。