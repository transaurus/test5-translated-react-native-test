---
id: publishing-to-app-store
title: Publishing to Apple App Store
---

發布流程與其他原生 iOS 應用程式相同，但需注意一些額外考量事項。

:::info
若您使用 Expo，請參閱 Expo 的[部署至應用商店指南](https://docs.expo.dev/distribution/app-stores/)，以建置並提交應用程式至 Apple App Store。本指南適用於任何 React Native 應用程式，可自動化部署流程。
:::

### 1. 設定發布方案

為 App Store 分發建置應用程式時，需在 Xcode 中使用 `Release` 方案。以 `Release` 建置的應用程式會自動停用應用內開發者選單，防止使用者在生產環境中誤觸。同時會將 JavaScript 程式碼本地打包，因此您可將應用程式安裝至裝置並在未連接電腦時進行測試。

要設定應用程式使用 `Release` 方案建置，請前往 **Product** → **Scheme** → **Edit Scheme**。在側邊欄選擇 **Run** 標籤，然後將 Build Configuration 下拉選單設為 `Release`。

![](/docs/assets/ConfigureReleaseScheme.png)

#### 專業建議

當應用程式套件體積增長時，您可能會在啟動畫面與根應用程式視圖顯示之間看到空白閃屏。若發生此情況，可將以下程式碼加入 `AppDelegate.m` 以保持啟動畫面在過渡期間顯示。

```objectivec
  // Place this code after "[self.window makeKeyAndVisible]" and before "return YES;"
  UIStoryboard *sb = [UIStoryboard storyboardWithName:@"LaunchScreen" bundle:nil];
  UIViewController *vc = [sb instantiateInitialViewController];
  rootView.loadingView = vc.view;
```

每次針對實體裝置建置時（即使是 Debug 模式）都會生成靜態套件。若想節省時間，可透過在 Xcode Build Phase 的 `Bundle React Native code and images` shell 腳本中加入以下內容來關閉 Debug 模式下的套件生成：

```shell
 if [ "${CONFIGURATION}" == "Debug" ]; then
  export SKIP_BUNDLING=true
 fi
```

### 2. 為發布建置應用程式

您現在可透過按下 <kbd>Cmd ⌘</kbd> + <kbd>B</kbd> 或從選單欄選擇 **Product** → **Build** 來為發布建置應用程式。完成發布建置後，即可將應用程式分發給測試人員並提交至 App Store。

:::info
您也可使用 `React Native CLI` 執行此操作，透過 `--mode` 選項設定值為 `Release`（例如在專案根目錄執行：`npm run ios -- --mode="Release"` 或 `yarn ios --mode Release`）。
:::

完成測試並準備發布至 App Store 時，請按照本指南後續步驟操作。

- 啟動終端機，導航至應用程式的 iOS 資料夾並輸入 `open .`。
- 雙擊 YOUR_APP_NAME.xcworkspace，這將啟動 XCode。
- 點擊 `Product` → `Archive`。請確保將裝置設為「Any iOS Device (arm64)」。

:::note
請檢查您的 Bundle Identifier，確保與 Apple Developer Dashboard 中建立的 Identifiers 完全一致。
:::

- 歸檔完成後，在歸檔視窗中點擊 `Distribute App`。
- 選擇 `App Store Connect`（若需發布至 App Store）。
- 點擊 `Upload` → 確保所有核取方框已勾選，點擊 `Next`。
- 根據需求選擇 `Automatically manage signing` 或 `Manually manage signing`。
- 點擊 `Upload`。
- 現在您可在 App Store Connect 的 TestFlight 中找到該建置版本。

填寫必要資訊後，在 Build 區段選擇應用程式建置版本，點擊 `Save` → `Submit For Review`。

### 4. 螢幕截圖

Apple Store 要求提供最新裝置的螢幕截圖。相關裝置規格參考可查閱[此處](https://developer.apple.com/help/app-store-connect/reference/screenshot-specifications/)。請注意，若已提供其他尺寸的截圖，部分顯示尺寸的截圖並非必要。