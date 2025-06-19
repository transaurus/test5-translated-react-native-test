---
id: upgrading
title: Upgrading to new versions
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

升級至新版本的 React Native 將讓您獲得更多 API、視圖、開發者工具和其他好處。升級需要一些努力，但我們盡量使其過程直觀簡單。

## Expo 專案

Upgrading your Expo project to a new version of React Native requires updating the `react-native`, `react`, and `expo` package versions in your `package.json` file. Expo provides an `upgrade` command to handle upgrading these and any other known dependencies for you. See the [Upgrading Expo SDK Walkthrough](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/) for up-to-date information about upgrading your project.

## React Native 專案

由於典型的 React Native 專案基本上由一個 Android 專案、一個 iOS 專案和一個 JavaScript 專案組成，升級可能會相當棘手。[Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) 是一個網頁工具，可以幫助您在升級應用程式時提供兩個版本之間的所有變更。它還顯示特定檔案的註解，以幫助理解為什麼需要該變更。

### 1. 選擇版本

首先，您需要選擇要從哪個版本升級到哪個版本，預設情況下會選擇最新的主要版本。選擇後，您可以點擊「Show me how to upgrade」按鈕。

💡 主要更新會在頂部顯示一個「useful content」區塊，其中包含升級時可能有幫助的連結。

:::tip
或者，您可以執行 `npx react-native upgrade`，它會自動檢查您當前的版本和最新的可用版本，並顯示已選擇版本的 Upgrade Helper 頁面連結。
:::

### 2. 升級相依套件

第一個顯示的檔案是 `package.json`，建議更新其中顯示的相依套件。例如，如果 `react-native` 和 `react` 顯示為變更，則可以通過執行以下命令在您的專案中安裝它們：

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

新版本可能包含對執行 `npx react-native init` 時生成的其他檔案的更新，這些檔案列在 Upgrade Helper 頁面的 `package.json` 之後。如果沒有其他變更，則只需重新建置專案即可繼續開發。

如果有變更，您可以手動更新它們，從頁面中的變更複製並貼上，或者通過執行 React Native CLI 升級指令來完成：

```shell
npx react-native upgrade
```

這將根據最新模板檢查您的檔案並執行以下操作：

- 如果模板中有新檔案，則會創建它。
- 如果模板中的檔案與您的檔案相同，則會跳過。
- 如果您的專案中的檔案與模板不同，則會提示您；您可以選擇保留您的檔案或使用模板版本覆蓋它。

> 某些升級無法通過 React Native CLI 自動完成，需要手動操作，例如從 `0.28` 升級到 `0.29`，或從 `0.56` 升級到 `0.57`。升級時請務必檢查 [release notes](https://github.com/facebook/react-native/releases)，以識別您的專案可能需要的手動變更。

### 疑難排解

#### 我已完成所有變更，但我的應用程式仍在使用舊版本

這類錯誤通常與快取有關，建議安裝 [react-native-clean-project](https://github.com/pmadruga/react-native-clean-project) 來清除專案的所有快取，然後再次執行。