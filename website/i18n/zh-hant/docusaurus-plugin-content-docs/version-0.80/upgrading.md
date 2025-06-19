---
id: upgrading
title: Upgrading to new versions
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

升級至新版本的 React Native 將讓您獲得更多 API、視圖元件、開發者工具和其他優化功能。升級過程需要少量調整，但我們會盡可能讓流程保持直觀。

## Expo 專案

Upgrading your Expo project to a new version of React Native requires updating the `react-native`, `react`, and `expo` package versions in your `package.json` file. Expo recommends upgrading SDK versions incrementally, one at a time. Doing so will help you pinpoint breakages and issues that arise during the upgrade process. See the [Upgrading Expo SDK Walkthrough](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/) for up-to-date information about upgrading your project.

## React Native 專案

由於典型的 React Native 專案本質上由 Android 專案、iOS 專案和 JavaScript 專案組成，升級過程較為複雜。[升級助手](https://react-native-community.github.io/upgrade-helper/) 是一款網頁工具，可透過完整顯示兩個版本間的所有變更來協助升級，並提供檔案變更註解以說明修改原因。

### 1. 選擇版本

首先需選擇要從哪個版本升級至哪個版本（預設為最新主要版本），選擇後點擊「顯示升級方式」按鈕。

💡 主要版本更新時，頂部會顯示「實用內容」區塊，包含升級相關輔助連結。

### 2. 升級依賴套件

首先顯示的是 `package.json` 檔案，建議更新其中標示的依賴套件。例如若顯示需變更 `react-native` 和 `react`，可執行以下指令安裝：

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

### 3. 升級專案檔案

The new release may contain updates to other files that are generated when you run `npx react-native init`, those files are listed after the `package.json` in the [Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) page. If there aren't other changes then you only need to rebuild the project to continue developing. In case there are changes you need to manually apply them into your project.

### 疑難排解

#### 已完成所有變更，但應用程式仍使用舊版本

此類錯誤通常與快取有關，建議安裝 [react-native-clean-project](https://github.com/pmadruga/react-native-clean-project) 清除專案所有快取後重新執行。