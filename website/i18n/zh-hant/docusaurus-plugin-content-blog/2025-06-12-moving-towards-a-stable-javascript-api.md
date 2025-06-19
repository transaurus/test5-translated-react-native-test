---
title: 'Moving Towards a Stable JavaScript API (New Changes in 0.80)'
authors: [huntie, iwoplaza, jpiasecki, coado]
tags: [announcement]
date: 2025-06-12T16:00
---

在 React Native 0.80 版本中，我們將對 JavaScript API 進行兩項重大變更：棄用深度導入(deprecation of deep imports)與導入全新的嚴格 TypeScript API(Strict TypeScript API)。這些變更是為了明確定義 React Native 的公共 API 並為使用者與框架提供可靠的型別安全(type safety)。

**重點摘要：**

- **Deep imports deprecation**: From 0.80, we're introducing deprecation warnings for deep imports from the `react-native` package.
- **Opt-in Strict TypeScript API**: We are moving to from-source TypeScript types and a new public API baseline under TypeScript. These enable stronger and more futureproof type accuracy, and will be a one-time breaking change. [Opt in](/blog/2025/06/12/moving-towards-a-stable-javascript-api#strict-typescript-api) via `compilerOptions` in your project's `tsconfig.json`.
- We'll work with the community over time to ensure that these changes work for everyone, before enabling the Strict TypeScript API by default in a future React Native release.

<!--truncate-->

## 變更內容與原因

我們正致力於改進並穩定 React Native 的公共 JavaScript API——也就是當你導入 `'react-native'` 時所獲得的內容。

長期以來，我們對此僅有近似實現。React Native 原始碼使用 [Flow](https://flow.org/) 撰寫，但開源社群早已轉向 TypeScript，這也是公共 API 被使用與驗證相容性的方式。我們的型別定義一直由社群（熱心）[貢獻](https://www.npmjs.com/package/@types/react-native)，後來被合併至程式碼庫中。然而這些型別依賴人工維護且缺乏自動化工具，導致正確性缺口。

此外，我們的公共 JS API 在模組邊界方面定義模糊——例如應用程式碼可以存取內部 `'react-native/Libraries/'` 的深度導入路徑，但這些內部實作經常會隨著更新而變動。

在 0.80 版本中，我們透過棄用深度導入並引入可選用的新型別生成基準（稱為**嚴格 TypeScript API**）來解決這些問題。這將為未來提供穩定的 React Native API 奠定基礎。

## Deprecating deep imports from `react-native`

我們對 API 的主要變更是棄用深度導入（參見 [RFC](https://github.com/react-native-community/discussions-and-proposals/pull/894)），並在 ESLint 與 JS 控制台發出警告。數值與型別的深度導入應更新為 `react-native` 的根導入(root import)。

```js title=""
// Before - import from subpath
import {Alert} from 'react-native/Libraries/Alert/Alert';

// After - import from `react-native`
import {Alert} from 'react-native';
```

此變更將 JavaScript API 的總表面積縮減為固定可控制的匯出集合，以便在未來版本中實現穩定。我們預計在 0.82 版本移除這些導入路徑。

:::info[API feedback]

Some APIs are not exported at root, and will become unavailable without deep imports. We have an **[open feedback thread](https://github.com/react-native-community/discussions-and-proposals/discussions/893)** and will be working with the community to finalize the exports in our public API. Please share your feedback!

:::

**選擇退出**

請注意，我們計劃在未來版本中從 React Native API 移除深度導入功能，建議將這些導入更新為根路徑導入。

<details>
<summary>**Opting out of warnings**</summary>

#### ESLint

Disable the `no-deep-imports` rule using `overrides`.

<!-- prettier-ignore -->
```js title=".eslintrc.js"
  overrides: [
    {
      files: ['*.js', '*.jsx', '*.ts', '*.tsx'],
      rules: {
        '@react-native/no-deep-imports': 0,
      },
    },
  ]
```

#### Console warnings

Pass the `disableDeepImportWarnings` option to `@react-native/babel-preset`.

<!-- prettier-ignore -->
```js title="babel.config.js"
module.exports = {
  presets: [
    ['module:@react-native/babel-preset', {disableDeepImportWarnings: true}]
  ],
};
```

Restart your app with `--reset-cache` to clear the Metro cache.

```sh title=""
npx @react-native-community/cli start --reset-cache
```

</details>

<details>
<summary>**Opting out of warnings (Expo)**</summary>

#### ESLint

Disable the `no-deep-imports` rule using `overrides`.

<!-- prettier-ignore -->
```js title=".eslintrc.js"
overrides: [
  {
    files: ['*.js', '*.jsx', '*.ts', '*.tsx'],
    rules: {
      '@react-native/no-deep-imports': 0,
    },
  },
];
```

#### Console warnings

Pass the `disableDeepImportWarnings` option to `babel-preset-expo`.

<!-- prettier-ignore -->
```js title="babel.config.js"
module.exports = function (api) {
  api.cache(true);
  return {
    presets: [['babel-preset-expo', {disableDeepImportWarnings: true}]],
  };
};
```

Restart your app with `--clear` to clear the Metro cache.

```sh name=""
npx expo start --clear
```

</details>

## 嚴格 TypeScript API（可選用）

嚴格 TypeScript API 是 `react-native` 套件中的一組新型別定義，可透過 `tsconfig.json` 選擇啟用。這些新型別將與現有 TS 型別並存，您可自行決定遷移時機。

新型別具有以下特性：

1. **直接從原始碼生成** — 提升覆蓋率與正確性，您將獲得更有保障的相容性。
2. **限制於 `react-native` 的索引檔案** — 更嚴格定義公共 API，意味著內部檔案變更時不會破壞 API。

當社群準備就緒時，嚴格 TypeScript API 將成為未來的預設 API — 與深層導入移除同步進行。這意味著現在開始啟用是**明智之舉**，您將為 React Native 未來穩定的 JS API 做好準備。

```json title="tsconfig.json"
{
  "extends": "@react-native/typescript-config",
  "compilerOptions": {
    ...
    "customConditions": ["react-native-strict-api"]
  }
}
```

:::note[Under the hood]

This will instruct TypeScript to resolve `react-native` types from our new [`types_generated/`](https://www.npmjs.com/package/react-native?activeTab=code) dir, instead of the previous [`types/`](https://www.npmjs.com/package/react-native?activeTab=code) dir (manually maintained). No restart of TypeScript or your editor is required.

:::

### 重大變更：禁止深層導入

如上所述，嚴格 TypeScript API 下的類型現在僅能從主 `'react-native'` 導入路徑解析，依循前述棄用政策強制執行[套件封裝](/blog/2023/06/21/package-exports-support)。

```tsx
// Before - import from subpath
import {Alert} from 'react-native/Libraries/Alert/Alert';

// After - MUST import from `react-native`
import {Alert} from 'react-native';
```

:::tip[關鍵優勢]

我們已將公共 API 範圍限定於 React Native 精心維護的 `index.js` 檔案輸出。這意味著程式碼庫其他位置的檔案變更將不再成為破壞性變更。

:::

### 重大變更：部分類型名稱/結構已調整

類型現在改為從原始碼生成，而非手動維護。在此過程中：

- 我們已修正社群貢獻類型長期累積的差異 — 同時提升原始碼的類型覆蓋率。
- 針對可簡化或降低歧義的類型，我們刻意更新了部分類型名稱與結構。

:::tip[關鍵優勢]

由於類型現在直接從 React Native 原始碼生成，您可以確信類型檢查器**始終準確**反映特定 `react-native` 版本。

:::

#### 範例：更嚴格的導出符號

The `Linking` API is now a single `interface`, rather than two exports. This follows for a number of other APIs ([see docs](/docs/strict-typescript-api)).

```tsx
// Before
import {Linking, LinkingStatic} from 'react-native';

function foo(linking: LinkingStatic) {}
foo(Linking);

// After
import {Linking} from 'react-native';

function foo(linking: Linking) {}
foo(Linking);
```

#### 範例：修正/更完整的類型

過往手動類型定義可能產生類型缺口。透過 Flow → TypeScript 的自動生成機制，這些缺口已不復存在（且在原始碼層級受益於 Flow 對跨平台程式碼的額外類型驗證）。

```tsx
import {Dimensions} from 'react-native';

// Before - Type error
// After - number | undefined
const {densityDpi} = Dimensions.get();
```

### 其他重大變更

請參閱文件中的[專用指南](/docs/strict-typescript-api)，其中詳述所有類型破壞性變更及程式碼遷移方法。

## 推行時程

我們理解任何 React Native 的破壞性變更都需要開發者花時間調整應用程式。

#### 現階段 — 選擇性啟用 (0.80)

The `"react-native-strict-api"` opt-in is stable in the 0.80 release.

- 這是一次性遷移。我們預計應用程式與函式庫會在未來幾次版本更新中逐步啟用。
- 無論採用何種模式，您的應用程式在執行時期都不會受影響 — 此變更僅涉及 TypeScript 分析層面。
- **此外**，我們將透過[專用反饋討論串](https://github.com/react-native-community/discussions-and-proposals/discussions/893)收集關於缺失 API 的意見。

:::tip[推薦]

嚴格 TypeScript API 將在未來成為我們的預設 API。

如果您有時間，現在就值得在您的 `tsconfig.json` 中測試這個選擇加入功能，以確保您的應用程式或函式庫能夠適應未來。這將立即評估在嚴格 API 下您的應用程式中是否有任何類型錯誤被引入。**可能完全沒有(!)** — 如果是這樣，您就可以放心了。

:::

#### 未來 — 預設啟用嚴格 TypeScript API

未來，我們將要求所有程式碼庫使用我們的嚴格 API，並移除舊有的類型定義。

這個時間表將基於社群的回饋。至少在接下來的兩個 React Native 版本中，嚴格 API 將保持為選擇加入功能。

## 常見問題

<details>
<summary>
**I'm using subpath imports today. What should I do?**
</summary>

Please migrate to the root `'react-native'` import path.

- Subpath imports (e.g. `'react-native/Libraries/Alert/Alert'`) are becoming private APIs. Without preventing access to implementation files inside React Native, we can’t offer a stable JavaScript API.
- We want our deprecation warnings to motivate community feedback, which can be raised via our [centralized discussion thread](https://github.com/react-native-community/discussions-and-proposals/discussions/893), if you believe we are not exposing code paths that are crucial for your app. Where justified, we may promote APIs to the index export.

</details>

<details>
<summary>
**I'm a library maintainer. How does this change impact me?**
</summary>

Both apps and libraries can opt in at their own pace, since `tsconfig.json` will only affect the immediate codebase.

- Typically, `node_modules` is excluded from validation by the TypeScript server in a React Native project. Therefore, your package's exported type definitions are the source of truth.

**💡 We want feedback!** As with changed subpath imports, if you encounter any integration issues with the Strict API, please let us know [on GitHub](https://github.com/react-native-community/discussions-and-proposals/discussions/893).

</details>

<details>
<summary>
**Does this guarantee a final API for React Native yet?**
</summary>

Sadly, not yet. In 0.80, we've made a tooling investment so that React Native's existing JS API baseline can be accurately consumed via TypeScript — enabling future stable changes. We're formalizing the existing API you know and love.

In the future, we will take action to finalise the APIs we currently offer in core — across each language surface. API changes will be communicated via RFCs/announcements, and typically a deprecation cycle.

</details>

<details>
<summary>
**Why isn't React Native written in TypeScript?**
</summary>

React Native is core infrastructure at Meta. We test every merged change across our Family of Apps, before they hit general open source availability.

At this scale and sensitivity, correctness matters. The bottom line is that Flow offers us greater performance and greater strictness than TypeScript, including specific [multi-platform support for React Native](https://flow.org/en/docs/react/multiplatform/).

</details>

## 致謝

這些變更得以實現，要感謝 [Iwo Plaza](https://x.com/iwoplaza)、[Jakub Piasecki](https://x.com/breskin67)、[Dawid Małecki](https://github.com/coado)、[Alex Hunt](https://x.com/huntie) 和 [Riccardo Cipolleschi](https://x.com/CipolleschiR)。

同時也感謝 [Pieter Vanderwerff](https://github.com/pieterv)、[Rubén Norte](https://github.com/rubennorte) 和 [Rob Hogan](https://x.com/robjhogan) 的額外協助與意見。

:::note[Learn more]

<div style={{display: 'flex'}}>
<p style={{flex: 1, marginRight: 40, marginBottom: 0}}>
<strong style={{ display: 'block', marginTop: 8, marginBottom: 8 }}>Watch the talk!</strong>
<span style={{ display: 'block', marginBottom: 8 }}>We shared a deep dive into our motivations and the work behind the Strict TypeScript API at <strong>App.js 2025</strong>.</span>
**[View on YouTube](https://www.youtube.com/live/UTaJlqhTk2g?si=SDRmj80kss7hXuGG&t=6520)**
</p>
<img
  src="/blog/assets/0.80-js-stable-api-appjs.jpg"
  style={{ flexShrink: 0, width: '200px', aspectRatio: '16/9', objectFit: 'contain' }}
  alt="App.js 2025 Talk"
/>
</div>

:::