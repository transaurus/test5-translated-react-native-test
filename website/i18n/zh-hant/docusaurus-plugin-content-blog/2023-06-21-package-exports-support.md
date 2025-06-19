---
title: 'Package Exports Support in React Native'
authors: [huntie]
tags: [announcement, metro]
date: 2023-06-21
---

# React Native 中的套件匯出支援

隨著 [React Native 0.72](/blog/2023/06/21/0.72-metro-package-exports-symlinks) 的發布，我們的 JavaScript 建構工具 Metro 現在包含對 `package.json` 中 [`"exports"`](https://nodejs.org/docs/latest-v18.x/api/packages.html#exports) 欄位的測試版支援。當[啟用](/blog/2023/06/21/package-exports-support#enabling-package-exports-beta)後，它新增了以下功能：

- [React Native 專案將能直接與更多 npm 套件配合使用](/blog/2023/06/21/package-exports-support#for-app-developers)
- [套件定義其 API 並針對 React Native 的新能力](/blog/2023/06/21/package-exports-support#for-package-maintainers-preview)
- [套件解析的一些破壞性變更（在邊緣情況下）](/blog/2023/06/21/package-exports-support#breaking-changes)

在這篇文章中，我們將介紹套件匯出的運作方式，以及這些變更對身為 React Native 應用開發者或套件維護者的您意味著什麼。

<!-- truncate -->

## 什麼是套件匯出？

套件匯出功能於 Node.js 12.7.0 中引入，是 npm 套件指定**入口點**的現代方法 — 定義套件子路徑的外部導入映射及其應解析到的檔案。

支援 `"exports"` 改善了 React Native 專案與更廣泛 JavaScript 生態系統的協作（[目前約有 16.6k 個套件使用](https://github.com/search?q=path%3A%2A%2A%2Fpackage.json+%22%5C%22access%5C%22%3A+%5C%22public%5C%22%22+%22%5C%22exports%5C%22%22+NOT+path%3A%2A%2A%2Fnode_modules+NOT+is%3Afork+NOT+is%3Aarchived&type=code)），並為套件作者提供標準化功能集來開發跨平台套件以支援 React Native。

[`"exports"`](https://nodejs.org/docs/latest-v19.x/api/packages.html#exports) 可與 [`"main"`](https://nodejs.org/docs/latest-v19.x/api/packages.html#main) 並用或替代，置於 `package.json` 檔案中。

```json
{
  "name": "@storybook/addon-actions",
  "main": "./dist/index.js",
  ...
  "exports": {
    ".": {
      "node": "./dist/index.js",
      "import": "./dist/index.mjs",
      "default": "./dist/index.js"
    },
    "./preview": {
      "import": "./dist/preview.mjs",
      "default": "./dist/preview.js"
    },
    ...
    "./package.json": "./package.json"
  }
}
```

以下是應用程式代碼透過導入 `@storybook/addon-actions` 的不同子路徑來使用上述套件的範例。

```js
import {action} from '@storybook/addon-actions';
// -> '@storybook/addon-actions/dist/index.js'

import {action} from '@storybook/addon-actions/preview';
// -> '@storybook/addon-actions/dist/preview.js'

import helpers from '@storybook/addon-actions/src/preset/addArgsHelpers';
// Inaccessible - not listed in "exports"!
```

套件匯出的主要功能包括：

- **套件封裝**：只有 `"exports"` 中定義的子路徑可從套件外部導入 — 讓套件能控制其公開 API。
- **子路徑別名**：套件可定義映射到不同檔案位置的自定義子路徑（包括透過[子路徑模式](https://nodejs.org/docs/latest-v19.x/api/packages.html#subpath-patterns)）— 允許重新定位檔案同時保持公開 API。
- **條件式匯出**：一個子路徑可能根據環境解析到不同的底層檔案。例如針對 `"node"`、`"browser"` 或 `"react-native"` 運行環境 — 取代 [`"browser"` 欄位規範](https://github.com/defunctzombie/package-browser-field-spec)。

:::note
The full capabilities for `"exports"` are detailed in the [Node.js Package Entry Points spec](https://nodejs.org/docs/latest-v19.x/api/packages.html#package-entry-points).

Since these features overlap with existing React Native concepts (such as [platform-specific extensions](/docs/platform-specific-code)), and since `"exports"` had been live in the npm ecosystem for some time, we reached out to the React Native community to make sure our implementation would meet developers' needs ([PR](https://github.com/react-native-community/discussions-and-proposals/pull/534), [final RFC](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0534-metro-package-exports-support.md)).
:::

## 針對應用程式開發者

Package Exports 功能現已開放測試版啟用。

- 依賴 Package Exports 功能的套件（如 [**Firebase**](https://www.npmjs.com/package/firebase) 和 [**Storybook**](https://www.npmjs.com/search?q=%40storybook)）的導入現在應能按設計運作。
- 使用 Metro 的 React Native for Web 專案現在可利用 `"browser"` 條件導出，無需再使用變通方案。

啟用 Package Exports 會帶來一些[邊緣案例的破壞性變更](#breaking-changes)，可能影響特定專案，您可立即[進行測試](#validating-changes-in-your-project)。

**在未來的 React Native 版本中，Package Exports 將預設啟用**。由於雞與蛋問題，部分套件先前因 React Native 應用程式而延遲遷移至 `"exports"`，或使用我們的 `"react-native"` 根欄位應急方案。Metro 支援這些功能將推動生態系統向前發展。

### 啟用 Package Exports（測試版）

Package Exports can be enabled in your app's [**metro.config.js**](https://github.com/facebook/react-native/blob/0.72-stable/packages/react-native/template/metro.config.js) file via the [`resolver.unstable_enablePackageExports`](https://metrobundler.dev/docs/configuration/#unstable_enablepackageexports-experimental) option.

```js
const config = {
  // ...
  resolver: {
    unstable_enablePackageExports: true,
  },
};
```

Metro 提供兩個額外的解析器選項來配置條件導出行為：

- [`unstable_conditionNames`](https://metrobundler.dev/docs/configuration/#unstable_conditionnames-experimental) — The set of [condition names](https://nodejs.org/docs/latest-v19.x/api/packages.html#community-conditions-definitions) to assert when resolving conditional exports. By default, we match `['require', 'import', 'react-native']`.
- [`unstable_conditionsByPlatform`](https://metrobundler.dev/docs/configuration/#unstable_conditionsbyplatform-experimental) — The additional condition names to assert when resolving for a given platform target. By default, this matches `'browser'` when the platform is `'web'`.

:::tip
**Remember to use the React Native [Jest preset](https://github.com/facebook/react-native/blob/main/template/jest.config.js#L2)!** Jest includes support for Package Exports by default. In tests, you can override which `customExportConditions` are resolved using the [`testEnvironmentOptions`](https://jestjs.io/docs/configuration#testenvironmentoptions-object) option.

**If you are using TypeScript**, resolution behaviour can be matched by setting [`moduleResolution: 'bundler'`](https://www.typescriptlang.org/tsconfig#moduleResolution) and [`resolvePackageJsonImports: false`](https://www.typescriptlang.org/tsconfig#resolvePackageJsonExports) within your project's `tsconfig.json`.
:::

#### 驗證專案中的變更

對於現有專案，建議早期採用者按照以下步驟確認啟用 `unstable_enablePackageExports` 後是否發生解析變更。此為一次性流程。很可能完全沒有變更，但我們希望開發者能明確選擇啟用。

<details>
<summary>💡 Validating changes in your project</summary>

:::note
If you are not using Yarn, substitute `yarn` for `npx` (or the relevant tool used in your project).
:::

1. Get all resolved dependencies (before changes):

   ```sh
   # Replace index.js with your entry file if needed, such as App.js
   yarn metro get-dependencies index.js --platform android --output before.txt
   ```

   - **Expo CLI**: Run `npx expo customize metro.config.js` if your project doesn't have a `metro.config.js` file yet.
   - For full coverage, substitute `--platform android` for the other platforms in use by your app (e.g. `ios`, `web`).

2. Enable `resolver.unstable_enablePackageExports` in `metro.config.js`.
3. Get all resolved dependencies (after changes):

   ```sh
   yarn metro get-dependencies index.js --platform android --output after.txt
   ```

4. Compare!

   ```sh
   diff before.txt after.txt
   ```

</details>

### 破壞性變更

我們決定在 Metro 中實作符合規範的 Package Exports（這需要一些破壞性變更），但同時保持向後相容性（協助現有導入的應用程式逐步遷移）。

關鍵的破壞性變更是：當套件提供 `"exports"` 時，將優先參考該欄位（而非其他 `package.json` 欄位）——且會直接使用匹配的子路徑目標。

- Metro 不會針對導入說明符展開 [`sourceExts`](https://metrobundler.dev/docs/configuration/#sourceexts)。
- Metro 不會針對目標檔案解析[平台特定副檔名](/docs/platform-specific-code)。

詳情請參閱 Metro 文件中的所有[**破壞性變更**](https://metrobundler.dev/docs/package-exports#summary-of-breaking-changes)。

### 套件封裝採用寬鬆模式

當 Metro 遇到未列在 `"exports"` 中的子路徑時，**會退回舊版解析方式**。此相容性功能旨在減少既有 React Native 專案中原本允許的導入操作所產生的摩擦。

Metro 不會拋出錯誤，而是記錄警告訊息。

```sh
warn: You have imported the module "foo/private/fn.js" which is not listed in
the "exports" of "foo". Consider updating your call site or asking the package
maintainer(s) to expose this API.
```

:::note
我們計劃在未來實作嚴格的套件封裝模式，以符合 Node 的預設行為。因此**建議所有開發者處理這些警告**（若使用者收到相關提示）。
:::

## 給套件維護者（預覽）

:::info
根據我們的[推出計劃](#the-future-stable-exports-enabled-by-default)，Package Exports 功能將在下一版 React Native（0.73）中為多數專案預設啟用。

**我們目前沒有計劃在短期內移除對 `"main"` 欄位及其他現有套件解析功能的支援。**
:::

Package Exports 提供了限制套件內部存取的能力，並為函式庫鎖定 React Native 與 React Native for Web 提供更可預測的功能。

### If you are using `"exports"` today

若您的套件同時使用 `"exports"` 與現有的 `"react-native"` 根欄位，請注意上述使用者的[重大變更](#breaking-changes)。對於在 Metro 中啟用此功能的使用者，`"exports"` 現在會優先於模組解析過程中被考量。

實際上，我們預期使用者的主要變動將來自套件封裝中不可存取子路徑的強制警告（透過 `"exports"` 規範）。

### 遷移至 `"exports"`

**為套件添加 `"exports"` 欄位完全是可選的**。現有的套件解析功能對於未使用 `"exports"` 的套件將保持相同行為——且我們沒有移除此行為的計劃。

我們認為 `"exports"` 的新功能為 React Native 套件維護者提供了引人注目的功能集。

- **強化套件 API**：現在是審視套件模組 API 的好時機，可透過匯出的子路徑別名正式定義。這能防止使用者存取內部 API，減少錯誤發生的範圍。
- **條件式匯出**：若您的套件目標是 React Native for Web（即 `"react-native"` 和 `"browser"`），我們現在讓套件能控制這些條件的解析順序（參見下個標題）。

若決定引入 `"exports"`，**建議將其視為重大變更**。我們在 Metro 文件中準備了[**遷移指南**](https://metrobundler.dev/docs/package-exports#migration-guide-for-package-maintainers)，內容包含如何取代平台特定擴充等功能。

:::note
**請勿依賴 Metro 實作中的寬鬆行為**。雖然 Metro 保持向後相容性，但套件應遵循規範文件中的 `"exports"` 定義及其他工具的嚴格實作方式。
:::

### 新的 `"react-native"` 條件

我們已將 `"react-native"` 納入社群條件（用於條件式匯出）。這代表 React Native 框架與其他已知執行環境（如 `"node"` 和 `"deno"`）並列（參見 [RFC](https://github.com/nodejs/node/pull/45367)）。

> [社群條件定義——**`"react-native"`**](https://nodejs.org/docs/latest-v19.x/api/packages.html#community-conditions-definitions)
>
> _將由 React Native 框架匹配（所有平台）。若目標為 React Native for Web，應在此條件前指定 "browser"。_

這取代了先前的 `"react-native"` 根欄位。過去解析優先順序由專案決定的方式，[在使用 React Native for Web 時會產生歧義](https://github.com/expo/router/issues/37#issuecomment-1275925758)。在 `"exports"` 規範下，_套件能明確定義條件進入點的解析順序_——消除了此歧義。

```json
  "exports": {
    "browser": "./dist/index-browser.js",
    "react-native": "./dist/index-react-native.js",
    "default": "./dist/index.js"
  }
```

:::note
我們決定不引入 `"android"` 和 `"ios"` 條件，原因在於現有平台選擇方法已廣泛使用，且此行為在不同框架間的運作方式過於複雜。請改用 [`Platform.select()`](/docs/platform#select) API。
:::

## The future: Stable `"exports"`, enabled by default

在下一個 React Native 版本中，我們計劃移除此功能的 `unstable_` 前綴（在完成預定的效能優化與錯誤修正後），並將預設啟用 Package Exports 解析功能。

當 `"exports"` 全面啟用後，我們能帶領 React Native 社群向前邁進——例如，React Native 的核心套件可進行更新，以更明確區分公開模組與內部模組。

![Package Exports 支援的推出計劃](/blog/assets/package-exports-rollout.png)

## 致謝

感謝 React Native 社群成員對 RFC 提供的反饋：[@SimenB](https://github.com/SimenB)、[@tido64](https://github.com/tido64)、[@byCedric](https://github.com/byCedric)、[@thymikee](https://github.com/thymikee)。

特別感謝 Meta 的 [@motiz88](https://github.com/motiz88) 和 [@robhogan](https://github.com/robhogan) 對此功能開發的支持。