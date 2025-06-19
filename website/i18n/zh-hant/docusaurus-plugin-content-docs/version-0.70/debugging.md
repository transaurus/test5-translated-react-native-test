---
id: debugging
title: Debugging
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 存取應用內開發者選單

您可以透過搖動裝置或在 iOS 模擬器的硬體選單中選擇「搖動手勢」來存取開發者選單。當應用在 iOS 模擬器中運行時，您也可以使用鍵盤快捷鍵 `⌘D`；在 macOS 的 Android 模擬器中使用 `⌘M`，或在 Windows 和 Linux 中使用 `Ctrl+M`。另外，對於 Android 裝置，您可以執行指令 `adb shell input keyevent 82` 來開啟開發者選單（82 是選單鍵的代碼）。

![](/docs/assets/DeveloperMenu.png)

> 開發者選單在正式（生產）版本中會被停用。

## 啟用快速重新整理

快速重新整理是 React Native 的一項功能，讓您能幾乎即時獲得對 React 元件變更的反饋。在除錯時，啟用[快速重新整理](fast-refresh.md)會有所幫助。快速重新整理預設為啟用狀態，您可以在 React Native 開發者選單中切換「啟用快速重新整理」。啟用後，大多數編輯應該能在一兩秒內看到效果。

## 啟用鍵盤快捷鍵

React Native 在 iOS 模擬器中支援一些鍵盤快捷鍵，如下所述。要在 macOS 上啟用這些快捷鍵，請在模擬器應用程式中打開 I/O 選單，選擇鍵盤，並確保「連接硬體鍵盤」已勾選。

## LogBox

開發版本中的錯誤和警告會顯示在應用內的 LogBox 中。

> LogBox 在正式（生產）版本中會自動停用。

### 主控台錯誤與警告

主控台錯誤和警告會以帶有紅色或黃色標記的螢幕通知形式顯示，分別代表主控台中的錯誤或警告數量。要查看主控台錯誤或警告，請點擊通知以查看該日誌的完整螢幕資訊，並可翻頁瀏覽主控台中的所有日誌。

這些通知可以透過 `LogBox.ignoreAllLogs()` 隱藏。這在進行產品演示等場合非常有用。此外，也可以透過 `LogBox.ignoreLogs()` 逐個隱藏通知。這對於無法修復的第三方依賴項中的嘈雜警告特別有用。

> 僅在最後手段時忽略日誌，並建立任務來修復任何被忽略的日誌。

```jsx
import {LogBox} from 'react-native';

// Ignore log notification by message:
LogBox.ignoreLogs(['Warning: ...']);

// Ignore all log notifications:
LogBox.ignoreAllLogs();
```

### 未處理的錯誤

未處理的 JavaScript 錯誤（例如 `undefined is not a function`）會自動開啟全螢幕的 LogBox 錯誤，並顯示錯誤來源。這些錯誤可以關閉或最小化，以便您能查看錯誤發生時的應用狀態，但應始終予以解決。

### 語法錯誤

當發生語法錯誤時，全螢幕的 LogBox 錯誤會自動開啟，並顯示堆疊追蹤和語法錯誤的位置。此錯誤無法關閉，因為它代表無效的 JavaScript 執行，必須在繼續應用前修復。要關閉這些錯誤，請修復語法錯誤並儲存以自動關閉（啟用快速重新整理時）或按 cmd+r 重新載入（停用快速重新整理時）。

## Chrome 開發者工具

要在 Chrome 中除錯 JavaScript 程式碼，請從開發者選單中選擇「遠端除錯 JS」。這將在 [http://localhost:8081/debugger-ui](http://localhost:8081/debugger-ui) 開啟一個新分頁。

Select `Tools → Developer Tools` from the Chrome Menu to open the [Developer Tools](https://developer.chrome.com/devtools). You may also access the DevTools using keyboard shortcuts (`⌘⌥I` on macOS, `Ctrl` `Shift` `I` on Windows). You may also want to enable [Pause On Caught Exceptions](http://stackoverflow.com/questions/2233339/javascript-is-there-a-way-to-get-chrome-to-break-on-all-errors/17324511#17324511) for a better debugging experience.

> 注意：在 Android 上，若偵錯工具與裝置間的時間不同步，可能會導致動畫效果、事件行為等運作異常或結果不準確。請在偵錯機器上執行 ``adb shell "date `date +%m%d%H%M%Y.%S%3N`"`` 指令校正時間（需 root 權限才能在實體裝置上使用此指令）。

> 注意：React Developer Tools Chrome 擴充功能無法與 React Native 搭配使用，但可改用其獨立版本。詳見[此章節](debugging.md#react-developer-tools)說明。

### 使用自訂 JavaScript 偵錯工具

若要使用自訂 JavaScript 偵錯工具取代 Chrome 開發者工具，請將 `REACT_DEBUGGER` 環境變數設為啟動該偵錯工具的命令。接著從開發者選單選擇「Debug JS Remotely」即可開始偵錯。

偵錯工具會收到以空格分隔的所有專案根目錄列表。例如，若設定 `REACT_DEBUGGER="node /path/to/launchDebugger.js --port 2345 --type ReactNative"`，則會使用指令 `node /path/to/launchDebugger.js --port 2345 --type ReactNative /path/to/reactNative/app` 來啟動偵錯工具。

> 以此方式執行的自訂偵錯命令應為短生命週期程序，且輸出不應超過 200 KB。

## Safari 開發者工具

無需啟用「Debug JS Remotely」，即可使用 Safari 偵錯 iOS 版應用程式。

- On a physical device go to: `Settings → Safari → Advanced → Make sure "Web Inspector" is turned on` (This step is not needed on the Simulator)
- On your Mac enable Develop menu in Safari: `Preferences → Advanced → Select "Show Develop menu in menu bar"`
- Select your app's JSContext: `Develop → Simulator (or other device) → JSContext`
- Safari's Web Inspector should open which has a Console and a Debugger

雖然原始碼對照表可能預設未啟用，但可參照[此指南](http://blog.nparashuram.com/2019/10/debugging-react-native-ios-apps-with.html)或[影片教學](https://www.youtube.com/watch?v=GrGqIIz51k4)啟用功能，並在原始碼正確位置設定中斷點。

需注意每次重新載入應用程式（透過即時重新載入或手動重新載入）時，都會建立新的 JSContext。勾選「自動顯示 JSContext 的網頁檢查器」可避免手動選取最新 JSContext。

## React 開發者工具

可使用 [React Developer Tools 獨立版本](https://github.com/facebook/react/tree/main/packages/react-devtools)來偵錯 React 元件層級結構。請先全域安裝 `react-devtools` 套件：

> 注意：`react-devtools` 第 4 版需搭配 `react-native` 0.62 以上版本才能正常運作。

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
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

在終端機執行 `react-devtools` 以啟動獨立版開發工具應用程式：

```shell
react-devtools
```

![React 開發者工具](/docs/assets/ReactDevTools.png)

應能在數秒內連接到模擬器。

:::info
若連接模擬器時遇到問題（特別是 Android 12），可嘗試在新終端機執行 `adb reverse tcp:8097 tcp:8097`。
:::

:::info
If you prefer to avoid global installations, you can add `react-devtools` as a project dependency. Add the `react-devtools` package to your project using `npm install --save-dev react-devtools`, then add `"react-devtools": "react-devtools"` to the `scripts` section in your `package.json`, and then run `npm run react-devtools` from your project folder to open the DevTools.
:::

### 與 React Native 檢查器整合

開啟應用內的開發者選單並選擇「切換檢查器」。這將顯示一個疊加層，讓你可以點擊任何UI元素並查看相關資訊：

![React Native 檢查器](/docs/assets/Inspector.gif)

然而，當 `react-devtools` 運行時，檢查器會進入折疊模式，並改以開發者工具作為主要介面。在此模式下，點擊模擬器中的元素會在開發者工具中顯示相關元件：

![React 開發者工具檢查器整合](/docs/assets/ReactDevToolsInspector.gif)

你可以從同一選單中選擇「切換檢查器」來退出此模式。

### 檢查元件實例

在 Chrome 中調試 JavaScript 時，你可以在瀏覽器控制台中檢查 React 元件的 props 和 state。

首先，按照 Chrome 調試說明打開 Chrome 控制台。

確保 Chrome 控制台左上角的下拉選單顯示 `debuggerWorker.js`。**此步驟至關重要。**

然後在 React 開發者工具中選擇一個 React 元件。頂部的搜索框可幫助你按名稱查找元件。一旦選中，該元件將在 Chrome 控制台中作為 `$r` 可用，讓你檢查其 props、state 和實例屬性。

![React 開發者工具 Chrome 控制台整合](/docs/assets/ReactDevToolsDollarR.gif)

## 效能監控器

你可以通過在開發者選單中選擇「效能監控器」來啟用效能疊加層，以幫助調試效能問題。

<hr style={{marginTop: 25, marginBottom: 25}} />

## 調試應用狀態

[Reactotron](https://github.com/infinitered/reactotron) 是一個開源桌面應用，允許你檢查 Redux 或 MobX-State-Tree 的應用狀態，查看自定義日誌，運行自定義命令（如重置狀態），存儲和恢復狀態快照，以及其他有助於調試 React Native 應用的功能。

你可以在 [README](https://github.com/infinitered/reactotron) 中查看安裝說明。如果你使用 Expo，這裡有一篇文章詳細介紹了 [如何在 Expo 中安裝](https://shift.infinite.red/start-using-reactotron-in-your-expo-project-today-in-3-easy-steps-a03d11032a7a)。

# 原生調試

<div className="banner-native-code-required">
  <h3>Projects with Native Code Only</h3><p>
    The following section only applies to projects with native code exposed. If you are using the managed Expo workflow, see the guide on <a href="https://docs.expo.dev/workflow/prebuild/" target="_blank">prebuild</a> to use this API.</p>
</div>

## 訪問控制台日誌

你可以在應用運行時，使用以下命令在終端中顯示 iOS 或 Android 應用的控制台日誌：

```shell
npx react-native log-ios
npx react-native log-android
```

你也可以通過 iOS 模擬器中的 `Debug → Open System Log...` 或在 Android 設備或模擬器上運行應用時，在終端中運行 `adb logcat *:S ReactNative:V ReactNativeJS:V` 來訪問這些日誌。

> 如果你使用 Create React Native App 或 Expo CLI，控制台日誌已顯示在與打包程序相同的終端輸出中。

## 使用 Chrome 開發者工具在設備上調試

> 如果你使用 Create React Native App 或 Expo CLI，這已為你配置好。

在 iOS 設備上，打開文件 [`RCTWebSocketExecutor.mm`](https://github.com/facebook/react-native/blob/0.70-stable/React/CoreModules/RCTWebSocketExecutor.mm) 並將 "localhost" 更改為你電腦的 IP 地址，然後從開發者選單中選擇「遠程調試 JS」。

在通過 USB 連接的 Android 5.0+ 設備上，你可以使用 [`adb` 命令行工具](http://developer.android.com/tools/help/adb.html) 設置從設備到電腦的端口轉發：

`adb reverse tcp:8081 tcp:8081`

或者，從開發者選單中選擇「開發設定」，然後將「裝置的除錯伺服器主機」設定更新為與您電腦的 IP 位址相符。

> 如果您遇到任何問題，可能是某個 Chrome 擴充功能與除錯工具產生了非預期的互動。嘗試停用所有擴充功能，然後逐一重新啟用，直到找出有問題的擴充功能為止。

## 除錯原生程式碼

當處理原生程式碼時，例如編寫原生模組時，您可以從 Android Studio 或 Xcode 啟動應用程式，並利用原生除錯功能（設定中斷點等），就像在建置標準原生應用程式時一樣。