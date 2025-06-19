## 核心概念

將 React Native 元件整合至 Android 應用程式的關鍵在於：

1. 設定 React Native 依賴項與目錄結構。
2. 使用 JavaScript 開發 React Native 元件。
3. 在 Android 應用程式中加入 `ReactRootView`，此視圖將作為 React Native 元件的容器。
4. 啟動 React Native 伺服器並執行原生應用程式。
5. 驗證應用程式中 React Native 的部分是否如預期運作。

## 必要條件

請遵循[環境設定指南](environment-setup)中的 React Native CLI 快速入門，為建置 Android 版 React Native 應用程式設定開發環境。

### 1. 設定目錄結構

為確保流程順暢，請為整合的 React Native 專案建立新資料夾，然後將現有的 Android 專案複製到 `/android` 子資料夾中。

### 2. 安裝 JavaScript 依賴項

前往專案的根目錄，並建立包含以下內容的新 `package.json` 檔案：

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

接著，請確認您已[安裝 yarn 套件管理工具](https://yarnpkg.com/lang/en/docs/install/)。

安裝 `react` 和 `react-native` 套件。開啟終端機或命令提示字元，導航至包含 `package.json` 檔案的目錄，然後執行：

```shell
$ yarn add react-native
```

這將顯示類似以下的訊息（請在 yarn 輸出中向上滾動以查看）：

> 警告："react-native@0.70.5" 有未滿足的 peer 依賴項 "react@18.1.0"

這是正常的，表示我們還需要安裝 React：

```shell
$ yarn add react@version_printed_above
```

Yarn 已建立新的 `/node_modules` 資料夾。此資料夾儲存建置專案所需的所有 JavaScript 依賴項。

將 `node_modules/` 加入您的 `.gitignore` 檔案。

## 將 React Native 加入您的應用程式

### 設定 Gradle

React Native 使用 React Native Gradle 外掛程式來設定您的依賴項與專案配置。

首先，請編輯您的 `settings.gradle` 檔案，加入這一行：

```groovy
includeBuild('../node_modules/react-native-gradle-plugin')
```

接著，您需要開啟頂層的 `build.gradle` 並包含這一行：

```diff
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath("com.android.tools.build:gradle:7.3.1")
+       classpath("com.facebook.react:react-native-gradle-plugin")
    }
}
```

這確保 React Native Gradle 外掛程式在您的專案中可用。
最後，在您應用程式的 `build.gradle` 檔案（位於應用程式資料夾內的另一個 `build.gradle` 檔案）中加入以下幾行：

```diff
apply plugin: "com.android.application"
+apply plugin: "com.facebook.react"

repositories {
    mavenCentral()
}

dependencies {
    // Other dependencies here
+   implementation "com.facebook.react:react-android"
+   implementation "com.facebook.react:hermes-android"
}
```

這些依賴項可在 `mavenCentral()` 上取得，因此請確認您已在 `repositories{}` 區塊中定義它。

:::info
我們刻意不為這些 `implementation` 依賴項指定版本，因為 React Native Gradle 外掛程式會負責處理。若您未使用 React Native Gradle 外掛程式，則需手動指定版本。
:::

### 啟用原生模組自動連結

若要使用[自動連結](https://github.com/react-native-community/cli/blob/master/docs/autolinking.md)的功能，我們需在幾個地方進行設定。首先在 `settings.gradle` 中加入以下項目：

```gradle
apply from: file("../node_modules/@react-native-community/cli-platform-android/native_modules.gradle"); applyNativeModulesSettingsGradle(settings)
```

接著在 `app/build.gradle` 的最底部加入以下項目：

```gradle
apply from: file("../../node_modules/@react-native-community/cli-platform-android/native_modules.gradle"); applyNativeModulesAppBuildGradle(project)
```

### 設定權限

接下來，請確認您的 `AndroidManifest.xml` 中有網路權限：

<uses-permission android:name="android.permission.INTERNET" />

如果需要存取 `DevSettingsActivity`，請在 `AndroidManifest.xml` 中加入：

<activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />

這僅在開發模式下用於從開發伺服器重新載入 JavaScript，因此如果需要，您可以在正式版本中移除此設定。

### 明文流量（API 等級 28+）

> 從 Android 9（API 等級 28）開始，預設禁用明文流量；這會阻止您的應用程式連接到 [Metro 打包器][metro]。以下變更允許在除錯版本中使用明文流量。

#### 1. 將 `usesCleartextTraffic` 選項套用至您的除錯版 `AndroidManifest.xml`

```xml
<!-- ... -->
<application
  android:usesCleartextTraffic="true" tools:targetApi="28" >
  <!-- ... -->
</application>
<!-- ... -->
```

正式版本不需要此設定。

如需了解更多關於網路安全設定和明文流量政策的資訊，[請參閱此連結](https://developer.android.com/training/articles/security-config#CleartextTrafficPermitted)。

### 程式碼整合

現在我們將實際修改原生 Android 應用程式以整合 React Native。

#### React Native 元件

我們將編寫的第一段程式碼是用於整合到我們應用程式中的新「高分」畫面的實際 React Native 程式碼。

##### 1. 建立 `index.js` 檔案

首先，在 React Native 專案的根目錄中建立一個空的 `index.js` 檔案。

`index.js` 是 React Native 應用程式的起點，並且始終是必需的。它可以是一個小檔案，`require` 其他屬於您的 React Native 元件或應用程式的檔案，或者它可以包含所需的所有程式碼。在我們的例子中，我們將把所有內容放在 `index.js` 中。

##### 2. 新增您的 React Native 程式碼

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

##### 3. 為開發錯誤覆蓋層設定權限

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

#### 魔法：`ReactRootView`

讓我們新增一些原生程式碼以啟動 React Native 運行時並告訴它渲染我們的 JS 元件。為此，我們將建立一個 `Activity`，該 Activity 建立一個 `ReactRootView`，在其中啟動一個 React 應用程式並將其設為主要內容視圖。

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

> 如果您使用的是 React Native 的入門套件，請將 "HelloWorld" 字串替換為您的 index.js 檔案中的字串（這是 `AppRegistry.registerComponent()` 方法的第一個參數）。

執行「Sync Project files with Gradle」操作。

若您使用 Android Studio，可使用 `Alt + Enter` 為您的 MyReactActivity 類別新增所有缺失的導入項目。請注意使用您套件的 `BuildConfig`，而非 `facebook` 套件中的版本。

我們需將 `MyReactActivity` 的主題設為 `Theme.AppCompat.Light.NoActionBar`，因部分 React Native UI 元件依賴此主題。

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

還需將返回按鈕事件傳遞給 React Native：

```kotlin
override fun onBackPressed() {
    reactInstanceManager.onBackPressed()
    super.onBackPressed()
}
```

這允許 JavaScript 控制硬體返回按鈕的行為（例如實現導航）。若 JavaScript 未處理返回按鈕事件，則會呼叫 `invokeDefaultOnBackPressed` 方法，預設情況下會結束當前 `Activity`。

最後，需設定開發者選單。預設透過搖動裝置觸發，但在模擬器中不實用。因此改為按下硬體選單按鈕觸發（Android Studio 模擬器請使用 `Ctrl + M`）：

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

您已完成整合 React Native 與現有應用程式的基本步驟。現在我們將啟動 [Metro 打包工具][metro] 來建置 `index.bundle` 套件，並在本地端啟動伺服器提供服務。

##### 1. 執行打包伺服器

執行應用前需先啟動開發伺服器。在 React Native 專案根目錄執行以下指令：

```shell
$ yarn start
```

##### 2. 執行應用程式

現在如常建置並執行您的 Android 應用程式。

當進入應用內 React 驅動的活動時，應會從開發伺服器載入 JavaScript 程式碼並顯示：

![螢幕截圖](/docs/assets/EmbeddedAppAndroid.png)

### 在 Android Studio 建立正式版建置

您也可使用 Android Studio 建立正式版建置！流程與建置既有原生 Android 應用程式相同。

若依前述方式使用 React Native Gradle 插件，從 Android Studio 執行應用時應能正常運作。

若未使用 React Native Gradle 插件，每次正式建置前需額外執行以下指令來建立 React Native 套件（該套件將與原生 Android 應用程式一併發布）：

```shell
$ npx react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/com/your-company-name/app-package-name/src/main/assets/index.android.bundle --assets-dest android/com/your-company-name/app-package-name/src/main/res/
```

> 請注意替換正確路徑，若 assets 資料夾不存在請先建立。

現在如常透過 Android Studio 建立原生應用程式的正式版建置即可！

### 後續步驟

至此您可繼續常規開發。請參閱我們的[除錯指南](debugging)與[部署文件](running-on-device)以深入瞭解 React Native 開發。

[metro]: https://metrobundler.dev/