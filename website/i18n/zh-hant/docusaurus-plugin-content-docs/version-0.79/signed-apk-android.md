---
id: signed-apk-android
title: Publishing to Google Play Store
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

Android要求所有應用程式在安裝前必須使用憑證進行數位簽署。若要在[Google Play商店](https://play.google.com/store)發佈Android應用程式，需使用發布金鑰簽署，且後續更新都需使用同一金鑰。自2017年起，透過[Google Play應用程式簽署](https://developer.android.com/studio/publish/app-signing#app-signing-google-play)功能，Google Play可自動管理簽署流程。但上傳至Google Play前，應用程式需先以「上傳金鑰」簽署。Android開發者文件中的[簽署您的應用程式](https://developer.android.com/tools/publishing/app-signing.html)頁面有詳細說明。本指南將簡要介紹流程，並列出封裝JavaScript套件包的步驟。

:::info
若使用Expo，請參閱Expo指南中的[部署至應用商店](https://docs.expo.dev/distribution/app-stores/)，了解如何建構並提交應用程式至Google Play商店。本指南適用於任何React Native應用程式，可自動化部署流程。
:::

## 產生上傳金鑰

您可以使用`keytool`產生私密簽署金鑰。

### Windows

On Windows `keytool` must be run from `C:\Program Files\Java\jdkx.x.x_x\bin`, as administrator.

```shell
keytool -genkeypair -v -storetype PKCS12 -keystore my-upload-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

此命令會提示您輸入金鑰庫和金鑰的密碼，以及金鑰的辨別名稱欄位。接著會產生名為`my-upload-key.keystore`的金鑰庫檔案。

金鑰庫包含單一金鑰，有效期為10000天。別名是後續簽署應用程式時使用的名稱，請務必記下。

### macOS

在macOS上，若不確定JDK的bin目錄位置，可執行以下命令查詢：

```shell
/usr/libexec/java_home
```

命令會輸出JDK目錄，路徑類似如下：

```shell
/Library/Java/JavaVirtualMachines/jdkX.X.X_XXX.jdk/Contents/Home
```

使用`cd /your/jdk/path`切換至該目錄，並以sudo權限執行如下所示的keytool命令。

```shell
sudo keytool -genkey -v -keystore my-upload-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

:::caution
請妥善保管金鑰庫檔案。若遺失上傳金鑰或遭洩露，應[遵循這些指示](https://support.google.com/googleplay/android-developer/answer/7384423#reset)處理。
:::

## 設定Gradle變數

1. 將`my-upload-key.keystore`檔案置於專案目錄的`android/app`下。
2. 編輯`~/.gradle/gradle.properties`或`android/gradle.properties`檔案，加入以下內容（將`*****`替換為正確的金鑰庫密碼、別名和金鑰密碼）：

```
MYAPP_UPLOAD_STORE_FILE=my-upload-key.keystore
MYAPP_UPLOAD_KEY_ALIAS=my-key-alias
MYAPP_UPLOAD_STORE_PASSWORD=*****
MYAPP_UPLOAD_KEY_PASSWORD=*****
```

這些將成為全域Gradle變數，後續可在Gradle配置中用於簽署應用程式。

:::note[關於git使用的注意事項]
將上述Gradle變數儲存在`~/.gradle/gradle.properties`而非`android/gradle.properties`中，可避免被提交至git。您可能需先在用戶主目錄中建立`~/.gradle/gradle.properties`檔案才能加入變數。
:::

:::note[關於安全性的注意事項]
若不願以明文儲存密碼，且使用macOS，可[將憑證儲存在Keychain Access應用程式中](https://pilloxa.gitlab.io/posts/safer-passwords-in-gradle/)。如此便可省略`~/.gradle/gradle.properties`中的最後兩行。
:::

## 在應用程式的Gradle配置中加入簽署設定

最後一個需要完成的配置步驟是設定使用上傳金鑰簽署正式版建置。編輯專案目錄中的 `android/app/build.gradle` 檔案，並加入簽署配置：

```groovy
...
android {
    ...
    defaultConfig { ... }
    signingConfigs {
        release {
            if (project.hasProperty('MYAPP_UPLOAD_STORE_FILE')) {
                storeFile file(MYAPP_UPLOAD_STORE_FILE)
                storePassword MYAPP_UPLOAD_STORE_PASSWORD
                keyAlias MYAPP_UPLOAD_KEY_ALIAS
                keyPassword MYAPP_UPLOAD_KEY_PASSWORD
            }
        }
    }
    buildTypes {
        release {
            ...
            signingConfig signingConfigs.release
        }
    }
}
...
```

## 產生正式版 AAB

在終端機中執行以下指令：

```shell
npx react-native build-android --mode=release
```

此指令底層使用 Gradle 的 `bundleRelease`，會將應用程式執行所需的所有 JavaScript 程式碼打包至 AAB ([Android App Bundle](https://developer.android.com/guide/app-bundle))。若需變更 JavaScript 套件或繪圖資源的打包方式（例如變更預設檔案/資料夾名稱或專案結構），請參閱 `android/app/build.gradle` 以瞭解如何調整設定。

:::note
請確認 `gradle.properties` 未包含 `org.gradle.configureondemand=true`，否則正式版建置將跳過將 JS 與資源打包至應用程式二進位檔的步驟。
:::

產生的 AAB 檔案位於 `android/app/build/outputs/bundle/release/app-release.aab`，可直接上傳至 Google Play。

Google Play 需在 Google Play 控制台為您的應用程式設定「Google Play 應用程式簽署」功能才能接受 AAB 格式。若您要更新未使用此功能的現有應用程式，請參閱我們的[遷移章節](#migrating-old-android-react-native-apps-to-use-app-signing-by-google-play)瞭解如何變更配置。

## 測試應用程式正式版

上傳正式版至 Play 商店前，請務必徹底測試。先解除安裝裝置上所有舊版應用程式，再於專案根目錄執行以下指令進行安裝：

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

注意：僅有當您完成上述簽署設定後，才能使用 `--mode release` 參數。

您可以終止所有正在執行的打包程序，因為所有框架與 JavaScript 程式碼都已打包至 APK 資源中。

## 發布至其他商店

預設產生的 APK 包含 `x86`、`x86_64`、`ARMv7a` 和 `ARM64-v8a` 四種 CPU 架構的原生程式碼，使單一 APK 能相容多數 Android 裝置，但會導致 APK 因包含未使用的原生程式碼而體積膨脹。

您可以在 `android/app/build.gradle` 中加入以下設定，為每種 CPU 架構產生獨立 APK：

```diff
android {

    splits {
        abi {
            reset()
            enable true
            universalApk false
            include "armeabi-v7a", "arm64-v8a", "x86", "x86_64"
        }
    }

}
```

將這些檔案上傳至支援裝置篩選的商店（如 [Amazon AppStore](https://developer.amazon.com/docs/app-submission/device-filtering-and-compatibility.html) 或 [F-Droid](https://f-droid.org/en/)），使用者將自動取得對應的 APK。若需上傳至 [APKFiles](https://www.apkfiles.com/) 等不支援多 APK 的商店，請將 `universalApk false` 改為 `true` 以產生包含所有 CPU 架構的通用 APK。

請注意，您還需依[官方 Android 文件建議](https://developer.android.com/studio/build/configure-apk-splits#configure-APK-versions)設定不同的版本代碼。

## 啟用 Proguard 縮減 APK 體積（選用）

Proguard 工具能透過移除未使用的 React Native Java 位元碼（及其依賴庫）來小幅縮減 APK 體積。

:::caution[重要]
啟用 Proguard 後務必徹底測試應用程式。Proguard 通常需針對每個原生函式庫進行個別配置，請參閱 `app/proguard-rules.pro`。
:::

要啟用 Proguard，請編輯 `android/app/build.gradle`：

```groovy
/**
 * Run Proguard to shrink the Java bytecode in release builds.
 */
def enableProguardInReleaseBuilds = true
```

## 遷移舊版 Android React Native 應用程式以使用 Google Play 應用程式簽署

如果您正在從舊版 React Native 遷移，您的應用程式可能尚未使用 Google Play 應用程式簽署功能。我們建議您啟用此功能，以充分利用自動應用程式拆分等優勢。要從舊版簽署方式遷移，您需要先[生成新的上傳金鑰](#generating-an-upload-key)，然後在 `android/app/build.gradle` 中將發佈簽署配置替換為使用上傳金鑰而非發佈金鑰（請參閱[將簽署配置添加到 gradle](#adding-signing-config-to-your-apps-gradle-config) 部分）。完成後，請按照 [Google Play 幫助網站上的說明](https://support.google.com/googleplay/android-developer/answer/7384423) 將您的原始發佈金鑰發送給 Google Play。

## 預設權限

預設情況下，您的 Android 應用程式會添加 `INTERNET` 權限，因為幾乎所有應用程式都會使用它。`SYSTEM_ALERT_WINDOW` 權限會在偵錯模式下添加到您的 Android APK 中，但在正式版中會被移除。