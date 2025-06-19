---
id: integration-with-android-fragment
title: Integration with an Android Fragment
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

The guide for [Integration with Existing Apps](https://reactnative.dev/docs/integration-with-existing-apps) details how to integrate a full-screen React Native app into an existing Android app as an **Activity**.

若要在現有應用中的 **Fragments** 內使用 React Native 元件，則需要進行一些額外的設定。

### 1. 將 React Native 加入你的應用

按照[與現有應用整合](https://reactnative.dev/docs/integration-with-existing-apps)指南操作，直到最後一步，確保你能安全地在全螢幕 Activity 中運行你的 React Native 應用。

### 2. 為 React Native Fragment 添加 FrameLayout

在這個範例中，我們將使用 `FrameLayout` 來將 React Native Fragment 添加到 Activity 中。這種方法足夠靈活，可以適應在其他佈局（如 Bottom Sheets 或 Tab Layouts）中使用 React Native。

First add a `<FrameLayout>` with an id, width and height to your Activity's layout (e.g. `main_activity.xml` in the `res/layouts` folder). This is the layout you will find to render your React Native Fragment.

```xml
<FrameLayout
    android:id="@+id/react_native_fragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

### 3. 讓你的宿主 Activity 實現 `DefaultHardwareBackBtnHandler`

由於你的宿主 Activity 不是 `ReactActivity`，因此需要實現 `DefaultHardwareBackBtnHandler` 接口來處理返回按鈕的點擊事件。這是 React Native 處理返回按鈕點擊事件所必需的。

進入你的宿主 Activity，確保它實現了 `DefaultHardwareBackBtnHandler` 接口：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="kotlin">

```diff
package <your-package-here>

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
+import com.facebook.react.modules.core.DefaultHardwareBackBtnHandler

+class MainActivity : AppCompatActivity() {
+class MainActivity : AppCompatActivity(), DefaultHardwareBackBtnHandler {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.main_activity)

        findViewById<Button>(R.id.sample_button).setOnClickListener {
            // Handle button click
        }
    }

+   override fun invokeDefaultOnBackPressed() {
+       super.onBackPressed()
+   }
}
```

</TabItem>
<TabItem value="java">

```diff
package <your-package-here>;

import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;
+import com.facebook.react.modules.core.DefaultHardwareBackBtnHandler;

-class MainActivity extends AppCompatActivity {
+class MainActivity extends AppCompatActivity implements DefaultHardwareBackBtnHandler {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main_activity);

        findViewById(R.id.button_appcompose).setOnClickListener(button -> {
            // Handle button click
        });
    }

+   @Override
+   public void invokeDefaultOnBackPressed() {
+       super.onBackPressed();
+   }
}
```

</TabItem>
</Tabs>

### 4. 將 React Native Fragment 添加到 FrameLayout

最後，我們可以更新 Activity，將 React Native Fragment 添加到 FrameLayout 中。在這個具體範例中，我們假設你的 Activity 有一個 id 為 `sample_button` 的按鈕，點擊後會將 React Native Fragment 渲染到 FrameLayout 中。

按如下方式更新你的 Activity 的 `onCreate` 方法：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="kotlin">

```diff
package <your-package-here>

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
+import com.facebook.react.ReactFragment
import com.facebook.react.modules.core.DefaultHardwareBackBtnHandler

public class MainActivity : AppCompatActivity(), DefaultHardwareBackBtnHandler {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.main_activity)

        findViewById<Button>(R.id.sample_button).setOnClickListener {
+           val reactNativeFragment = ReactFragment.Builder()
+               .setComponentName("HelloWorld")
+               .setLaunchOptions(Bundle().apply { putString("message", "my value") })
+               .build()
+           supportFragmentManager
+               .beginTransaction()
+               .add(R.id.react_native_fragment, reactNativeFragment)
+               .commit()
        }
    }

   override fun invokeDefaultOnBackPressed() {
       super.onBackPressed()
   }
}
```

</TabItem>
<TabItem value="java">

```diff
package <your-package-here>;

import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;
+import com.facebook.react.ReactFragment;
import com.facebook.react.modules.core.DefaultHardwareBackBtnHandler;

public class MainActivity extends AppCompatActivity implements DefaultHardwareBackBtnHandler {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main_activity);

        findViewById(R.id.button_appcompose).setOnClickListener(button -> {
+           Bundle launchOptions = new Bundle();
+           launchOptions.putString("message", "my value");
+
+           ReactFragment fragment = new ReactFragment.Builder()
+                   .setComponentName("HelloWorld")
+                   .setLaunchOptions(launchOptions)
+                   .build();
+           getSupportFragmentManager()
+                   .beginTransaction()
+                   .add(R.id.react_native_fragment, fragment)
+                   .commit();
        });
    }

    @Override
    public void invokeDefaultOnBackPressed() {
        super.onBackPressed();
    }
}
```

</TabItem>
</Tabs>

讓我們來看看上面的代碼。

The `ReactFragment.Builder()` is used to create a new `ReactFragment` and then we use the `supportFragmentManager` to add that Fragment to the `FrameLayout`.

在 builder 中，你可以自定義 Fragment 的創建方式：

- `setComponentName` 是你想要渲染的元件名稱。它與你在 `index.js` 中 `registerComponent` 方法指定的字符串相同。
- `setLaunchOptions` 是一個可選方法，用於將初始 props 傳遞給你的元件。這是可選的，如果不使用可以移除。

### 5. 測試你的整合

確保你運行 `yarn start` 來啟動打包器，然後在 Android Studio 中運行你的 Android 應用。應用應該會從開發伺服器加載 JavaScript/TypeScript 代碼，並在你的 Activity 中的 React Native Fragment 中顯示。

你的應用應該看起來像這樣：

![截圖](/docs/assets/EmbeddedAppAndroidFragmentVideo.gif)