---
id: publishing-to-app-store
title: Publishing to Apple App Store
---

發布流程與其他原生 iOS 應用程式相同，但需注意一些額外事項。

:::info
若您使用 Expo，請參閱 Expo 的 [部署至應用商店指南](https://docs.expo.dev/distribution/app-stores/) 來建置並提交應用至 Apple App Store。本指南適用於任何 React Native 應用程式，可自動化部署流程。
:::

### 1. 配置發布方案

為 App Store 分發建置應用需在 Xcode 中使用 `Release` 方案。以 `Release` 建置的應用會自動停用應用內開發者選單，避免使用者在生產環境中誤觸。同時會將 JavaScript 程式碼本地打包，使您能在未連接電腦時於裝置上測試應用。

配置應用使用 `Release` 方案建置：前往 **Product** → **Scheme** → **Edit Scheme**。在側邊欄選擇 **Run** 標籤，並將 Build Configuration 下拉選單設為 `Release`。

![](/docs/assets/ConfigureReleaseScheme.png)

#### 專業建議

當應用套件體積增長時，您可能會在啟動畫面與根應用視圖之間看到空白閃屏。若發生此情況，可將以下程式碼加入 `AppDelegate.m` 以在過渡期間保持啟動畫面顯示。

```objectivec
  // Place this code after "[self.window makeKeyAndVisible]" and before "return YES;"
  UIStoryboard *sb = [UIStoryboard storyboardWithName:@"LaunchScreen" bundle:nil];
  UIViewController *vc = [sb instantiateInitialViewController];
  rootView.loadingView = vc.view;
```

每次針對實體裝置建置時（即使是 Debug 模式）都會生成靜態套件。若需節省時間，可透過在 Xcode Build Phase 的 `Bundle React Native code and images` shell 腳本中加入以下內容來關閉 Debug 模式的套件生成：

```shell
 if [ "${CONFIGURATION}" == "Debug" ]; then
  export SKIP_BUNDLING=true
 fi
```

### 2. 建置發布版應用

現在可透過按下 <kbd>Cmd ⌘</kbd> + <kbd>B</kbd> 或從選單欄選擇 **Product** → **Build** 來建置發布版應用。發布版建置完成後，即可分發應用給測試人員並提交至 App Store。

:::info
亦可使用 `React Native CLI` 執行此操作，透過 `--mode` 選項設定值為 `Release`（例如在專案根目錄執行：`npm run ios -- --mode="Release"` 或 `yarn ios --mode Release`）。
:::

完成測試並準備發布至 App Store 時，請依循本指南操作。

- Launch your terminal, and navigate into the iOS folder of your app and type `open .`.
- Double click on YOUR_APP_NAME.xcworkspace. It should launch XCode.
- Click on `Product` → `Archive`. Make sure to set the device to "Any iOS Device (arm64)".

:::note
請檢查 Bundle Identifier 是否與 Apple Developer Dashboard 中建立的 Identifiers 完全一致。
:::

- After the archive is completed, in the archive window, click on `Distribute App`.
- Click on `App Store Connect` now (if you want to publish in App Store).
- Click `Upload` → Make sure all the check boxes are selected, hit `Next`.
- Choose between `Automatically manage signing` and `Manually manage signing` based on your needs.
- Click on `Upload`.
- Now you can find it in the App Store Connect under TestFlight.

填寫必要資訊後，在 Build 區段選擇應用建置版本，點擊 `Save` → `Submit For Review`。