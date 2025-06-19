---
id: upgrading
title: Upgrading to new versions
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

升級至新版本的 React Native 將讓您獲得更多 API、視圖、開發者工具和其他好處。升級需要一些努力，但我們會盡量讓過程簡單明瞭。

## Expo 專案

Upgrading your Expo project to a new version of React Native requires updating the `react-native`, `react`, and `expo` package versions in your `package.json` file. Expo recommends upgrading SDK versions incrementally, one at a time. Doing so will help you pinpoint breakages and issues that arise during the upgrade process. See the [Upgrading Expo SDK Walkthrough](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/) for up-to-date information about upgrading your project.

## React Native 專案

由於典型的 React Native 專案基本上由 Android 專案、iOS 專案和 JavaScript 專案組成，升級可能會相當棘手。[Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) 是一個網頁工具，可以幫助您在升級應用程式時提供兩個版本之間的所有變更。它還會顯示特定文件的註解，以幫助理解為什麼需要這些變更。

### 1. 選擇版本

首先，您需要選擇要從哪個版本升級到哪個版本，預設會選擇最新的主要版本。選擇後，您可以點擊「Show me how to upgrade」按鈕。

💡 主要更新會在頂部顯示一個「useful content」部分，其中包含幫助您升級的連結。

### 2. 升級依賴項

第一個顯示的文件是 `package.json`，最好更新其中顯示的依賴項。例如，如果 `react-native` 和 `react` 顯示為變更，則可以通過運行以下命令在您的專案中安裝它們：

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

#### 我已完成所有變更，但我的應用程式仍在使用舊版本

這類錯誤通常與快取有關，建議安裝 [react-native-clean-project](https://github.com/pmadruga/react-native-clean-project) 來清除所有專案快取，然後再次運行。