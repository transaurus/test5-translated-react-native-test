---
id: typescript
title: Using TypeScript
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

[TypeScript][ts] 是一種透過添加型別定義來擴展 JavaScript 的語言。新建的 React Native 專案預設採用 TypeScript，但同時也支援 JavaScript 和 Flow。

## TypeScript 入門指南

透過 [React Native CLI](getting-started-without-a-framework#step-1-creating-a-new-application) 或熱門模板（如 [Ignite][ignite]）新建的專案會預設使用 TypeScript。

TypeScript 也可與 [Expo][expo] 搭配使用，後者維護了 TypeScript 模板，或在專案中添加 `.ts` 或 `.tsx` 檔案時會提示自動安裝並配置 TypeScript。

```shell
npx create-expo-app --template
```

## 在現有專案中添加 TypeScript

1. 將 TypeScript、型別定義和 ESLint 插件添加到您的專案中。

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm install -D typescript @react-native/typescript-config @types/jest @types/react @types/react-test-renderer
```

</TabItem>
<TabItem value="yarn">

```shell
yarn add --dev typescript @react-native/typescript-config @types/jest @types/react @types/react-test-renderer
```

</TabItem>
</Tabs>

:::note
此命令會添加所有依賴項的最新版本。可能需要調整版本以匹配專案現有套件。您可以使用 [React Native Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) 等工具查看 React Native 內建的版本。
:::

2. 添加 TypeScript 配置檔案。在專案根目錄創建 `tsconfig.json`：

```json title="tsconfig.json"
{
  "extends": "@react-native/typescript-config"
}
```

3. 將 JavaScript 檔案重新命名為 `*.tsx`

> 您應保持 `./index.js` 入口檔案不變，否則在打包生產版本時可能會遇到問題。

4. 執行 `tsc` 進行新 TypeScript 檔案的型別檢查。

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npx tsc
```

</TabItem>
<TabItem value="yarn">

```shell
yarn tsc
```

</TabItem>
</Tabs>

## 使用 JavaScript 替代 TypeScript

React Native 新建專案預設採用 TypeScript，但仍可使用 JavaScript。副檔名為 `.jsx` 的檔案會被視為 JavaScript 而非 TypeScript，且不會進行型別檢查。JavaScript 模組仍可被 TypeScript 模組導入，反之亦然。

## TypeScript 與 React Native 的協作原理

開箱即用，TypeScript 原始碼在打包時會透過 [Babel][babel] 進行轉換。我們建議僅使用 TypeScript 編譯器進行型別檢查。這是新建專案中 `tsc` 的預設行為。若您要將現有 TypeScript 程式碼移植到 React Native，使用 Babel 替代 TypeScript 時需注意 [一兩個注意事項][babel-7-caveats]。

## React Native + TypeScript 實例展示

You can provide an interface for a React Component's [Props](props) and [State](state) via `React.Component<Props, State>` which will provide type-checking and editor auto-completing when working with that component in JSX.

```tsx title="components/Hello.tsx"
import {useState} from 'react';
import {Button, StyleSheet, Text, View} from 'react-native';

export type Props = {
  name: string;
  baseEnthusiasmLevel?: number;
};

function Hello({name, baseEnthusiasmLevel = 0}: Props) {
  const [enthusiasmLevel, setEnthusiasmLevel] = useState(
    baseEnthusiasmLevel,
  );

  const onIncrement = () =>
    setEnthusiasmLevel(enthusiasmLevel + 1);
  const onDecrement = () =>
    setEnthusiasmLevel(
      enthusiasmLevel > 0 ? enthusiasmLevel - 1 : 0,
    );

  const getExclamationMarks = (numChars: number) =>
    numChars > 0 ? Array(numChars + 1).join('!') : '';

  return (
    <View style={styles.container}>
      <Text style={styles.greeting}>
        Hello {name}
        {getExclamationMarks(enthusiasmLevel)}
      </Text>
      <View>
        <Button
          title="Increase enthusiasm"
          accessibilityLabel="increment"
          onPress={onIncrement}
          color="blue"
        />
        <Button
          title="Decrease enthusiasm"
          accessibilityLabel="decrement"
          onPress={onDecrement}
          color="red"
        />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  greeting: {
    fontSize: 20,
    fontWeight: 'bold',
    margin: 16,
  },
});

export default Hello;
```

您可以在 [TypeScript 遊樂場][tsplay] 中進一步探索語法。

## 實用資源推薦

- [TypeScript 手冊][ts-handbook]
- [React 的 TypeScript 文檔][react-ts]
- [React + TypeScript 速查表][cheat] 提供了關於如何搭配使用 React 與 TypeScript 的優秀概述

## 在 TypeScript 中使用自訂路徑別名

要讓自訂路徑別名在 TypeScript 中生效，需同時配置 Babel 和 TypeScript 的路徑別名。方法如下：

1. 編輯 `tsconfig.json` 以設定 [自訂路徑映射][path-map]。將 `src` 根目錄下的內容設為無前綴路徑即可訪問，並允許透過 `tests/File.tsx` 訪問任何測試檔案：

```diff
{
-  "extends": "@react-native/typescript-config"
+  "extends": "@react-native/typescript-config",
+  "compilerOptions": {
+    "baseUrl": ".",
+    "paths": {
+      "*": ["src/*"],
+      "tests": ["tests/*"],
+      "@components/*": ["src/components/*"],
+    },
+  }
}
```

2. 將 [`babel-plugin-module-resolver`][bpmr] 作為開發依賴添加到專案中：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm install --save-dev babel-plugin-module-resolver
```

</TabItem>
<TabItem value="yarn">

```shell
yarn add --dev babel-plugin-module-resolver
```

</TabItem>
</Tabs>

3. 最後，配置您的 `babel.config.js`（請注意，`babel.config.js` 的語法與 `tsconfig.json` 不同）：

```diff
{
   presets: ['module:metro-react-native-babel-preset'],
+  plugins: [
+    [
+       'module-resolver',
+       {
+         root: ['./src'],
+         extensions: ['.ios.js', '.android.js', '.js', '.ts', '.tsx', '.json'],
+         alias: {
+           tests: ['./tests/'],
+           "@components": "./src/components",
+         }
+       }
+    ]
+  ]
}
```

[react-ts]: https://react.dev/learn/typescript

[ts]: https://www.typescriptlang.org/

[flow]: https://flow.org

[ts-template]: https://github.com/react-native-community/react-native-template-typescript

[babel]: /docs/javascript-environment#javascript-syntax-transformers

[babel-7-caveats]: https://babeljs.io/docs/en/next/babel-plugin-transform-typescript

[cheat]: https://github.com/typescript-cheatsheets/react-typescript-cheatsheet#reacttypescript-cheatsheets

[ts-handbook]: https://www.typescriptlang.org/docs/handbook/intro.html

[path-map]: https://www.typescriptlang.org/docs/handbook/module-resolution.html#path-mapping

[bpmr]: https://github.com/tleunen/babel-plugin-module-resolver

[expo]: https://expo.io

[ignite]: https://github.com/infinitered/ignite

[tsplay]: https://www.typescriptlang.org/play?strictNullChecks=false&jsx=3#code/JYWwDg9gTgLgBAJQKYEMDG8BmUIjgcilQ3wG4BYAKFEljgG8AhAVxhggDsAaOAZRgCeAGyS8AFkiQweAFSQAPaXABqwJAHcAvnGy4CRdDAC0HFDGAA3JGSpUFteILBI4ABRxgAznAC8DKnBwpiBIAFxwnjBQwBwA5hSUgQBGKJ5IAKIcMGLMnsCpIAAySFZCAPzhHMwgSUhQCZq2lGickXAAEkhCQhDhyIYAdABiAMIAPO4QXgB8vnAAFPRBKCE8KWmZ2bn5nkUlXXMADHCaAJS+s-QBcC0cbQDaSFk5eQXFpTxpMJsvO3ulAF05v0MANcqIYGYkPN1hlnts3vshKcEtdbm1OABJDhoIghLJzebnHyzL4-BG7d5deZPLavSlIuAAajgAEYUWjWvBOAARJC4pD4+B+IkXCJScn0-7U2m-RGlOCzY5lOCyinSoRwIxsuDhQ4cyicu7wWIS+RoIQrMzATgAWRQUAA1t4RVUQCMxA7PJVqrUoMTZm6PV7FXBlXAAIJQKAoATzIOeqDeFnsgYAKwgMXm+AAhPhzuF8DZDYk4EQYMwoBwFtdAmNVBoIoIRD56JFhEhPANbpCYnVNNNa4E4GM5Iomx3W+2RF3YkQpDFYgOh8OOl0evR8ARGqXV4F6MEkDu98P6KbvubLSBrXaHc6afCpVTkce92MAPRjmCD3fD+tqdQfxPOsWDYTgVz3cwYBbAAibEBVSFw1SlGCINXdA0E7PIkmAIRgEEQoUFqIQfBgmIBSFVDfxPTh3Cw1ssRxPFaVfYCbggHooFIpIhGYJAqLY98gOAsZQPYDg0OHKDYL5BC0lVR8-gEti4AwrDgBwvCCKIrpSIAE35ZismUtjaKITxPAYjhZKMmBWOAlpONIog9JMvchIgj8G0AocvIA4SDU0VFmi5CcZzmfgO3ESQYG7AwYGhK5Sx7FA+ygcIktXTARHkcJWS4IcUDw2IOExBKQG9OAYMwrI6hggrfzTXJzEwAQRk4BKsnCaraTq65NAawI5xixcMqHTAOt4YAAC8wjgAAmQ5BuHCasgAdSQYBYjEGBCySDi9PwZbAmvKBYhiPKADZloGqgzmC+xoHgAzMBQZghHgTpuggBIgA