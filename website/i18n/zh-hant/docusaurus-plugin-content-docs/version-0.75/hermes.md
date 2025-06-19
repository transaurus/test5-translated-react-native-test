---
id: hermes
title: Using Hermes
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<a href="https://hermesengine.dev">
  <img width={300} height={300} className="hermes-logo" src="/docs/assets/HermesLogo.svg" style={{height: "auto"}}/>
</a>

[Hermes](https://hermesengine.dev) 是一個專為 React Native 優化的開源 JavaScript 引擎。對多數應用而言，相較於 JavaScriptCore，使用 Hermes 能帶來更快的啟動速度、更低的記憶體消耗以及更小的應用體積。
React Native 預設已整合 Hermes，無需額外配置即可啟用。

## 內建版 Hermes

React Native 附帶 **內建版本** 的 Hermes。
每當發布新版 React Native 時，我們都會為您構建相應的 Hermes 版本。這確保您使用的 Hermes 與當前 React Native 版本完全兼容。

過去我們曾遇到 Hermes 與 React Native 版本匹配的問題。此方案徹底解決了該問題，為用戶提供與特定 React Native 版本完全兼容的 JS 引擎。

此變更對 React Native 用戶完全透明。您仍可透過本頁面描述的指令停用 Hermes。
您可[在此頁面閱讀技術實現詳情](/architecture/bundled-hermes)。

## 確認 Hermes 已啟用

若您近期新建專案，應能在歡迎視窗中查看 Hermes 啟用狀態：

![AwesomeProject 中查看 JS 引擎狀態的位置](/docs/assets/HermesApp.jpg)

JavaScript 環境中會存在 `HermesInternal` 全域變數，可用於驗證 Hermes 是否啟用：

```jsx
const isHermes = () => !!global.HermesInternal;
```

:::caution
若您使用非標準方式載入 JS 套件包，可能出現 `HermesInternal` 變數存在但未使用高效能預編譯位元組碼的情況。
請確認您使用的是 `.hbc` 檔案，並參照下文進行前後效能比對。
:::

要驗證 Hermes 的優勢，請嘗試建置應用發布版本進行比較。例如在專案根目錄執行：

<Tabs groupId="platform" queryString defaultValue={constants.defaultPlatform} values={constants.platforms} className="pill-tabs">
<TabItem value="android">

[//]: # 'Android'

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run android -- --mode="release"
```

</TabItem>
<TabItem value="yarn">

```shell
yarn android --mode release
```

</TabItem>
</Tabs>

</TabItem>
<TabItem value="ios">

[//]: # 'iOS'

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run ios -- --mode="Release"
```

</TabItem>
<TabItem value="yarn">

```shell
yarn ios --mode Release
```

</TabItem>
</Tabs>

</TabItem>
</Tabs>

此指令會在建置階段將 JavaScript 編譯為位元組碼，從而提升裝置上的應用啟動速度。

## 使用 Chrome 開發者工具調試 Hermes 上的 JS

Hermes 透過實作 Chrome 檢查器協議支援 Chrome 調試器。這意味著可直接使用 Chrome 工具調試運行在模擬器或實體裝置上 Hermes 中的 JavaScript 代碼。

:::info
請注意，這與[調試](debugging#remote-debugging)章節記載的「遠端 JS 調試」（應用內開發選單已棄用功能）有本質區別——後者實際是在開發機（筆電或桌機）的 Chrome V8 引擎上運行 JS 代碼，而非連接裝置上的 JS 引擎。
:::

Chrome 透過 Metro 連接裝置上運行的 Hermes，因此需知曉 Metro 監聽位址。通常為 `localhost:8081`，但此為[可配置項](https://metrobundler.dev/docs/configuration)。執行 `yarn start` 時，該位址會顯示在啟動時的 stdout 中。

確認 Metro 伺服器監聽位址後，可透過以下步驟連接 Chrome：

1. Navigate to `chrome://inspect` in a Chrome browser instance.

2. Use the `Configure...` button to add the Metro server address (typically `localhost:8081` as described above).

![Chrome 開發者工具裝置頁中的配置按鈕](/docs/assets/HermesDebugChromeConfig.png)

![添加 Chrome 開發者工具網路目標的對話框](/docs/assets/HermesDebugChromeMetroAddress.png)

3. 現在您應該會看到一個標示為「Hermes React Native」的目標，並附有「inspect」連結，可用於啟動偵錯工具。如果沒有看到「inspect」連結，請確認 Metro 伺服器正在運行。![目標檢查連結](/docs/assets/HermesDebugChromeInspect.png)

4. 現在您可以使用 Chrome 的偵錯工具。例如，若要在下次執行 JavaScript 時設定中斷點，請點擊暫停按鈕，並在應用程式中觸發會導致 JavaScript 執行的動作。![偵錯工具中的暫停按鈕](/docs/assets/HermesDebugChromePause.png)

## 在舊版 React Native 中啟用 Hermes

自 React Native 0.70 起，Hermes 已成為預設引擎。本節說明如何在舊版 React Native 中啟用 Hermes。
首先，請確保您至少使用 React Native 0.60.4（Android）或 0.64（iOS）版本以啟用 Hermes。

若您現有的應用程式基於較舊的 React Native 版本，您需要先進行升級。請參閱[升級至新版 React Native](/docs/upgrading)了解如何操作。升級應用程式後，請確認一切運作正常，再嘗試切換至 Hermes。

:::caution[React Native 相容性注意事項]
每個 Hermes 版本都針對特定的 RN 版本發布。基本原則是嚴格遵循 [Hermes 發布版本](https://github.com/facebook/hermes/releases)。
版本不匹配可能導致應用程式在最壞情況下立即崩潰。
:::

:::info[Windows 使用者注意事項]
Hermes 需要 [Microsoft Visual C++ 2015 Redistributable](https://www.microsoft.com/en-us/download/details.aspx?id=48145)。
:::

### Android

編輯您的 `android/gradle.properties` 檔案，並確認 `hermesEnabled` 設為 true：

```diff
# Use this property to enable or disable the Hermes JS engine.
# If set to false, you will be using JSC instead.
hermesEnabled=true
```

:::note
此屬性在 React Native 0.71 中新增。如果在 `gradle.properties` 檔案中找不到該屬性，請參考您使用的 React Native 版本對應的文件。
:::

此外，若您使用 ProGuard，則需在 `proguard-rules.pro` 中加入以下規則：

```
-keep class com.facebook.hermes.unicode.** { *; }
-keep class com.facebook.jni.** { *; }
```

接著，若您已至少建構過應用程式一次，請先清除建構：

```shell
$ cd android && ./gradlew clean
```

這樣就完成了！現在您應該可以如常開發和部署應用程式：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run android
```

</TabItem>
<TabItem value="yarn">

```shell
yarn android
```

</TabItem>
</Tabs>

### iOS

自 React Native 0.64 起，Hermes 也可在 iOS 上運行。要為 iOS 啟用 Hermes，請編輯您的 `ios/Podfile` 檔案並進行如下變更：

```diff
   use_react_native!(
     :path => config[:reactNativePath],
     # to enable hermes on iOS, change `false` to `true` and then install pods
     # By default, Hermes is disabled on Old Architecture, and enabled on New Architecture.
     # You can enable/disable it manually by replacing `flags[:hermes_enabled]` with `true` or `false`.
-    :hermes_enabled => flags[:hermes_enabled],
+    :hermes_enabled => true
   )
```

預設情況下，若您使用新架構，將會使用 Hermes。透過指定如 `true` 或 `false` 的值，您可以隨意啟用/停用 Hermes。

設定完成後，您可以透過以下指令安裝 Hermes pods：

```shell
$ cd ios && pod install
```

這樣就完成了！現在您應該可以如常開發和部署應用程式：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run ios
```

</TabItem>
<TabItem value="yarn">

```shell
yarn ios
```

</TabItem>
</Tabs>

## 切換回 JavaScriptCore

React Native 也支援使用 JavaScriptCore 作為 [JavaScript 引擎](javascript-environment)。請遵循以下指示以退出 Hermes。

### Android

編輯您的 `android/gradle.properties` 檔案，並將 `hermesEnabled` 設回 false：

```diff
# Use this property to enable or disable the Hermes JS engine.
# If set to false, you will be using JSC instead.
hermesEnabled=false
```

### iOS

編輯您的 `ios/Podfile` 檔案並進行如下變更：

```diff
   use_react_native!(
     :path => config[:reactNativePath],
+    :hermes_enabled => false,
     # Enables Flipper.
     #
     # Note that if you have use_frameworks! enabled, Flipper will not work and
     # you should disable the next line.
     :flipper_configuration => flipper_config,
     # An absolute path to your application root.
     :app_path => "#{Pod::Config.instance.installation_root}/.."
   )
```