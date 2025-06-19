---
id: debugging
title: Debugging
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 存取應用內開發者選單

您可以透過搖動裝置或在 iOS 模擬器的「硬體」選單中選擇「搖動手勢」來存取開發者選單。當應用在 iOS 模擬器中運行時，您也可以使用鍵盤快捷鍵 `⌘D`；在 macOS 的 Android 模擬器上使用 `⌘M`，或在 Windows 和 Linux 上使用 `Ctrl+M`。另外，對於 Android 裝置，您可以執行指令 `adb shell input keyevent 82` 來開啟開發者選單（82 是選單鍵的代碼）。

![](/docs/assets/DeveloperMenu.png)

> 開發者選單在正式（生產）版本中會被停用。

## 啟用快速重新整理

快速重新整理是 React Native 的一項功能，讓您能夠在修改 React 元件時獲得近乎即時的反饋。在除錯時，啟用[快速重新整理](fast-refresh.md)會有所幫助。快速重新整理預設是啟用的，您可以在 React Native 開發者選單中切換「啟用快速重新整理」。啟用後，大多數的編輯應該會在一兩秒內顯示出來。

## 啟用鍵盤快捷鍵

React Native 在 iOS 模擬器中支援一些鍵盤快捷鍵。以下會描述這些快捷鍵。在 macOS 上，要啟用這些快捷鍵，請在模擬器應用中開啟 I/O 選單，選擇「鍵盤」，並確保「連接硬體鍵盤」已勾選。

## LogBox

開發版本中的錯誤和警告會顯示在應用內的 LogBox 中。

> LogBox 在正式（生產）版本中會自動停用。

### 主控台錯誤與警告

主控台錯誤和警告會以帶有紅色或黃色標記的螢幕通知顯示，並分別標示主控台中錯誤或警告的數量。要查看主控台錯誤或警告，請點擊通知以查看完整的日誌資訊，並可翻頁瀏覽主控台中的所有日誌。

這些通知可以透過 `LogBox.ignoreAllLogs()` 來隱藏。這在進行產品演示等場合非常有用。此外，也可以透過 `LogBox.ignoreLogs()` 針對特定日誌隱藏通知。這在第三方依賴庫中有無法修復的嘈雜警告時特別實用。

> 忽略日誌應作為最後手段，並應建立任務來修復任何被忽略的日誌。

```tsx
import {LogBox} from 'react-native';

// Ignore log notification by message:
LogBox.ignoreLogs(['Warning: ...']);

// Ignore all log notifications:
LogBox.ignoreAllLogs();
```

### 未處理的錯誤

未處理的 JavaScript 錯誤（例如 `undefined is not a function`）會自動開啟全螢幕的 LogBox 錯誤，並顯示錯誤來源。這些錯誤可以關閉或最小化，以便您查看錯誤發生時應用的狀態，但應始終予以解決。

### Syntax Errors

When syntax error occurs the full screen LogBox error will automatically open with the stack trace and location of the syntax error. This error is not dismissable because it represents invalid JavaScript execution that must be fixed before continuing with your app. To dismiss these errors, fix the syntax error and either save to automatically dismiss (with Fast Refresh enabled) or cmd+r to reload (with Fast Refresh disabled).

## Chrome 開發者工具

要在 Chrome 中除錯 JavaScript 程式碼，請從開發者選單中選擇「遠端除錯 JS」。這將在 [http://localhost:8081/debugger-ui](http://localhost:8081/debugger-ui) 開啟一個新分頁。

Select `Tools → Developer Tools` from the Chrome Menu to open the [Developer Tools](https://developer.chrome.com/devtools). You may also access the DevTools using keyboard shortcuts (`⌘⌥I` on macOS, `Ctrl` `Shift` `I` on Windows). You may also want to enable [Pause On Caught Exceptions](http://stackoverflow.com/questions/2233339/javascript-is-there-a-way-to-get-chrome-to-break-on-all-errors/17324511#17324511) for a better debugging experience.

> Note: on Android, if the times between the debugger and device have drifted; things such as animation, event behavior, etc., might not work properly or the results may not be accurate. Please correct this by running ``adb shell "date `date +%m%d%H%M%Y.%S%3N`"`` on your debugger machine. Root access is required for the use in real device.

> 注意：React Developer Tools Chrome 擴充功能無法與 React Native 相容，但可使用其獨立版本。詳見[此章節](debugging.md#react-developer-tools)說明。

### 使用自訂 JavaScript 偵錯工具

若要改用自訂 JavaScript 偵錯工具替代 Chrome 開發者工具，需將 `REACT_DEBUGGER` 環境變數設為啟動偵錯工具的命令。接著從開發者選單選擇「Debug JS Remotely」即可開始偵錯。

偵錯工具會接收到以空格分隔的專案根目錄列表。例如，若設定 `REACT_DEBUGGER="node /path/to/launchDebugger.js --port 2345 --type ReactNative"`，則會使用指令 `node /path/to/launchDebugger.js --port 2345 --type ReactNative /path/to/reactNative/app` 啟動偵錯工具。

> 此方式執行的自訂偵錯命令應為短暫執行的程序，且輸出不得超過 200 KB。

## Safari 開發者工具

無需啟用「Debug JS Remotely」，即可透過 Safari 偵錯 iOS 版應用程式。

- On a physical device go to: `Settings → Safari → Advanced → Make sure "Web Inspector" is turned on` (This step is not needed on the Simulator)
- On your Mac enable Develop menu in Safari: `Preferences → Advanced → Select "Show Develop menu in menu bar"`
- Select your app's JSContext: `Develop → Simulator (or other device) → JSContext`
- Safari's Web Inspector should open which has a Console and a Debugger

預設可能未啟用原始碼對應，可參照[此指南](http://blog.nparashuram.com/2019/10/debugging-react-native-ios-apps-with.html)或[影片](https://www.youtube.com/watch?v=GrGqIIz51k4)啟用，並在原始碼正確位置設定中斷點。

需注意每次應用程式重新載入（透過即時重新載入或手動操作）時，都會建立新的 JSContext。勾選「自動顯示 JSContext 的網頁檢查器」可避免手動選取最新 JSContext。

## React 開發者工具

可使用 [React Developer Tools 獨立版本](https://github.com/facebook/react/tree/main/packages/react-devtools)偵錯 React 元件層級結構。需先全域安裝 `react-devtools` 套件：

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

應於數秒內自動連接模擬器。

:::info
若連接模擬器時遇到問題（特別是 Android 12），可嘗試在新終端機執行 `adb reverse tcp:8097 tcp:8097`。
:::

:::info
If you prefer to avoid global installations, you can add `react-devtools` as a project dependency. Add the `react-devtools` package to your project using `npm install --save-dev react-devtools`, then add `"react-devtools": "react-devtools"` to the `scripts` section in your `package.json`, and then run `npm run react-devtools` from your project folder to open the DevTools.
:::

### 與 React Native 檢查器整合

開啟應用內的開發者選單並選擇「切換檢查器」。這將顯示一個疊加層，讓您可以點擊任何 UI 元素並查看相關資訊：

![React Native 檢查器](/docs/assets/Inspector.gif)

然而，當 `react-devtools` 運行時，檢查器會進入折疊模式，並改以 DevTools 作為主要 UI。在此模式下，點擊模擬器中的內容將在 DevTools 中顯示相關元件：

![React DevTools 檢查器整合](/docs/assets/ReactDevToolsInspector.gif)

您可以在同一選單中選擇「切換檢查器」以退出此模式。

### 檢查元件實例

在 Chrome 中調試 JavaScript 時，您可以在瀏覽器控制台中檢查 React 元件的 props 和 state。

首先，按照在 Chrome 中調試的指示開啟 Chrome 控制台。

確保 Chrome 控制台左上角的下拉選單顯示 `debuggerWorker.js`。**此步驟至關重要。**

然後在 React DevTools 中選擇一個 React 元件。頂部有一個搜尋框可幫助您按名稱查找元件。一旦選擇它，它將在 Chrome 控制台中作為 `$r` 可用，讓您可以檢查其 props、state 和實例屬性。

![React DevTools Chrome 控制台整合](/docs/assets/ReactDevToolsDollarR.gif)

## 效能監控器

您可以通過在開發者選單中選擇「效能監控器」來啟用效能疊加層，以幫助您調試效能問題。

<hr style={{marginTop: 25, marginBottom: 25}} />

## 調試應用程式狀態

[Reactotron](https://github.com/infinitered/reactotron) 是一個開源桌面應用程式，允許您檢查 Redux 或 MobX-State-Tree 應用程式狀態，查看自訂日誌，運行自訂命令（如重置狀態），儲存和恢復狀態快照，以及其他有助於調試 React Native 應用的功能。

您可以在 [README](https://github.com/infinitered/reactotron) 中查看安裝說明。如果您使用 Expo，這裡有一篇文章詳細介紹了 [如何在 Expo 上安裝](https://shift.infinite.red/start-using-reactotron-in-your-expo-project-today-in-3-easy-steps-a03d11032a7a)。

# 原生調試

<div className="banner-native-code-required">
  <h3>Projects with Native Code Only</h3><p>
    The following section only applies to projects with native code exposed. If you are using the managed Expo workflow, see the guide on <a href="https://docs.expo.dev/workflow/prebuild/" target="_blank">prebuild</a> to use this API.</p>
</div>

## 存取控制台日誌

您可以在應用運行時，在終端中使用以下命令來顯示 iOS 或 Android 應用的控制台日誌：

```shell
npx react-native log-ios
npx react-native log-android
```

You may also access these through `Debug → Open System Log...` in the iOS Simulator or by running `adb logcat "*:S" ReactNative:V ReactNativeJS:V` in a terminal while an Android app is running on a device or emulator.

> 如果您使用 Create React Native App 或 Expo CLI，控制台日誌已經顯示在與打包器相同的終端輸出中。

## 使用 Chrome 開發者工具在設備上調試

> 如果您使用 Create React Native App 或 Expo CLI，這已經為您配置好了。

在 iOS 設備上，開啟檔案 [`RCTWebSocketExecutor.mm`](https://github.com/facebook/react-native/blob/0.71-stable/React/CoreModules/RCTWebSocketExecutor.mm) 並將 "localhost" 更改為您電腦的 IP 地址，然後在開發者選單中選擇「遠端調試 JS」。

在通過 USB 連接的 Android 5.0+ 設備上，您可以使用 [`adb` 命令行工具](http://developer.android.com/tools/help/adb.html) 來設置從設備到您電腦的端口轉發：

`adb reverse tcp:8081 tcp:8081`

或者，從開發者選單中選擇「開發設定」，然後將「裝置的除錯伺服器主機」設定更新為與您電腦的 IP 位址相符。

> 如果遇到任何問題，可能是某個 Chrome 擴充功能與除錯工具產生意外的交互作用。嘗試停用所有擴充功能，然後逐一重新啟用，直到找出有問題的擴充功能。

## 除錯原生程式碼

當處理原生程式碼時，例如編寫原生模組，您可以從 Android Studio 或 Xcode 啟動應用程式，並利用原生除錯功能（設置斷點等），就像在開發標準原生應用程式時一樣。

另一個選項是使用 React Native CLI 運行您的應用程式，並將原生 IDE（Android Studio 或 Xcode）的原生除錯工具附加到該程序。

### Android Studio

在 Android Studio 中，您可以通過點擊選單欄上的「Run」選項，選擇「Attach to Process...」，然後選擇正在運行的 React Native 應用程式來實現這一點。

### Xcode

在 Xcode 中，點擊頂部選單欄上的「Debug」，選擇「Attach to process」選項，然後在「Likely Targets」列表中選擇應用程式。