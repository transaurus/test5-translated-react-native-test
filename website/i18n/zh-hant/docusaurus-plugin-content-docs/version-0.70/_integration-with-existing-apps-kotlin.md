## 核心概念

將 React Native 元件整合至 Android 應用的關鍵在於：

1. 設定 React Native 依賴項與目錄結構。
2. 使用 JavaScript 開發 React Native 元件。
3. 在 Android 應用中添加 `ReactRootView`，此視圖將作為 React Native 元件的容器。
4. 啟動 React Native 伺服器並運行原生應用。
5. 驗證應用中的 React Native 功能是否如預期運作。

## 必要條件

請依照[環境設定指南](environment-setup)中的 React Native CLI 快速入門，配置開發環境以建構 Android 版 React Native 應用。

### 1. 設定目錄結構

為確保流程順暢，請為整合的 React Native 專案建立新資料夾，然後將現有的 Android 專案複製到 `/android` 子資料夾中。

### 2. 安裝 JavaScript 依賴項

進入專案的根目錄，並建立包含以下內容的新 `package.json` 檔案：

```
{
  "name": "MyReactNativeApp",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "yarn react-native start"
  }
}
```

接著，請確認已[安裝 yarn 套件管理工具](https://yarnpkg.com/lang/en/docs/install/)。

安裝 `react` 和 `react-native` 套件。開啟終端機或命令提示字元，導航至包含 `package.json` 檔案的目錄並執行：

```shell
$ yarn add react-native
```

這將輸出類似以下的訊息（請在 yarn 輸出中向上滾動以查看）：

> 警告："react-native@0.52.2" 有未滿足的對等依賴項 "react@16.2.0"。

這沒問題，表示我們還需要安裝 React：

```shell
$ yarn add react@version_printed_above
```

Yarn 已建立新的 `/node_modules` 資料夾，此資料夾儲存建構專案所需的所有 JavaScript 依賴項。

請將 `node_modules/` 加入你的 `.gitignore` 檔案。

## 將 React Native 加入應用

### 配置 maven

將 React Native 和 JSC 依賴項加入應用的 `build.gradle` 檔案：

```gradle
dependencies {
    implementation "com.android.support:appcompat-v7:27.1.1"
    ...
    implementation "com.facebook.react:react-native:+" // From node_modules
    implementation "org.webkit:android-jsc:+"
}
```

> 若需確保原生建構中始終使用特定 React Native 版本，請將 `+` 替換為從 `npm` 下載的實際 React Native 版本號。

在頂層 `settings.gradle` 的 "dependencyResolutionManagement" 區塊中，加入本地 React Native 和 JSC 的 maven 目錄條目，並確保其位於其他 maven 儲存庫之上：

```gradle
dependencyResolutionManagement {
    ...
    repositories {
        ...
        maven {
            url "$rootDir/../node_modules/react-native/android"
        }
        maven {
            url("$rootDir/../node_modules/jsc-android/dist")
        }
    }
}
```

> 若專案在頂層 `build.gradle` 中配置了依賴儲存庫，請務必將條目加入 "allprojects" 區塊，並置於其他 maven 儲存庫之上：

```gradle
allprojects {
    repositories {
        maven {
            // All of React Native (JS, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
        maven {
            // Android JSC is installed from npm
            url("$rootDir/../node_modules/jsc-android/dist")
        }
        ...
    }
    ...
}
```

> 請確認路徑正確！在 Android Studio 中執行 Gradle 同步後，不應出現任何「Failed to resolve: com.facebook.react:react-native:0.x.x」錯誤。

### 啟用原生模組自動連結

為使用[自動連結](https://github.com/react-native-community/cli/blob/master/docs/autolinking.md)功能，需在幾處進行配置。首先在 `settings.gradle` 中加入以下條目：

```gradle
apply from: file("../node_modules/@react-native-community/cli-platform-android/native_modules.gradle"); applyNativeModulesSettingsGradle(settings)
```

接著在 `app/build.gradle` 的最底部加入以下條目：

```gradle
apply from: file("../../node_modules/@react-native-community/cli-platform-android/native_modules.gradle"); applyNativeModulesAppBuildGradle(project)
```

### 配置權限

接下來，請確保 `AndroidManifest.xml` 中包含網路權限：

<uses-permission android:name="android.permission.INTERNET" />

如需存取 `DevSettingsActivity`，請在 `AndroidManifest.xml` 中加入：

<activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />

此設定僅在開發模式下用於從開發伺服器重新載入 JavaScript，因此若需釋出版本可將其移除。

### 明文傳輸 (API 等級 28+)

> 從 Android 9 (API 等級 28) 開始，預設禁用明文傳輸；這會阻止應用程式連接到 [Metro 打包器][metro]。以下修改允許在除錯版本中使用明文傳輸。

#### 1. Apply the `usesCleartextTraffic` option to your Debug `AndroidManifest.xml`

```xml
<!-- ... -->
<application
  android:usesCleartextTraffic="true" tools:targetApi="28" >
  <!-- ... -->
</application>
<!-- ... -->
```

釋出版本無需此設定。

關於網路安全配置與明文傳輸政策的詳細說明，請參閱[此連結](https://developer.android.com/training/articles/security-config#CleartextTrafficPermitted)。

### 程式碼整合

現在我們將實際修改原生 Android 應用程式以整合 React Native。

#### React Native 元件

我們要編寫的第一段程式碼是新「高分」畫面的實際 React Native 程式碼，該畫面將整合到我們的應用程式中。

##### 1. 建立 `index.js` 檔案

首先，在 React Native 專案的根目錄建立一個空的 `index.js` 檔案。

`index.js` 是 React Native 應用程式的起點，且為必要檔案。它可以是一個小檔案，用於 `require` 其他屬於 React Native 元件或應用程式的檔案，也可以包含所需的所有程式碼。在此範例中，我們將所有內容放在 `index.js` 中。

##### 2. 加入 React Native 程式碼

In your `index.js`, create your component. In our sample here, we will add a `<Text>` component within a styled `<View>`:

```jsx
import React from 'react';
import {AppRegistry, StyleSheet, Text, View} from 'react-native';

const HelloWorld = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.hello}>Hello, World</Text>
    </View>
  );
};
var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  hello: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
});

AppRegistry.registerComponent(
  'MyReactNativeApp',
  () => HelloWorld,
);
```

##### 3. 設定開發錯誤覆蓋層的權限

If your app is targeting the Android `API level 23` or greater, make sure you have the permission `android.permission.SYSTEM_ALERT_WINDOW` enabled for the development build. You can check this with `Settings.canDrawOverlays(this)`. This is required in dev builds because React Native development errors must be displayed above all the other windows. Due to the new permissions system introduced in the API level 23 (Android M), the user needs to approve it. This can be achieved by adding the following code to your Activity's in `onCreate()` method.

```kotlin
companion object {
    const val OVERLAY_PERMISSION_REQ_CODE = 1  // Choose any value
}

...

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    if(!Settings.canDrawOverlays(this)) {
        val intent = Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
                                    Uri.parse("package: $packageName"))
        startActivityForResult(intent, OVERLAY_PERMISSION_REQ_CODE);
    }
}
```

Finally, the `onActivityResult()` method (as shown in the code below) has to be overridden to handle the permission Accepted or Denied cases for consistent UX. Also, for integrating Native Modules which use `startActivityForResult`, we need to pass the result to the `onActivityResult` method of our `ReactInstanceManager` instance.

```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    if (requestCode == OVERLAY_PERMISSION_REQ_CODE) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (!Settings.canDrawOverlays(this)) {
                // SYSTEM_ALERT_WINDOW permission not granted
            }
        }
    }
    reactInstanceManager?.onActivityResult(this, requestCode, resultCode, data)
}
```

#### 關鍵步驟：`ReactRootView`

讓我們加入一些原生程式碼來啟動 React Native 運行時並告訴它渲染我們的 JS 元件。為此，我們將建立一個 `Activity`，該 Activity 會建立一個 `ReactRootView`，在其中啟動 React 應用程式並將其設為主要內容視圖。

> If you are targeting Android version &lt;5, use the `AppCompatActivity` class from the `com.android.support:appcompat` package instead of `Activity`.

```kotlin
class MyReactActivity : Activity(), DefaultHardwareBackBtnHandler {
    private lateinit var reactRootView: ReactRootView
    private lateinit var reactInstanceManager: ReactInstanceManager
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        SoLoader.init(this, false)
        reactRootView = ReactRootView(this)
        val packages: List<ReactPackage> = PackageList(application).packages
        // Packages that cannot be autolinked yet can be added manually here, for example:
        // packages.add(MyReactNativePackage())
        // Remember to include them in `settings.gradle` and `app/build.gradle` too.
        reactInstanceManager = ReactInstanceManager.builder()
            .setApplication(application)
            .setCurrentActivity(this)
            .setBundleAssetName("index.android.bundle")
            .setJSMainModulePath("index")
            .addPackages(packages)
            .setUseDeveloperSupport(BuildConfig.DEBUG)
            .setInitialLifecycleState(LifecycleState.RESUMED)
            .build()
        // The string here (e.g. "MyReactNativeApp") has to match
        // the string in AppRegistry.registerComponent() in index.js
        reactRootView?.startReactApplication(reactInstanceManager, "MyReactNativeApp", null)
        setContentView(reactRootView)
    }

    override fun invokeDefaultOnBackPressed() {
        super.onBackPressed()
    }
}
```

> 若你使用 React Native 的入門套件，請將 "HelloWorld" 字串替換為你的 index.js 檔案中的字串（這是 `AppRegistry.registerComponent()` 方法的第一個參數）。

執行「Sync Project files with Gradle」操作。

若您使用 Android Studio，請使用 `Alt + Enter` 為您的 MyReactActivity 類別新增所有缺失的導入項目。請注意使用您套件的 `BuildConfig`，而非 `facebook` 套件中的版本。

我們需要將 `MyReactActivity` 的主題設為 `Theme.AppCompat.Light.NoActionBar`，因為部分 React Native UI 元件依賴此主題。

```xml
<activity
  android:name=".MyReactActivity"
  android:label="@string/app_name"
  android:theme="@style/Theme.AppCompat.Light.NoActionBar">
</activity>
```

> A `ReactInstanceManager` can be shared by multiple activities and/or fragments. You will want to make your own `ReactFragment` or `ReactActivity` and have a singleton _holder_ that holds a `ReactInstanceManager`. When you need the `ReactInstanceManager` (e.g., to hook up the `ReactInstanceManager` to the lifecycle of those Activities or Fragments) use the one provided by the singleton.

接著，我們需將部分活動生命週期回調傳遞給 `ReactInstanceManager` 和 `ReactRootView`：

```kotlin
override fun onPause() {
    super.onPause()
    reactInstanceManager.onHostPause(this)
}

override fun onResume() {
    super.onResume()
    reactInstanceManager.onHostResume(this, this)
}

override fun onDestroy() {
    super.onDestroy()
    reactInstanceManager.onHostDestroy(this)
    reactRootView.unmountReactApplication()
}
```

我們還需將返回按鈕事件傳遞給 React Native：

```kotlin
override fun onBackPressed() {
    reactInstanceManager.onBackPressed()
    super.onBackPressed()
}
```

這允許 JavaScript 控制使用者按下硬體返回鍵時的行為（例如實現導航）。若 JavaScript 未處理返回按鈕事件，則會呼叫您的 `invokeDefaultOnBackPressed` 方法。預設情況下，這將結束您的 `Activity`。

最後，我們需啟用開發者選單。預設情況下，這需搖動裝置觸發，但在模擬器中並不實用。因此我們設定當按下硬體選單鍵時顯示（若使用 Android Studio 模擬器，請按 `Ctrl + M`）：

```kotlin
override fun onKeyUp(keyCode: Int, event: KeyEvent?): Boolean {
    if (keyCode == KeyEvent.KEYCODE_MENU && reactInstanceManager != null) {
        reactInstanceManager.showDevOptionsDialog()
        return true
    }
    return super.onKeyUp(keyCode, event)
}
```

現在您的活動已準備好執行 JavaScript 程式碼。

### 測試整合結果

您已完成整合 React Native 與現有應用程式的基本步驟。現在我們將啟動 [Metro 打包工具][metro] 來建構 `index.bundle` 套件，並運行本地伺服器來提供該套件。

##### 1. 執行打包伺服器

要運行您的應用程式，需先啟動開發伺服器。為此，請在 React Native 專案的根目錄執行以下命令：

```shell
$ yarn start
```

##### 2. 運行應用程式

現在如常建構並運行您的 Android 應用程式。

當進入應用程式中由 React 驅動的活動時，它應從開發伺服器載入 JavaScript 程式碼並顯示：

![螢幕截圖](/docs/assets/EmbeddedAppAndroid.png)

### 在 Android Studio 中建立正式版建構

您也可使用 Android Studio 建立正式版建構！步驟與為既有原生 Android 應用建立正式版同樣簡便。僅需在每次正式建構前多執行一個步驟：需先執行以下命令建立 React Native 套件，該套件將包含在您的原生 Android 應用中：

```shell
$ npx react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/com/your-company-name/app-package-name/src/main/assets/index.android.bundle --assets-dest android/com/your-company-name/app-package-name/src/main/res/
```

> 請注意替換為正確路徑，若 assets 資料夾不存在請先建立。

現在，如常透過 Android Studio 建立原生應用的正式版建構即可！

### 後續步驟

至此，您可繼續常規開發。請參考我們的[除錯指南](debugging)與[部署文件](running-on-device)以深入瞭解 React Native 開發。

[metro]: https://metrobundler.dev/