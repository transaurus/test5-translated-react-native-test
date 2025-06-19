---
id: publishing-to-app-store
title: Publishing to Apple App Store
---

發布流程與其他原生 iOS 應用程式相同，但需注意一些額外考量事項。

:::info
若您使用 Expo，請參閱 Expo 的[部署至應用商店指南](https://docs.expo.dev/distribution/app-stores/)來建置並提交應用至 Apple App Store。本指南適用於任何 React Native 應用程式，可自動化部署流程。
:::

### 1. 啟用應用程式傳輸安全性

應用程式傳輸安全性（ATS）是 iOS 9 引入的安全功能，會拒絕所有未透過 HTTPS 傳送的 HTTP 請求。這可能導致 HTTP 流量被阻擋，包括開發用的 React Native 伺服器。為簡化開發流程，React Native 專案預設會停用對 `localhost` 的 ATS。

You should re-enable ATS prior to building your app for production by removing the `localhost` entry from the `NSExceptionDomains` dictionary and setting `NSAllowsArbitraryLoads` to `false` in your `Info.plist` file in the `ios/` folder. You can also re-enable ATS from within Xcode by opening your target properties under the Info pane and editing the App Transport Security Settings entry.

:::note
若您的應用程式需在生產環境存取 HTTP 資源，請學習如何設定專案的 ATS。
:::

### 2. 設定發布方案

為 App Store 分發建置應用程式時，需在 Xcode 中使用 `Release` 方案。以 `Release` 建置的應用程式會自動停用開發者選單，避免使用者在生產環境誤觸。同時會將 JavaScript 程式碼本地打包，讓您能在未連接電腦時於裝置上測試應用程式。

要設定應用程式使用 `Release` 方案建置，請前往 **Product** → **Scheme** → **Edit Scheme**。在側邊欄選擇 **Run** 標籤，然後將 Build Configuration 下拉選單設為 `Release`。

![](/docs/assets/ConfigureReleaseScheme.png)

#### 專業建議

當應用程式套件體積增長時，您可能會在啟動畫面與主應用程式視圖間看到空白閃爍畫面。若發生此情況，可將以下程式碼加入 `AppDelegate.m`，讓啟動畫面在轉場期間持續顯示。

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

### 3. 為發布建置應用程式

現在您可以透過按下 `⌘B` 或從選單選擇 **Product** → **Build** 來為發布建置應用程式。完成發布建置後，即可將應用分發給測試人員並提交至 App Store。

:::info
您也可使用 `React Native CLI` 執行此操作，透過 `--mode` 選項設定為 `Release`（例如 `npx react-native run-ios --mode Release`）。
:::

完成測試並準備發布至 App Store 時，請遵循本指南後續步驟。

- 開啟終端機，導航至應用程式的 iOS 資料夾並輸入 `open .`。
- 雙擊 YOUR_APP_NAME.xcworkspace，這將啟動 Xcode。
- 點擊 `Product` → `Archive`。請確保裝置設定為「Any iOS Device (arm64)」。

:::note
請檢查您的 Bundle Identifier，確認其與 Apple Developer Dashboard 中建立的 Identifiers 完全一致。
:::

- After the archive is completed, in the archive window, click on `Distribute App`.
- Click on `App Store Connect` now (if you want to publish in App Store).
- Click `Upload` → Make sure all the check boxes are selected, hit `Next`.
- Choose between `Automatically manage signing` and `Manually manage signing` based on your needs.
- Click on `Upload`.
- Now you can find it in the App Store Connect under TestFlight.

Now fill up the necessary information and in the Build Section, select the build of the app and click on `Save` → `Submit For Review`.