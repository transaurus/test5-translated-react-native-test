---
id: integration-with-android-fragment
title: Integration with an Android Fragment
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

The guide for [Integration with Existing Apps](https://reactnative.dev/docs/integration-with-existing-apps) details how to integrate a full-screen React Native app into an existing Android app as an **Activity**.

若要在現有應用中的**Fragments**內使用React Native元件，則需要進行一些額外的設定。

### 1. 將React Native加入你的應用

按照[與現有應用整合](https://reactnative.dev/docs/integration-with-existing-apps)指南操作至最後，確保你能安全地在全螢幕Activity中運行你的React Native應用。

### 2. 為React Native Fragment添加FrameLayout

在這個範例中，我們將使用`FrameLayout`來將React Native Fragment添加到Activity中。這種方法足夠靈活，可以適應在其他佈局中使用React Native，例如Bottom Sheets或Tab Layouts。

First add a `<FrameLayout>` with an id, width and height to your Activity's layout (e.g. `main_activity.xml` in the `res/layouts` folder). This is the layout you will find to render your React Native Fragment.

```xml
<FrameLayout
    android:id="@+id/react_native_fragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

### 3. 讓你的宿主Activity實現`DefaultHardwareBackBtnHandler`

由於你的宿主activity不是`ReactActivity`，你需要實現`DefaultHardwareBackBtnHandler`接口來處理返回按鈕的點擊事件。這是React Native用來處理返回按鈕點擊事件所必需的。

進入你的宿主activity並確保它實現了`DefaultHardwareBackBtnHandler`接口：

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

### 4. 將React Native Fragment添加到FrameLayout

最後，我們可以更新Activity以將React Native Fragment添加到FrameLayout中。在這個具體的範例中，我們假設你的Activity有一個id為`sample_button`的按鈕，點擊後會將React Native Fragment渲染到FrameLayout中。

按如下方式更新你的Activity的`onCreate`方法：

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

讓我們看看上面的代碼。

The `ReactFragment.Builder()` is used to create a new `ReactFragment` and then we use the `supportFragmentManager` to add that Fragment to the `FrameLayout`.

在builder內部，你可以自定義Fragment的創建方式：

- `setComponentName`是你想要渲染的元件名稱。它與你在`index.js`中`registerComponent`方法指定的字符串相同。
- `setLaunchOptions`是一個可選方法，用於向你的元件傳遞初始props。這是可選的，如果你不需要使用可以移除它。

### 5. 測試你的整合

確保你運行`yarn start`來啟動bundler，然後在Android Studio中運行你的Android應用。應用應該會從開發伺服器加載JavaScript/TypeScript代碼，並在你的Activity中的React Native Fragment中顯示它。

你的應用應該看起來像這樣：

![截圖](/docs/assets/EmbeddedAppAndroidFragmentVideo.gif)