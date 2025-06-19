---
id: publishing-to-app-store
title: Publishing to Apple App Store
---

發布流程與其他原生 iOS 應用程式相同，但需注意一些額外考量事項。

:::info
若使用 Expo，請參閱 Expo 的[部署至應用商店指南](https://docs.expo.dev/distribution/app-stores/)，以建置並提交應用至 Apple App Store。本指南適用於任何 React Native 應用程式，可自動化部署流程。
:::

### 1. 啟用應用程式傳輸安全性

應用程式傳輸安全性（ATS）是 iOS 9 引入的安全功能，會拒絕所有未透過 HTTPS 傳送的 HTTP 請求。這可能導致 HTTP 流量被封鎖，包括開發用的 React Native 伺服器。React Native 專案預設會停用 `localhost` 的 ATS，以簡化開發流程。

You should re-enable ATS prior to building your app for production by removing the `localhost` entry from the `NSExceptionDomains` dictionary and setting `NSAllowsArbitraryLoads` to `false` in your `Info.plist` file in the `ios/` folder. You can also re-enable ATS from within Xcode by opening your target properties under the Info pane and editing the App Transport Security Settings entry.

:::note
若應用程式需在生產環境存取 HTTP 資源，請學習如何設定專案的 ATS。
:::

### 2. 設定發布方案

為 App Store 發佈建置應用程式時，需使用 Xcode 的 `Release` 方案。`Release` 建置會自動停用應用內開發者選單，避免使用者在生產環境誤觸。同時會將 JavaScript 程式碼本地打包，使裝置離線時仍能測試應用。

要設定應用使用 `Release` 方案建置：前往 **Product** → **Scheme** → **Edit Scheme**。在側邊欄選擇 **Run** 分頁，將 Build Configuration 下拉選單設為 `Release`。

![](/docs/assets/ConfigureReleaseScheme.png)

#### 專業建議

當應用套件體積增長時，可能會在啟動畫面與主應用畫面間看到空白閃爍。若發生此情況，可將以下程式碼加入 `AppDelegate.m`，使過渡期間持續顯示啟動畫面。

```objectivec
  // Place this code after "[self.window makeKeyAndVisible]" and before "return YES;"
  UIStoryboard *sb = [UIStoryboard storyboardWithName:@"LaunchScreen" bundle:nil];
  UIViewController *vc = [sb instantiateInitialViewController];
  rootView.loadingView = vc.view;
```

The static bundle is built every time you target a physical device, even in Debug. If you want to save time, turn off bundle generation in Debug by adding the following to your shell script in the Xcode Build Phase `Bundle React Native code and images`:

```shell
 if [ "${CONFIGURATION}" == "Debug" ]; then
  export SKIP_BUNDLING=true
 fi
```

### 3. 建置發布版應用

現在可透過按下 <kbd>Cmd ⌘</kbd> + <kbd>B</kbd> 或選單列中的 **Product** → **Build** 來建置發布版應用。完成後即可將應用分發給測試人員或提交至 App Store。

:::info
亦可使用 `React Native CLI` 執行此操作：透過 `--mode` 選項指定值為 `Release`（例如在專案根目錄執行：`npm run ios -- --mode="Release"` 或 `yarn ios --mode Release`）。
:::

完成測試並準備發布至 App Store 時，請遵循本指南後續步驟。

- 開啟終端機，導航至應用程式的 iOS 資料夾並輸入 `open .`。
- 雙擊 YOUR_APP_NAME.xcworkspace，這將啟動 Xcode。
- 點擊 `Product` → `Archive`。請確認裝置設定為「Any iOS Device (arm64)」。

:::note
請檢查 Bundle Identifier 是否與 Apple Developer Dashboard 中建立的 Identifiers 完全一致。
:::

- After the archive is completed, in the archive window, click on `Distribute App`.
- Click on `App Store Connect` now (if you want to publish in App Store).
- Click `Upload` → Make sure all the check boxes are selected, hit `Next`.
- Choose between `Automatically manage signing` and `Manually manage signing` based on your needs.
- Click on `Upload`.
- Now you can find it in the App Store Connect under TestFlight.

Now fill up the necessary information and in the Build Section, select the build of the app and click on `Save` → `Submit For Review`.