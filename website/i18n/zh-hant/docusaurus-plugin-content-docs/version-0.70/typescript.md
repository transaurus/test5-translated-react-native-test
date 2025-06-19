---
id: typescript
title: Using TypeScript
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

[TypeScript][ts] is a language which extends JavaScript by adding type definitions, much like [Flow][flow]. While React Native is built in Flow, it supports both TypeScript _and_ Flow by default.

## 開始使用 TypeScript

如果您要開始一個新專案，有幾種不同的方式可以開始。

您可以使用 [TypeScript 模板][ts-template]：

```shell
npx react-native init MyApp --template react-native-template-typescript
```

:::note

如果上述命令失敗，可能是因為您的系統上安裝了舊版本的 `react-native` 或 `react-native-cli`。要解決此問題，請嘗試解除安裝 CLI：

- `npm uninstall -g react-native-cli` 或 `yarn global remove react-native-cli`

然後再次執行 `npx` 命令。
或者，您也可以使用以下命令來開始使用您的模板。

- `npx react-native --ignore-existing init MyApp --template react-native-template-typescript`

:::

您可以使用 [Expo][expo]，它維護了 TypeScript 模板，或者在專案中添加 `.ts` 或 `.tsx` 文件時會提示您自動安裝和配置 TypeScript：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npx create-expo-app --template
```

</TabItem>
<TabItem value="yarn">

```shell
yarn create expo-app --template
```

</TabItem>
</Tabs>

或者您可以使用 [Ignite][ignite]，它也有一個 TypeScript 模板：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm install -g ignite-cli
ignite new MyTSProject
```

</TabItem>
<TabItem value="yarn">

```shell
yarn global add ignite-cli
ignite new MyTSProject
```

</TabItem>
</Tabs>

## 在現有專案中添加 TypeScript

1. 將 TypeScript 以及 React Native 和 Jest 的類型定義添加到您的專案中。

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm install -D typescript @types/jest @types/react @types/react-native @types/react-test-renderer @tsconfig/react-native
```

</TabItem>
<TabItem value="yarn">

```shell
yarn add -D typescript @types/jest @types/react @types/react-native @types/react-test-renderer @tsconfig/react-native
```

</TabItem>
</Tabs>

2. 添加一個 TypeScript 配置文件。在您的專案根目錄中創建一個 `tsconfig.json`：

```json
{
  "extends": "@tsconfig/react-native/tsconfig.json"
}
```

3. 創建一個 `jest.config.js` 文件來配置 Jest 以使用 TypeScript

```js
module.exports = {
  preset: 'react-native',
  moduleFileExtensions: [
    'ts',
    'tsx',
    'js',
    'jsx',
    'json',
    'node',
  ],
};
```

4. 將一個 JavaScript 文件重命名為 `*.tsx`

> 您應該保持 `./index.js` 入口文件不變，否則在打包生產版本時可能會遇到問題。

5. 運行 `yarn tsc` 來對您的新 TypeScript 文件進行類型檢查。

## TypeScript 和 React Native 的工作原理

開箱即用，將您的文件轉換為 JavaScript 是通過與非 TypeScript React Native 專案相同的 [Babel 基礎設施][babel] 進行的。我們建議您僅使用 TypeScript 編譯器進行類型檢查。如果您有現有的 TypeScript 代碼要移植到 React Native，使用 Babel 而不是 TypeScript 會有 [一兩個注意事項][babel-7-caveats]。

## React Native + TypeScript 是什麼樣子

You can provide an interface for a React Component's [Props](props) and [State](state) via `React.Component<Props, State>` which will provide type-checking and editor auto-completing when working with that component in JSX.

```tsx title="components/Hello.tsx"
import React from 'react';
import {Button, StyleSheet, Text, View} from 'react-native';

export type Props = {
  name: string;
  baseEnthusiasmLevel?: number;
};

const Hello: React.FC<Props> = ({
  name,
  baseEnthusiasmLevel = 0,
}) => {
  const [enthusiasmLevel, setEnthusiasmLevel] = React.useState(
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
};

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

## 哪裡可以找到有用的建議

- [TypeScript 手冊][ts-handbook]
- [React 關於 TypeScript 的文檔][react-ts]
- [React + TypeScript 速查表][cheat] 提供了關於如何將 React 與 TypeScript 結合使用的良好概述

## 在 TypeScript 中使用自定義路徑別名

要在 TypeScript 中使用自定義路徑別名，您需要設置路徑別名以同時適用於 Babel 和 TypeScript。方法如下：

1. 編輯您的 `tsconfig.json` 以設置 [自定義路徑映射][path-map]。將 `src` 根目錄中的任何內容設置為無需前置路徑引用即可使用，並允許通過 `tests/File.tsx` 訪問任何測試文件：

```diff {2-7}
    "target": "esnext",
+     "baseUrl": ".",
+     "paths": {
+       "*": ["src/*"],
+       "tests": ["tests/*"],
+       "@components/*": ["src/components/*"],
+     },
    }
```

2. 將 [`babel-plugin-module-resolver`][bpmr] 作為開發依賴添加到你的專案中：

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

3. 最後，配置你的 `babel.config.js`（注意 `babel.config.js` 的語法與 `tsconfig.json` 不同）：

```diff {3-13}
{
  plugins: [
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
  ]
}
```

[react-ts]: https://reactjs.org/docs/static-type-checking.html#typescript

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