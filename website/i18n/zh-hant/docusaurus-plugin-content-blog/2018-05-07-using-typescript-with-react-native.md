---
title: Using TypeScript with React Native
author: Ash Furrow
authorTitle: Software Engineer at Artsy
authorURL: 'https://github.com/ashfurrow'
authorImageURL: 'https://avatars2.githubusercontent.com/u/498212?s=460&v=4'
authorTwitter: ashfurrow
tags: [engineering]
---

JavaScript！我們都愛它。但有些人也喜歡[型別](https://en.wikipedia.org/wiki/Data_type)。幸運的是，我們有選項可以為JavaScript添加更強的型別。我最喜歡的是[TypeScript](https://www.typescriptlang.org)，但React Native原生支援[Flow](https://flow.org)。選擇哪一種取決於個人偏好，它們各自有不同的方式來為JavaScript添加型別的魔法。今天，我們將探討如何在React Native應用程式中使用TypeScript。

這篇文章使用微軟的[TypeScript-React-Native-Starter](https://github.com/Microsoft/TypeScript-React-Native-Starter)倉庫作為指南。

**更新**：自從這篇部落格文章撰寫以來，事情變得更加簡單。你可以用以下一個命令取代文章中描述的所有設定步驟：

```sh
npx react-native init MyAwesomeProject --template react-native-template-typescript
```

However, there _are_ some limitations to Babel's TypeScript support, which the blog post above goes into in detail. The steps outlined in _this_ post still work, and Artsy is still using [react-native-typescript-transformer](https://github.com/ds300/react-native-typescript-transformer) in production, but the fastest way to get up and running with React Native and TypeScript is using the above command. You can always switch later if you have to.

無論如何，祝你玩得開心！原始的部落格文章繼續如下。

## 先決條件

由於你可能在幾個不同的平台上開發，針對幾種不同類型的設備，基本設定可能會比較複雜。你應該首先確保你能夠運行一個普通的React Native應用程式而不使用TypeScript。按照[React Native網站上的指示開始](/docs/getting-started)。當你成功部署到設備或模擬器時，你就可以開始一個TypeScript React Native應用程式了。

你還需要[Node.js](https://nodejs.org/en/)、[npm](https://www.npmjs.com)和[Yarn](https://yarnpkg.com/lang/en)。

## 初始化

一旦你嘗試過搭建一個普通的React Native專案，你就可以開始添加TypeScript了。讓我們繼續進行。

```sh
react-native init MyAwesomeProject
cd MyAwesomeProject
```

## 添加TypeScript

下一步是將TypeScript添加到你的專案中。以下命令將：

- add TypeScript to your project
- add [React Native TypeScript Transformer](https://github.com/ds300/react-native-typescript-transformer) to your project
- initialize an empty TypeScript config file, which we'll configure next
- add an empty React Native TypeScript Transformer config file, which we'll configure next
- adds [typings](https://github.com/DefinitelyTyped/DefinitelyTyped) for React and React Native

好的，讓我們繼續運行這些命令。

```sh
yarn add --dev typescript
yarn add --dev react-native-typescript-transformer
yarn tsc --init --pretty --jsx react
touch rn-cli.config.js
yarn add --dev @types/react @types/react-native
```

The `tsconfig.json` file contains all the settings for the TypeScript compiler. The defaults created by the command above are mostly fine, but open the file and uncomment the following line:

```js
{
  /* Search the config file for the following line and uncomment it. */
  // "allowSyntheticDefaultImports": true,  /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */
}
```

The `rn-cli.config.js` contains the settings for the React Native TypeScript Transformer. Open it and add the following:

```js
module.exports = {
  getTransformModulePath() {
    return require.resolve('react-native-typescript-transformer');
  },
  getSourceExts() {
    return ['ts', 'tsx'];
  },
};
```

## 遷移到TypeScript

將生成的`App.js`和`__tests_/App.js`文件重命名為`App.tsx`。`index.js`需要使用`.js`副檔名。所有新文件應該使用`.tsx`副檔名（或者`.ts`如果文件不包含任何JSX）。

如果你現在嘗試運行應用程式，你會得到一個錯誤，例如`object prototype may only be an object or null`。這是由於在同一行中未能從React導入默認導出以及命名導出所導致的。打開`App.tsx`並修改文件頂部的導入：

```diff
-import React, { Component } from 'react';
+import React from 'react'
+import { Component } from 'react';
```

這部分與 Babel 和 TypeScript 在 CommonJS 模組的交互方式差異有關。未來兩者將穩定採用相同的行為模式。

至此，您應該能夠運行 React Native 應用程式。

## 添加 TypeScript 測試基礎設施

React Native 內建 [Jest](https://github.com/facebook/jest)，若要為 TypeScript 編寫的 React Native 應用程式進行測試，我們需將 [ts-jest](https://www.npmjs.com/package/ts-jest) 加入 `devDependencies`。

```sh
yarn add --dev ts-jest
```

接著開啟 `package.json` 並將 `jest` 欄位替換為以下內容：

```js
{
  "jest": {
    "preset": "react-native",
    "moduleFileExtensions": [
      "ts",
      "tsx",
      "js"
    ],
    "transform": {
      "^.+\\.(js)$": "<rootDir>/node_modules/babel-jest",
      "\\.(ts|tsx)$": "<rootDir>/node_modules/ts-jest/preprocessor.js"
    },
    "testRegex": "(/__tests__/.*|\\.(test|spec))\\.(ts|tsx|js)$",
    "testPathIgnorePatterns": [
      "\\.snap$",
      "<rootDir>/node_modules/"
    ],
    "cacheDirectory": ".jest/cache"
  }
}
```

This will configure Jest to run `.ts` and `.tsx` files with `ts-jest`.

## 安裝依賴項的型別宣告

為獲得 TypeScript 的最佳開發體驗，我們需要型別檢查器理解依賴項的結構與 API。部分函式庫會在發佈套件時包含 `.d.ts` 檔案（型別宣告/型別定義檔案），這些檔案能描述底層 JavaScript 的結構。其他函式庫則需明確安裝 `@types/` npm 作用域下的對應套件。

例如，此處我們需要 Jest、React、React Native 及 React Test Renderer 的型別定義。

```ts
yarn add --dev @types/jest @types/react @types/react-native @types/react-test-renderer
```

我們將這些宣告檔案套件保存為 _開發_ 依賴項，因為這是僅在開發階段使用這些依賴的 React Native _應用程式_，而非運行時。若我們要發佈函式庫至 NPM，可能需要將部分型別依賴項設為常規依賴項。

您可查閱[此處了解更多關於取得 `.d.ts` 檔案](https://www.typescriptlang.org/docs/handbook/declaration-files/consumption.html)的資訊。

## 忽略更多檔案

在原始碼管控方面，您需開始忽略 `.jest` 資料夾。若使用 git，可直接在 `.gitignore` 檔案新增條目。

```config
# Jest
#
.jest/
```

作為檢查點，建議將檔案提交至版本控制系統。

```sh
git init
git add .gitignore # import to do this first, to ignore our files
git add .
git commit -am "Initial commit."
```

## 新增元件

讓我們為應用程式新增元件。現在建立一個 `Hello.tsx` 教學用元件，雖非實際開發中會撰寫的內容，但能展示如何在 React Native 中使用 TypeScript 處理非簡單案例。

建立 `components` 目錄並加入以下範例。

```ts
// components/Hello.tsx
import React from 'react';
import {Button, StyleSheet, Text, View} from 'react-native';

export interface Props {
  name: string;
  enthusiasmLevel?: number;
}

interface State {
  enthusiasmLevel: number;
}

export class Hello extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);

    if ((props.enthusiasmLevel || 0) <= 0) {
      throw new Error(
        'You could be a little more enthusiastic. :D',
      );
    }

    this.state = {
      enthusiasmLevel: props.enthusiasmLevel || 1,
    };
  }

  onIncrement = () =>
    this.setState({
      enthusiasmLevel: this.state.enthusiasmLevel + 1,
    });
  onDecrement = () =>
    this.setState({
      enthusiasmLevel: this.state.enthusiasmLevel - 1,
    });
  getExclamationMarks = (numChars: number) =>
    Array(numChars + 1).join('!');

  render() {
    return (
      <View style={styles.root}>
        <Text style={styles.greeting}>
          Hello{' '}
          {this.props.name +
            this.getExclamationMarks(this.state.enthusiasmLevel)}
        </Text>

        <View style={styles.buttons}>
          <View style={styles.button}>
            <Button
              title="-"
              onPress={this.onDecrement}
              accessibilityLabel="decrement"
              color="red"
            />
          </View>

          <View style={styles.button}>
            <Button
              title="+"
              onPress={this.onIncrement}
              accessibilityLabel="increment"
              color="blue"
            />
          </View>
        </View>
      </View>
    );
  }
}

// styles
const styles = StyleSheet.create({
  root: {
    alignItems: 'center',
    alignSelf: 'center',
  },
  buttons: {
    flexDirection: 'row',
    minHeight: 70,
    alignItems: 'stretch',
    alignSelf: 'center',
    borderWidth: 5,
  },
  button: {
    flex: 1,
    paddingVertical: 0,
  },
  greeting: {
    color: '#999',
    fontWeight: 'bold',
  },
});
```

哇！內容很多，讓我們逐步解析：

- Instead of rendering HTML elements like `div`, `span`, `h1`, etc., we're rendering components like `View` and `Button`. These are native components that work across different platforms.
- Styling is specified using the `StyleSheet.create` function that React Native gives us. React's stylesheets allow us to control our layout using Flexbox, and style using other constructs similar to those in CSS.

## 新增元件測試

現在我們有了元件，接著來測試它。

我們已安裝 Jest 作為測試執行器。接下來要為元件編寫快照測試，先安裝快照測試所需的附加套件：

```sh
yarn add --dev react-addons-test-utils
```

Now let's create a `__tests__` folder in the `components` directory and add a test for `Hello.tsx`:

```ts
// components/__tests__/Hello.tsx
import React from 'react';
import renderer from 'react-test-renderer';

import {Hello} from '@site/Hello';

it('renders correctly with defaults', () => {
  const button = renderer
    .create(<Hello name="World" enthusiasmLevel={1} />)
    .toJSON();
  expect(button).toMatchSnapshot();
});
```

首次執行測試時，它會建立渲染元件的快照並儲存於 `components/__tests__/__snapshots__/Hello.tsx.snap` 檔案中。當您修改元件時，需更新快照並檢查是否有非預期的變更。您可參閱[此處](https://facebook.github.io/jest/docs/en/tutorial-react-native.html)進一步了解如何測試 React Native 元件。

## 後續步驟

參考官方 [React 教學](https://reactjs.org/tutorial/tutorial.html)與狀態管理函式庫 [Redux](https://redux.js.org)，這些資源對開發 React Native 應用程式有所助益。此外，您也可了解 [ReactXP](https://microsoft.github.io/reactxp/)，這是一個完全以 TypeScript 編寫的元件庫，同時支援網頁版 React 與 React Native。

祝您在更安全的型別環境中享受 React Native 開發樂趣！