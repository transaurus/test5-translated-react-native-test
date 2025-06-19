---
id: metro
title: Metro
---

React Native 使用 [Metro](https://metrobundler.dev/) 來建置您的 JavaScript 程式碼與資源。

## 設定 Metro

Metro 的設定選項可透過專案中的 `metro.config.js` 檔案進行自訂。該檔案可匯出以下兩種形式：

- **An object (recommended)** that will be merged on top of Metro's internal config defaults.
- [**A function**](#advanced-using-a-config-function) that will be called with Metro's internal config defaults and should return a final config object.

:::tip
請參閱 Metro 官方文件的 [**設定 Metro**](https://metrobundler.dev/docs/configuration) 章節，以獲取所有可用設定選項的完整說明。
:::

在 React Native 中，您的 Metro 設定應繼承自 [`@react-native/metro-config`](https://www.npmjs.com/package/@react-native/metro-config) 或 [`@expo/metro-config`](https://www.npmjs.com/package/@expo/metro-config)。這些套件包含了建置與執行 React Native 應用程式所需的必要預設值。

以下是 React Native 範本專案中的預設 `metro.config.js` 檔案內容：

<!-- prettier-ignore -->

```js
const {getDefaultConfig, mergeConfig} = require('@react-native/metro-config');

/**
 * Metro configuration
 * https://metrobundler.dev/docs/configuration
 *
 * @type {import('metro-config').MetroConfig}
 */
const config = {};

module.exports = mergeConfig(getDefaultConfig(__dirname), config);
```

您可以在 `config` 物件中自訂所需的 Metro 選項。

### 進階：使用設定函式

匯出設定函式表示您選擇自行管理最終設定——**Metro 將不會套用任何內部預設值**。當需要從 Metro 讀取基礎預設設定物件，或需動態設定選項時，此模式會相當有用。

:::info
**自 `@react-native/metro-config` 0.72.1 版本起**，您不再需要使用設定函式來取得完整的預設設定。請參閱下方的**提示**段落。
:::

<!-- prettier-ignore -->

```js
const {getDefaultConfig, mergeConfig} = require('@react-native/metro-config');

module.exports = function (baseConfig) {
  const defaultConfig = mergeConfig(baseConfig, getDefaultConfig(__dirname));
  const {resolver: {assetExts, sourceExts}} = defaultConfig;

  return mergeConfig(
    defaultConfig,
    {
      resolver: {
        assetExts: assetExts.filter(ext => ext !== 'svg'),
        sourceExts: [...sourceExts, 'svg'],
      },
    },
  );
};
```

:::tip
使用設定函式屬於進階使用情境。相較於上述方法，更簡單的方式（例如自訂 `sourceExts`）是從 `@react-native/metro-config` 讀取這些預設值。

**替代方案**

<!-- prettier-ignore -->
```js
const defaultConfig = getDefaultConfig(__dirname);

const config = {
  resolver: {
    sourceExts: [...defaultConfig.resolver.sourceExts, 'svg'],
  },
};

module.exports = mergeConfig(defaultConfig, config);
```

**但請注意！** 我們建議在覆寫這些設定值時，採用複製並編輯的方式——將設定來源明確保留在您的設定檔案中。

✅ **建議做法**

<!-- prettier-ignore -->
```js
const config = {
  resolver: {
    sourceExts: ['js', 'ts', 'tsx', 'svg'],
  },
};
```

:::

## 深入了解 Metro

- [Metro 官方網站](https://metrobundler.dev/)
- [影片：App.js 2023 研討會「Metro & React Native 開發者體驗」演講](https://www.youtube.com/watch?v=c9D4pg0y9cI)