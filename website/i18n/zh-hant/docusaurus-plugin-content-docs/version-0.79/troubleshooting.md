---
id: troubleshooting
title: Troubleshooting
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

以下是設定 React Native 時可能遇到的一些常見問題。若您遇到的問題未列於此，請嘗試[在 GitHub 上搜尋相關議題](https://github.com/facebook/react-native/issues/)。

### 連接埠已被佔用

[Metro 打包工具][metro]預設使用 8081 連接埠。若該埠已被其他程序佔用，您可以終止該程序或變更打包工具使用的埠號。

#### 終止佔用 8081 埠的程序

執行以下指令找出佔用 8081 埠的行程 ID：

```shell
sudo lsof -i :8081
```

接著執行以下指令終止該程序：

```shell
kill -9 <PID>
```

在 Windows 系統上，您可以使用[資源監視器](https://stackoverflow.com/questions/48198/how-can-you-find-out-which-process-is-listening-on-a-port-on-windows)找出佔用 8081 埠的程序，並透過工作管理員將其關閉。

#### 使用 8081 以外的埠號

您可以透過 `port` 參數設定打包工具使用其他埠號，於專案根目錄執行：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm start -- --port=8088
```

</TabItem>
<TabItem value="yarn">

```shell
yarn start --port 8088
```

</TabItem>
</Tabs>

You will also need to update your applications to load the JavaScript bundle from the new port. If running on device from Xcode, you can do this by updating occurrences of `8081` to your chosen port in the `ios/__App_Name__.xcodeproj/project.pbxproj` file.

### NPM 鎖定錯誤

若使用 React Native CLI 時遇到類似 `npm WARN locking Error: EACCES` 的錯誤，請嘗試執行：

```shell
sudo chown -R $USER ~/.npm
sudo chown -R $USER /usr/local/lib/node_modules
```

### 缺少 React 相關函式庫

若您手動將 React Native 加入專案，請確認已包含所有使用的相依套件，例如 `RCTText.xcodeproj`、`RCTImage.xcodeproj`。接著需將這些套件編譯的二進位檔連結至您的應用程式。請於 Xcode 專案設定的 `Linked Frameworks and Binaries` 區段進行設定。詳細步驟參見：[連結函式庫](linking-libraries-ios.md#content)。

If you are using CocoaPods, verify that you have added React along with the subspecs to the `Podfile`. For example, if you were using the `<Text />`, `<Image />` and `fetch()` APIs, you would need to add these in your `Podfile`:

```
pod 'React', :path => '../node_modules/react-native', :subspecs => [
  'RCTText',
  'RCTImage',
  'RCTNetwork',
  'RCTWebSocket',
]
```

接著請執行 `pod install` 並確認專案中已生成包含 React 的 `Pods/` 目錄。此後 CocoaPods 會要求您使用生成的 `.xcworkspace` 檔案來載入這些相依套件。

#### 以 CocoaPod 形式使用時編譯失敗

可使用 [cocoapods-fix-react-native](https://github.com/orta/cocoapods-fix-react-native) 這個 CocoaPods 插件來處理因套件管理器差異導致的原始碼修正問題。

#### 參數列表過長：遞迴標頭檔展開失敗

在專案建置設定中，`User Search Header Paths` 和 `Header Search Paths` 用於指定 Xcode 搜尋程式碼中 `#import` 標頭檔的路徑。CocoaPods 會為 Pods 設定預設的搜尋路徑陣列。請確認這些設定未被覆寫，且配置的路徑中沒有過大的目錄。若某個目錄過大，Xcode 嘗試遞迴搜尋時會觸發此錯誤。

若要將 `User Search Header Paths` 和 `Header Search Paths` 的建置設定還原為 CocoaPods 預設值，請在「Build Settings」面板中選取該項目並按下刪除鍵。這將移除自訂覆寫並恢復為 CocoaPod 的預設值。

### 無可用傳輸方式

React Native 實作了 WebSocket 的 polyfill。這些 [polyfill](https://github.com/facebook/react-native/blob/main/packages/react-native/Libraries/Core/InitializeCore.js) 會隨著你透過 `import React from 'react'` 引入的 react-native 模組一起初始化。如果你載入了其他需要 WebSocket 的模組（例如 [Firebase](https://github.com/facebook/react-native/issues/3645)），請確保在 react-native 之後才載入/引入：

```
import React from 'react';
import Firebase from 'firebase';
```

## Shell 指令無回應例外

若遇到類似以下的 ShellCommandUnresponsiveException 例外：

```
Execution failed for task ':app:installDebug'.
  com.android.builder.testing.api.DeviceException: com.android.ddmlib.ShellCommandUnresponsiveException
```

Try [downgrading your Gradle version to 1.2.3](https://github.com/facebook/react-native/issues/2720) in `android/build.gradle`.

## react-native init 卡住

如果在執行 `npx react-native init` 時卡住，請嘗試以詳細模式重新執行，並參考 [#2797](https://github.com/facebook/react-native/issues/2797) 了解常見原因：

```shell
npx react-native init --verbose
```

當你正在除錯程序或需要更多關於錯誤的資訊時，可以使用詳細選項來輸出更多日誌和資訊以鎖定問題。

請在專案的根目錄下執行以下指令：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run android -- --verbose
```

</TabItem>
<TabItem value="yarn">

```shell
yarn android --verbose
```

</TabItem>
</Tabs>

## 無法啟動 react-native 套件管理員（Linux 系統）

### 情況 1：錯誤 "code":"ENOSPC","errno":"ENOSPC"

此問題是由於 [inotify](https://github.com/guard/listen/blob/master/README.md#increasing-the-amount-of-inotify-watchers)（Linux 上由 watchman 使用）可監控的目錄數量不足所致。解決方法是於終端機視窗中執行以下指令：

```shell
echo fs.inotify.max_user_watches=582222 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

### 錯誤：spawnSync ./gradlew EACCES

若在 macOS 上執行 `npm run android` 或 `yarn android` 時遇到上述錯誤，請嘗試執行 `sudo chmod +x android/gradlew` 指令將 `gradlew` 檔案設為可執行。

[metro]: https://metrobundler.dev/