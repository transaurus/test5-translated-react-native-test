---
id: integration-with-android-fragment
title: Integration with an Android Fragment
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

The guide for [Integration with Existing Apps](https://reactnative.dev/docs/integration-with-existing-apps) details how to integrate a full-screen React Native app into an existing Android app as an Activity. To use React Native components within Fragments in an existing app requires some additional setup. The benefit of this is that it allows for a native app to integrate React Native components alongside native fragments in an Activity.

### 1. 將React Native加入您的應用

按照[與現有應用整合](https://reactnative.dev/docs/integration-with-existing-apps)指南操作，直到「程式碼整合」章節。繼續完成該章節的步驟1：建立`index.android.js`檔案，以及步驟2：加入您的React Native程式碼。

### 2. 將您的應用與React Native Fragment整合

您可以將React Native元件渲染到Fragment中，而非全螢幕的React Native Activity。此元件可稱為「畫面」或「Fragment」，其運作方式與Android Fragment相同，通常包含子元件。這些元件可放置在`/fragments`資料夾中，而用於組成Fragment的子元件則可放在`/components`資料夾中。

您需要在主要的Application Java/Kotlin類別中實作`ReactApplication`介面。如果是透過Android Studio建立的新專案且使用預設Activity，則需建立新類別（例如`MyReactApplication.java`或`MyReactApplication.kt`）。若是現有類別，您可以在`AndroidManifest.xml`檔案中找到此主類別。在`<application />`標籤下，您會看到`android:name`屬性，例如`android:name=".MyReactApplication"`。此值即為您需實作並提供必要方法的類別。

請確認您的主要Application類別已實作ReactApplication：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="kotlin">

```kotlin
class MyReactApplication: Application(), ReactApplication {...}
```

</TabItem>
<TabItem value="java">

```java
public class MyReactApplication extends Application implements ReactApplication {...}
```

</TabItem>
</Tabs>

覆寫必要方法`getUseDeveloperSupport`、`getPackages`和`getReactNativeHost`：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="kotlin">

```kotlin
class MyReactApplication : Application(), ReactApplication {
    override fun onCreate() {
        super.onCreate()
        SoLoader.init(this, false)
    }
    private val reactNativeHost =
        object : DefaultReactNativeHost(this) {
            override fun getUseDeveloperSupport() = BuildConfig.DEBUG
            override fun getPackages(): List<ReactPackage> {
                val packages = PackageList(this).getPackages().toMutableList()
                // Packages that cannot be autolinked yet can be added manually here
                return packages
            }
        }
    override fun getReactNativeHost(): ReactNativeHost = reactNativeHost
}
```

</TabItem>
<TabItem value="java">

```java
public class MyReactApplication extends Application implements ReactApplication {
    @Override
    public void onCreate() {
        super.onCreate();
        SoLoader.init(this, false);
    }

    private final ReactNativeHost mReactNativeHost = new DefaultReactNativeHost(this) {
        @Override
        public boolean getUseDeveloperSupport() {
            return BuildConfig.DEBUG;
        }

        protected List<ReactPackage> getPackages() {
            List<ReactPackage> packages = new PackageList(this).getPackages();
            // Packages that cannot be autolinked yet can be added manually here
            return packages;
        }
    };

    @Override
    public ReactNativeHost getReactNativeHost() {
        return mReactNativeHost;
    }
}
```

</TabItem>
</Tabs>

若使用Android Studio，請按Alt + Enter為類別加入所有缺少的import。或手動加入以下必要import：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="kotlin">

```kotlin
import android.app.Application

import com.facebook.react.PackageList
import com.facebook.react.ReactApplication
import com.facebook.react.ReactNativeHost
import com.facebook.react.ReactPackage
import com.facebook.react.defaults.DefaultReactNativeHost
import com.facebook.soloader.SoLoader
```

</TabItem>
<TabItem value="java">

```java
import android.app.Application;

import com.facebook.react.PackageList;
import com.facebook.react.ReactApplication;
import com.facebook.react.ReactNativeHost;
import com.facebook.react.ReactPackage;
import com.facebook.react.defaults.DefaultReactNativeHost;
import com.facebook.soloader.SoLoader;

import java.util.List;
```

</TabItem>
</Tabs>

執行「Sync Project files with Gradle」操作。

### 步驟3. 為React Native Fragment加入FrameLayout

現在您需將React Native Fragment加入Activity中。對新專案而言，此Activity為`MainActivity`，但也可以是任何Activity。隨著您將更多React Native元件整合到應用中，可將更多Fragment加入其他Activity。

First add the React Native Fragment to your Activity's layout. For example `main_activity.xml` in the `res/layouts` folder.

加入一個具有id、寬度和高度的`<FrameLayout>`。此佈局將用於尋找並渲染您的React Native Fragment。

```xml
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/reactNativeFragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

### 步驟4. 將React Native Fragment加入FrameLayout

要將React Native Fragment加入佈局中，您需要有一個Activity。如前所述，新專案中此Activity為`MainActivity`。在此Activity中加入按鈕和事件監聽器。點擊按鈕時將渲染您的React Native Fragment。

修改Activity佈局以加入按鈕：

```xml
<Button
    android:layout_margin="10dp"
    android:id="@+id/button"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="Show react fragment" />
```

現在在您的Activity類別（例如`MainActivity.java`或`MainActivity.kt`）中，需為按鈕加入`OnClickListener`，實例化`ReactFragment`並將其加入Frame Layout。

在Activity頂部加入按鈕欄位：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="kotlin">

```kotlin
private lateinit var button: Button
```

</TabItem>
<TabItem value="java">

```java
private Button mButton;
```

</TabItem>
</Tabs>

更新Activity的`onCreate`方法如下：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="kotlin">

```kotlin
override fun onCreate(savedInstanceState: Bundle) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.main_activity)
    button = findViewById<Button>(R.id.button)
    button.setOnClickListener {
        val reactNativeFragment = ReactFragment.Builder()
                .setComponentName("HelloWorld")
                .setLaunchOptions(getLaunchOptions("test message"))
                .build()
        getSupportFragmentManager()
                .beginTransaction()
                .add(R.id.reactNativeFragment, reactNativeFragment)
                .commit()
    }
}
```

</TabItem>
<TabItem value="java">

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main_activity);

    mButton = findViewById(R.id.button);
    mButton.setOnClickListener(new View.OnClickListener() {
        public void onClick(View v) {
            Fragment reactNativeFragment = new ReactFragment.Builder()
                    .setComponentName("HelloWorld")
                    .setLaunchOptions(getLaunchOptions("test message"))
                    .build();

            getSupportFragmentManager()
                    .beginTransaction()
                    .add(R.id.reactNativeFragment, reactNativeFragment)
                    .commit();

        }
    });
}
```

</TabItem>
</Tabs>

在上述程式碼中，`Fragment reactNativeFragment = new ReactFragment.Builder()` 會建立 ReactFragment，而 `getSupportFragmentManager().beginTransaction().add()` 則會將該 Fragment 加入至 Frame Layout 中。

若您使用的是 React Native 的入門套件，請將 "HelloWorld" 字串替換為您 `index.js` 或 `index.android.js` 檔案中的字串（即 AppRegistry.registerComponent() 方法的第一個參數）。

新增 `getLaunchOptions` 方法，該方法可讓您將 props 傳遞至您的元件。此為選用功能，若您不需要傳遞任何 props，可移除 `setLaunchOptions`。

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="kotlin">

```kotlin
private fun getLaunchOptions(message: String) = Bundle().apply {
    putString("message", message)
}

```

</TabItem>
<TabItem value="java">

```java
private Bundle getLaunchOptions(String message) {
    Bundle initialProperties = new Bundle();
    initialProperties.putString("message", message);
    return initialProperties;
}
```

</TabItem>
</Tabs>

在您的 Activity 類別中新增所有缺少的 imports。請注意使用您套件的 BuildConfig，而非來自 facebook 套件的 BuildConfig！或者，以下是需要手動加入的必要 imports：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="kotlin">

```kotlin
import android.app.Application

import com.facebook.react.ReactApplication
import com.facebook.react.ReactNativeHost
import com.facebook.react.ReactPackage
import com.facebook.react.shell.MainReactPackage
import com.facebook.soloader.SoLoader

```

</TabItem>
<TabItem value="java">

```java
import android.app.Application;

import com.facebook.react.ReactApplication;
import com.facebook.react.ReactNativeHost;
import com.facebook.react.ReactPackage;
import com.facebook.react.shell.MainReactPackage;
import com.facebook.soloader.SoLoader;
```

</TabItem>
</Tabs>

執行「Sync Project files with Gradle」操作。

### 步驟 5. 測試您的整合

請確保執行 `yarn` 以安裝您的 react-native 相依套件，並執行 `yarn native` 以啟動 metro bundler。在 Android Studio 中執行您的 Android 應用程式，它應會從開發伺服器載入 JavaScript 程式碼，並在 Activity 中的 React Native Fragment 顯示該程式碼。

### 步驟 6. 額外設定 - 原生模組

您可能需要從您的 react 元件呼叫現有的 Java/Kotlin 程式碼。原生模組可讓您呼叫原生程式碼，並在您的原生應用程式中執行方法。請遵循此處的設定 [native-modules-android](/docs/0.74/native-modules-android)