## 核心概念

將 React Native 元件整合至 Android 應用的關鍵在於：

1. 設定 React Native 依賴項與目錄結構
2. 使用 JavaScript 開發 React Native 元件
3. 在 Android 應用中添加 `ReactRootView`，該視圖將作為 React Native 元件的容器
4. 啟動 React Native 伺服器並運行原生應用
5. 驗證應用的 React Native 功能是否符合預期

## 必要條件

請依照[環境設定指南](environment-setup)中的 React Native CLI 快速入門，配置 Android 版 React Native 應用的開發環境。

### 1. 設定目錄結構

為確保流程順暢，請為整合專案新建資料夾，並將現有 Android 專案複製到 `/android` 子資料夾中。

### 2. 安裝 JavaScript 依賴項

進入專案根目錄，建立包含以下內容的 `package.json` 檔案：

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

接著請確認已[安裝 yarn 套件管理工具](https://yarnpkg.com/lang/en/docs/install/)。

安裝 `react` 和 `react-native` 套件。開啟終端機或命令提示字元，導航至含 `package.json` 的目錄並執行：

```shell
$ yarn add react-native
```

這將輸出類似以下的訊息（請在 yarn 輸出中向上滾動查看）：

> 警告："react-native@0.70.5" 有未滿足的 peer 依賴項 "react@18.1.0"

這是正常現象，表示我們還需要安裝 React：

```shell
$ yarn add react@version_printed_above
```

Yarn 已建立新的 `/node_modules` 資料夾，此資料夾存放專案建構所需的所有 JavaScript 依賴項。

請將 `node_modules/` 加入 `.gitignore` 檔案。

## 將 React Native 加入應用

### 配置 Gradle

React Native 使用 React Native Gradle 插件來配置依賴項與專案設定。

首先編輯 `settings.gradle` 檔案，加入這行程式碼：

```groovy
includeBuild('../node_modules/react-native-gradle-plugin')
```

接著開啟頂層的 `build.gradle` 並包含這行程式碼：

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

這確保 React Native Gradle 插件能在專案中使用。
最後在應用模組的 `build.gradle` 檔案（位於應用資料夾內的另一個 `build.gradle` 檔案）中加入以下程式碼：

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

這些依賴項可從 `mavenCentral()` 取得，請確認已在 `repositories{}` 區塊中定義。

:::info
我們刻意不指定這些 `implementation` 依賴項的版本，因為 React Native Gradle 插件會自動處理。若未使用該插件，則需手動指定版本。
:::

### 啟用原生模組自動連結

要使用[自動連結](https://github.com/react-native-community/cli/blob/master/docs/autolinking.md)功能，需在幾個地方進行配置。首先在 `settings.gradle` 中加入：

```gradle
apply from: file("../node_modules/@react-native-community/cli-platform-android/native_modules.gradle"); applyNativeModulesSettingsGradle(settings)
```

接著在 `app/build.gradle` 最底部加入：

```gradle
apply from: file("../../node_modules/@react-native-community/cli-platform-android/native_modules.gradle"); applyNativeModulesAppBuildGradle(project)
```

### 配置權限

接著確認 `AndroidManifest.xml` 中包含網路權限：

<uses-permission android:name="android.permission.INTERNET" />

如需存取 `DevSettingsActivity`，請在 `AndroidManifest.xml` 中加入：

<activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />

此設定僅在開發模式下用於從開發伺服器重新載入 JavaScript，因此若需精簡發行版本，可移除該設定。

### 明文流量（API 等級 28+）

> 從 Android 9（API 等級 28）開始，預設禁用明文流量；這會阻止應用程式連接到 [Metro 打包工具][metro]。以下變更允許在偵錯版本中使用明文流量。

#### 1. 將 `usesCleartextTraffic` 選項套用至偵錯版 `AndroidManifest.xml`

```xml
<!-- ... -->
<application
  android:usesCleartextTraffic="true" tools:targetApi="28" >
  <!-- ... -->
</application>
<!-- ... -->
```

發行版本無需此設定。

深入了解網路安全設定與明文流量政策，請[參閱此連結](https://developer.android.com/training/articles/security-config#CleartextTrafficPermitted)。

### 程式碼整合

現在我們將實際修改原生 Android 應用程式以整合 React Native。

#### React Native 元件

我們將編寫的第一段程式碼是新「高分」畫面的實際 React Native 程式碼，該畫面將整合至我們的應用程式中。

##### 1. 建立 `index.js` 檔案

首先，在 React Native 專案的根目錄中建立一個空的 `index.js` 檔案。

`index.js` 是 React Native 應用程式的起點，且為必備檔案。它可以是一個小檔案，用於 `require` 其他屬於 React Native 元件或應用程式的檔案，也可以包含所需的所有程式碼。在本例中，我們將所有內容放入 `index.js`。

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

##### 3. 設定開發錯誤疊加層的權限

If your app is targeting the Android `API level 23` or greater, make sure you have the permission `android.permission.SYSTEM_ALERT_WINDOW` enabled for the development build. You can check this with `Settings.canDrawOverlays(this);`. This is required in dev builds because React Native development errors must be displayed above all the other windows. Due to the new permissions system introduced in the API level 23 (Android M), the user needs to approve it. This can be achieved by adding the following code to your Activity's in `onCreate()` method.

```java
private final int OVERLAY_PERMISSION_REQ_CODE = 1;  // Choose any value

...

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    if (!Settings.canDrawOverlays(this)) {
        Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
                                   Uri.parse("package:" + getPackageName()));
        startActivityForResult(intent, OVERLAY_PERMISSION_REQ_CODE);
    }
}
```

Finally, the `onActivityResult()` method (as shown in the code below) has to be overridden to handle the permission Accepted or Denied cases for consistent UX. Also, for integrating Native Modules which use `startActivityForResult`, we need to pass the result to the `onActivityResult` method of our `ReactInstanceManager` instance.

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == OVERLAY_PERMISSION_REQ_CODE) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (!Settings.canDrawOverlays(this)) {
                // SYSTEM_ALERT_WINDOW permission not granted
            }
        }
    }
    mReactInstanceManager.onActivityResult( this, requestCode, resultCode, data );
}
```

#### 魔法：`ReactRootView`

讓我們加入一些原生程式碼以啟動 React Native 運行時，並告訴它渲染我們的 JS 元件。為此，我們將建立一個 `Activity`，該 Activity 會建立一個 `ReactRootView`，在其中啟動 React 應用程式，並將其設為主要內容視圖。

> If you are targeting Android version &lt;5, use the `AppCompatActivity` class from the `com.android.support:appcompat` package instead of `Activity`.

```java
public class MyReactActivity extends Activity implements DefaultHardwareBackBtnHandler {
    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        SoLoader.init(this, false);

        mReactRootView = new ReactRootView(this);
        List<ReactPackage> packages = new PackageList(getApplication()).getPackages();
        // Packages that cannot be autolinked yet can be added manually here, for example:
        // packages.add(new MyReactNativePackage());
        // Remember to include them in `settings.gradle` and `app/build.gradle` too.

        mReactInstanceManager = ReactInstanceManager.builder()
                .setApplication(getApplication())
                .setCurrentActivity(this)
                .setBundleAssetName("index.android.bundle")
                .setJSMainModulePath("index")
                .addPackages(packages)
                .setUseDeveloperSupport(BuildConfig.DEBUG)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();
        // The string here (e.g. "MyReactNativeApp") has to match
        // the string in AppRegistry.registerComponent() in index.js
        mReactRootView.startReactApplication(mReactInstanceManager, "MyReactNativeApp", null);

        setContentView(mReactRootView);
    }

    @Override
    public void invokeDefaultOnBackPressed() {
        super.onBackPressed();
    }
}
```

> 若您使用 React Native 入門套件，請將 "HelloWorld" 字串替換為您 index.js 檔案中的字串（這是 `AppRegistry.registerComponent()` 方法的第一個參數）。

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

接著，我們需要將部分活動生命週期回調傳遞給 `ReactInstanceManager` 和 `ReactRootView`：

```java
@Override
protected void onPause() {
    super.onPause();

    if (mReactInstanceManager != null) {
        mReactInstanceManager.onHostPause(this);
    }
}

@Override
protected void onResume() {
    super.onResume();

    if (mReactInstanceManager != null) {
        mReactInstanceManager.onHostResume(this, this);
    }
}

@Override
protected void onDestroy() {
    super.onDestroy();

    if (mReactInstanceManager != null) {
        mReactInstanceManager.onHostDestroy(this);
    }
    if (mReactRootView != null) {
        mReactRootView.unmountReactApplication();
    }
}
```

我們還需將返回按鈕事件傳遞給 React Native：

```java
@Override
 public void onBackPressed() {
    if (mReactInstanceManager != null) {
        mReactInstanceManager.onBackPressed();
    } else {
        super.onBackPressed();
    }
}
```

這允許 JavaScript 控制使用者按下硬體返回按鈕時的行為（例如實現導航）。若 JavaScript 未處理返回按鈕事件，則會呼叫您的 `invokeDefaultOnBackPressed` 方法。預設情況下，這將結束您的 `Activity`。

最後，我們需要連接開發者選單。預設情況下，這需搖動裝置觸發，但在模擬器中並不實用。因此我們改為按下硬體選單按鈕時顯示（若使用 Android Studio 模擬器，請按 `Ctrl + M`）：

```java
@Override
public boolean onKeyUp(int keyCode, KeyEvent event) {
    if (keyCode == KeyEvent.KEYCODE_MENU && mReactInstanceManager != null) {
        mReactInstanceManager.showDevOptionsDialog();
        return true;
    }
    return super.onKeyUp(keyCode, event);
}
```

現在您的活動已準備好執行 JavaScript 程式碼。

### 測試整合結果

您已完成整合 React Native 與現有應用程式的基本步驟。現在我們將啟動 [Metro 打包工具][metro] 來建置 `index.bundle` 套件，並啟動本地伺服器來提供該套件。

##### 1. 執行打包伺服器

要運行您的應用程式，需先啟動開發伺服器。請在 React Native 專案的根目錄執行以下指令：

```shell
$ yarn start
```

##### 2. 執行應用程式

現在如常建置並運行您的 Android 應用程式。

當您進入應用內由 React 驅動的活動時，它應從開發伺服器載入 JavaScript 程式碼並顯示：

![螢幕截圖](/docs/assets/EmbeddedAppAndroid.png)

### 在 Android Studio 建立正式版建置

您也可使用 Android Studio 建立正式版建置！這與為既有原生 Android 應用建立正式版的流程同樣快速。

若您按照前述方式使用 React Native Gradle 插件，從 Android Studio 運行應用時一切應能正常運作。

若未使用 React Native Gradle 插件，則每次正式版建置前需額外執行以下步驟，以建立將包含在原生 Android 應用中的 React Native 套件：

```shell
$ npx react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/com/your-company-name/app-package-name/src/main/assets/index.android.bundle --assets-dest android/com/your-company-name/app-package-name/src/main/res/
```

> 請務必替換為正確路徑，若 assets 資料夾不存在則需建立。

現在，如常透過 Android Studio 建立原生應用的正式版建置，即可完成部署！

### 後續步驟

至此，您可繼續如常開發應用程式。請參閱我們的[除錯指南](debugging)與[裝置部署文件](running-on-device)，了解更多 React Native 開發相關資訊。

[metro]: https://metrobundler.dev/