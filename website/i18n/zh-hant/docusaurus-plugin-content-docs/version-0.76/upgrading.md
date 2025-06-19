---
id: upgrading
title: Upgrading to new versions
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

升級至新版本的 React Native 將讓您獲得更多 API、視圖、開發者工具和其他優化功能。升級過程需要少量努力，但我們會盡量使其對您直觀易操作。

## Expo 專案

Upgrading your Expo project to a new version of React Native requires updating the `react-native`, `react`, and `expo` package versions in your `package.json` file. Expo recommends upgrading SDK versions incrementally, one at a time. Doing so will help you pinpoint breakages and issues that arise during the upgrade process. See the [Upgrading Expo SDK Walkthrough](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/) for up-to-date information about upgrading your project.

## React Native 專案

由於典型的 React Native 專案本質上由 Android 專案、iOS 專案和 JavaScript 專案組成，升級過程可能相當複雜。[Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) 是一個網頁工具，可協助您升級應用程式，提供任意兩個版本之間完整的變更集。它還會顯示特定文件的註解，幫助理解為何需要該變更。

### 1. 選擇版本

首先需選擇要從哪個版本升級至哪個版本，預設會選取最新的主要版本。選擇完成後，可點擊「Show me how to upgrade」按鈕。

💡 主要版本更新會在頂部顯示「useful content」區段，其中包含升級時可能有用的連結。

### 2. 升級依賴項

顯示的第一個文件是 `package.json`，建議更新其中顯示的依賴項。例如，若 `react-native` 和 `react` 顯示為變更項目，可透過執行以下命令在專案中安裝：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
# {{VERSION}} and {{REACT_VERSION}} are the release versions showing in the diff
npm install react-native@{{VERSION}}
npm install react@{{REACT_VERSION}}
```

</TabItem>
<TabItem value="yarn">

```shell
# {{VERSION}} and {{REACT_VERSION}} are the release versions showing in the diff
yarn add react-native@{{VERSION}}
yarn add react@{{REACT_VERSION}}
```

</TabItem>
</Tabs>

### 3. 升級專案文件

The new release may contain updates to other files that are generated when you run `npx react-native init`, those files are listed after the `package.json` in the [Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) page. If there aren't other changes then you only need to rebuild the project to continue developing. In case there are changes you need to manually apply them into your project.

### 疑難排解

#### 我已完成所有變更，但應用程式仍使用舊版本

這類錯誤通常與快取有關，建議安裝 [react-native-clean-project](https://github.com/pmadruga/react-native-clean-project) 來清除專案的所有快取，然後再次執行。