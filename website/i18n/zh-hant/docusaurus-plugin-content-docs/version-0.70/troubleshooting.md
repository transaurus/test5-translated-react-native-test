---
id: troubleshooting
title: Troubleshooting
---

以下是設定 React Native 時可能遇到的一些常見問題。若您遇到的問題未列於此，請嘗試[在 GitHub 上搜尋相關議題](https://github.com/facebook/react-native/issues/)。

### 連接埠已被佔用

[Metro 打包工具][metro]預設使用 8081 連接埠。若該連接埠已被其他程序佔用，您可以選擇終止該程序，或變更打包工具使用的連接埠。

#### 終止佔用 8081 連接埠的程序

執行以下指令找出正在監聽 8081 連接埠的程序 ID：

```shell
sudo lsof -i :8081
```

接著執行以下指令終止該程序：

```shell
kill -9 <PID>
```

在 Windows 系統上，您可以使用[資源監視器](https://stackoverflow.com/questions/48198/how-can-you-find-out-which-process-is-listening-on-a-port-on-windows)找出佔用 8081 連接埠的程序，並透過工作管理員將其關閉。

#### 使用 8081 以外的連接埠

您可透過 `port` 參數設定打包工具使用其他連接埠：

```shell
npx react-native start --port=8088
```

You will also need to update your applications to load the JavaScript bundle from the new port. If running on device from Xcode, you can do this by updating occurrences of `8081` to your chosen port in the `ios/__App_Name__.xcodeproj/project.pbxproj` file.

### NPM 鎖定錯誤

若使用 React Native CLI 時遇到類似 `npm WARN locking Error: EACCES` 的錯誤，請嘗試執行以下指令：

```shell
sudo chown -R $USER ~/.npm
sudo chown -R $USER /usr/local/lib/node_modules
```

### 缺少 React 相關函式庫

若您手動將 React Native 加入專案，請確認已包含所有使用到的相依函式庫，例如 `RCTText.xcodeproj`、`RCTImage.xcodeproj`。接著需將這些函式庫編譯的二進位檔連結至您的應用程式。請於 Xcode 專案設定的 `Linked Frameworks and Binaries` 區段進行設定。詳細步驟請參閱：[連結函式庫](linking-libraries-ios.md#content)。

若使用 CocoaPods，請確認已於 `Podfile` 中加入 React 及其子規格。例如，若您使用 `<Text />`、`<Image />` 與 `fetch()` API，需在 `Podfile` 中加入以下內容：

```
pod 'React', :path => '../node_modules/react-native', :subspecs => [
  'RCTText',
  'RCTImage',
  'RCTNetwork',
  'RCTWebSocket',
]
```

接著請確認已執行 `pod install` 且專案中已建立包含 React 的 `Pods/` 目錄。CocoaPods 會指示您後續需使用產生的 `.xcworkspace` 檔案以使用這些安裝的相依套件。

#### 作為 CocoaPod 使用時 React Native 無法編譯

可使用名為 [cocoapods-fix-react-native](https://github.com/orta/cocoapods-fix-react-native) 的 CocoaPods 外掛，該外掛會處理因使用相依性管理工具而可能產生的原始碼後置修正問題。

#### 參數列表過長：遞迴標頭檔擴展失敗

在專案的建置設定中，`User Search Header Paths` 與 `Header Search Paths` 是兩個指定 Xcode 應於何處尋找程式碼中 `#import` 標頭檔的設定。對於 Pods，CocoaPods 使用預設的特定資料夾路徑陣列。請確認此設定未被覆寫，且設定的資料夾路徑沒有過大。若其中一個資料夾過大，Xcode 會嘗試遞迴搜尋整個目錄並最終拋出上述錯誤。

若要將 `User Search Header Paths` 與 `Header Search Paths` 建置設定恢復為 CocoaPods 的預設值，請於建置設定面板中選取該項目並按下刪除鍵。此操作會移除自訂覆寫並恢復為 CocoaPods 預設值。

### 無可用傳輸方式

React Native 為 WebSockets 實現了 polyfill。這些 [polyfills](https://github.com/facebook/react-native/blob/0.70-stable/Libraries/Core/InitializeCore.js) 會作為 react-native 模組的一部分初始化，你可以透過 `import React from 'react'` 將其引入應用程式。如果你需要載入另一個依賴 WebSockets 的模組（例如 [Firebase](https://github.com/facebook/react-native/issues/3645)），請確保在 react-native 之後載入/引入：

```
import React from 'react';
import Firebase from 'firebase';
```

## Shell 命令無回應異常

如果遇到 ShellCommandUnresponsiveException 異常，例如：

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

當你正在除錯程序或需要更詳細的錯誤資訊時，可以使用詳細選項輸出更多日誌和資訊來定位問題。

在根目錄下執行以下命令。

```shell
npx react-native run-android --verbose
```

## 無法啟動 react-native 套件管理器（Linux 系統）

### 情況 1：錯誤 "code":"ENOSPC","errno":"ENOSPC"

此問題是由於 [inotify](https://github.com/guard/listen/wiki/Increasing-the-amount-of-inotify-watchers)（Linux 上由 watchman 使用）可監控的目錄數量限制所導致。解決方法是執行以下終端命令：

```shell
echo fs.inotify.max_user_watches=582222 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

### 錯誤：spawnSync ./gradlew EACCES

如果在 macOS 上執行 `npm run android` 時遇到上述錯誤，請嘗試執行 `sudo chmod +x android/gradlew` 命令，將 `gradlew` 文件設為可執行。

[metro]: https://metrobundler.dev/