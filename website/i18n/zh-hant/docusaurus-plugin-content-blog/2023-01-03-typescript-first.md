---
title: First-class Support for TypeScript
authors: [lunaleaps, NickGerleman]
tags: [typescript, engineering]
date: 2023-01-03
---

# TypeScript 的一級支援

隨著 0.71 版本的發布，React Native 透過以下變更來提升 TypeScript 的使用體驗：

- [新的應用程式模板預設為 TypeScript](/blog/2023/01/03/typescript-first#new-app-template-is-typescript-by-default)
- [React Native 內建 TypeScript 宣告](/blog/2023/01/03/typescript-first#declarations-shipped-with-react-native)
- [React Native 文件以 TypeScript 為優先](/blog/2023/01/03/typescript-first#documentation-is-typescript-first)

本文將說明這些變更對 TypeScript 或 Flow 使用者的意義。

<!--truncate-->

## 新的應用程式模板預設為 TypeScript

從 0.71 版本開始，當你透過 React Native CLI 建立新的 React Native 應用程式時，預設會得到一個 TypeScript 應用程式！

```shell
npx react-native init My71App --version 0.71.0
```

![螢幕截圖顯示由 React Native CLI 生成的新應用程式在 iPhone 模擬器中運行。旁邊是 Visual Studio Code 編輯器的截圖，打開「App.tsx」以說明它正在運行 TypeScript 檔案。](/blog/assets/typescript-first-new-app.png)

新生成應用程式的起點將是 `App.tsx` 而非 `App.js`——完全使用 TypeScript 類型。新專案已預設設定好 `tsconfig.json`，因此你的 IDE 將立即協助你撰寫類型化的程式碼！

## React Native 內建 TypeScript 宣告

0.71 是第一個內建 TypeScript (TS) 宣告的版本。

在此之前，React Native 的 TypeScript 宣告由 [`@types/react-native`](https://www.npmjs.com/package/@types/react-native) 提供，該套件託管於 [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) 儲存庫。將 TypeScript 類型與 React Native 原始碼放在一起的決定是為了提高正確性和維護性。

`@types/react-native` 僅為穩定版本提供類型。這意味著如果你曾想在 TypeScript 中使用 React Native 的預發布版本，你必須使用舊版本的類型，而這些類型可能不準確。發布 `@types/react-native` 也容易出錯。發布會落後於 React Native 的發布，且過程需要手動檢查 React Native 公共 API 的類型變更並更新 TS 宣告以匹配。

將 TS 類型與 React Native 原始碼放在一起後，TS 類型的可見性和所有權更高。我們的團隊正在積極開發工具以保持 Flow 和 TS 之間的一致性。

此變更也減少了 React Native 使用者需要管理的依賴項。升級到 0.71 或更高版本時，你可以移除 `@types/react-native` 作為依賴項。[請參考新應用程式模板以了解如何設定 TypeScript 支援。](https://github.com/facebook/react-native/blob/main/template/tsconfig.json)

我們計劃在 0.73 及更高版本中棄用 `@types/react-native`。具體而言，這意味著：

- `@types/react-native` tracking React Native versions 0.71 and 0.72 will be released. They will be identical to the types in React Native on the relevant release branches.
- For React Native 0.73 and onward, TS types will only be available from React Native.

### 如何遷移

請盡快遷移到新的內建類型。以下是根據你的需求提供的遷移詳細資訊。

#### 應用程式維護者

Once you upgrade to React Native >= 0.71, you can remove the `@types/react-native` from your `devDependency`.

:::note

若您因使用的函式庫將 `@types/react-native` 列為 `peerDependency` 而出現警告，請向該函式庫提交 issue 或 PR，要求改用[可選的 peerDependencies](https://docs.npmjs.com/cli/v7/configuring-npm/package-json#peerdependenciesmeta)，目前可暫時忽略此警告。

:::

#### 函式庫維護者

Libraries that target versions of React Native below 0.71 may use a `peerDependency` of `@types/react-native` to typecheck against the apps version of typings. This dependency should be marked as optional in [`peerDependenciesMeta`](https://docs.npmjs.com/cli/v7/configuring-npm/package-json#peerdependenciesmeta) so that the typings are not required for users without TypeScript or for 0.71 users where typings are built-in.

#### Maintainer of TypeScript declarations that depend on `@types/react-native`

請查閱 [0.71 版本引入的重大變更](https://github.com/facebook/react-native/blob/main/CHANGELOG.md)，確認您是否已準備好遷移。

### 如果我使用 Flow 怎麼辦？

Flow 使用者仍可繼續對目標版本為 0.71+ 的應用程式進行型別檢查，但新模板中已不再預設包含 Flow 的配置邏輯。

以往 Flow 使用者會透過合併新應用模板中的 `.flowconfig` 並手動更新 `flow-bin` 來升級 React Native 的 Flow 型別定義。新應用模板已不再提供 `.flowconfig`，但 [React Native 代碼庫中仍保留了一份](https://github.com/facebook/react-native/blob/main/.flowconfig)可供參考使用。

若需新建一個使用 Flow 的 React Native 應用程式，可參考 [0.70 版本的新應用模板](https://github.com/facebook/react-native/tree/0.70-stable/template)。

### 如果發現 TypeScript 型別定義有錯誤怎麼辦？

無論您使用的是內建 TS 型別還是 `@types/react-native`，若發現錯誤請同時向 [React Native](https://github.com/facebook/react-native) 和 [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) 代碼庫提交 PR。若不知如何修正，請在 React Native 代碼庫提交 GitHub issue 並標註 [@lunaleaps](https://github.com/lunaleaps)。

## 文件優先支援 TypeScript

為確保一致的 TypeScript 使用體驗，我們已對 React Native 文件進行多項更新，將 TypeScript 設為新的預設語言。

程式碼範例現在支援內嵌 TypeScript，並已更新超過 170 個互動式範例以符合新模板的 linting、格式化和型別檢查要求。大多數範例同時適用於 TypeScript 和 JavaScript，若有不兼容之處，您可以切換查看不同語言的範例。

若發現錯誤或有改進建議，請記住網站也是開源的，我們非常歡迎您的 PR！

## 感謝 React Native TypeScript 社群！

最後，我們要感謝多年來社群為確保 React Native 開發者能順利使用 TypeScript 所付出的努力。

We want to thank all the [contributors](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react-native/index.d.ts#L3) that have been maintaining `@types/react-native` since [2015](https://github.com/DefinitelyTyped/DefinitelyTyped/commit/efce0c25ec532a4651859f10eda49e97a5716a42)! We see the effort and care you all have put into making sure React Native users have the best experience.

感謝 [@acoates](https://github.com/acoates)、[@eps1lon](https://github.com/eps1lon)、[@kelset](https://github.com/kelset)、[@tido64](https://github.com/tido64)、[@Titozzz](https://github.com/Titozzz) 和 [@ZihanChen-MSFT](https://github.com/ZihanChen-MSFT) 的協助，包括諮詢、提問、溝通與審查變更，將 TypeScript 類型定義移入 React Native 核心。

Similarly, we want to thank the [maintainers of `react-native-template-typescript`](https://github.com/react-native-community/react-native-template-typescript/graphs/contributors) for supporting the TypeScript experience for new app development in React Native since day one.

我們期待未來能在 React Native 代碼庫中更直接地協作，持續改善 React Native 的開發者體驗！