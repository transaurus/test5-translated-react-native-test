---
id: react-devtools
title: React Developer Tools
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

您可以使用 [React Developer Tools 的獨立版本](https://github.com/facebook/react/tree/main/packages/react-devtools) 來除錯 React 元件層級結構。要使用它，請全域安裝 `react-devtools` 套件：

<Tabs groupId="package-manager" defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm install -g react-devtools
```

</TabItem>
<TabItem value="yarn">

```shell
yarn global add react-devtools
```

</TabItem>
</Tabs>

現在從終端機執行 `react-devtools` 以啟動獨立的 DevTools 應用程式。它應該會在幾秒內連接到您的模擬器。

```shell
react-devtools
```

![React DevTools](/docs/assets/ReactDevTools.png)

:::info
If you prefer to avoid global installations, you can add `react-devtools` as a project dependency. Add the `react-devtools` package to your project using `npm install --save-dev react-devtools`, then add `"react-devtools": "react-devtools"` to the `scripts` section in your `package.json`, and then run `npm run react-devtools` from your project folder to open the DevTools.
:::

## 與 React Native Inspector 整合

開啟開發者選單並選擇「切換 Inspector」。它會顯示一個疊加層，讓您可以點擊任何 UI 元素並查看相關資訊：

![React Native Inspector](/docs/assets/Inspector.gif)

然而，當 `react-devtools` 正在執行時，Inspector 會進入折疊模式，並改以 DevTools 作為主要 UI。在此模式下，點擊模擬器中的某個元素會在 DevTools 中顯示相關元件：

![React DevTools Inspector 整合](/docs/assets/ReactDevToolsInspector.gif)

您可以在同一選單中選擇「切換 Inspector」以退出此模式。

## 除錯應用程式狀態

[Reactotron](https://github.com/infinitered/reactotron) 是一個開源桌面應用程式，允許您檢查 Redux 或 MobX-State-Tree 的應用程式狀態，以及查看自訂日誌、執行自訂命令（例如重設狀態）、儲存和還原狀態快照，以及其他對 React Native 應用程式有幫助的除錯功能。

您可以查看 [README](https://github.com/infinitered/reactotron) 中的安裝說明。如果您使用 Expo，這裡有一篇文章詳細說明 [如何在 Expo 上安裝](https://shift.infinite.red/start-using-reactotron-in-your-expo-project-today-in-3-easy-steps-a03d11032a7a)。

## 疑難排解

:::tip
一旦您啟動了 React DevTools，請按照指示操作。如果您在開啟 React DevTools 之前已經執行了應用程式，您可能需要 [開啟開發者選單](/docs/debugging#accessing-the-dev-menu) 來連接它們。

![React DevTools 連接](/docs/assets/ReactDevToolsConnection.gif)
:::

:::info
如果連接到模擬器時遇到問題（尤其是 Android 12），請嘗試在新的終端機中執行 `adb reverse tcp:8097 tcp:8097`。
:::