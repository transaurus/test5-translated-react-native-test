---
id: integration-with-android-fragment
title: Integration with an Android Fragment
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

The guide for [Integration with Existing Apps](https://reactnative.dev/docs/integration-with-existing-apps) details how to integrate a full-screen React Native app into an existing Android app as an **Activity**.

若要在現有應用中的**Fragments**內使用React Native元件，則需要進行一些額外的設定。

### 1. 將React Native加入您的應用

按照[與現有應用整合](https://reactnative.dev/docs/integration-with-existing-apps)指南操作，直到最後一步，確保您能安全地在全螢幕Activity中運行React Native應用。

### 2. 為React Native Fragment添加FrameLayout

在此範例中，我們將使用`FrameLayout`將React Native Fragment添加到Activity中。這種方法足夠靈活，可以適應在其他佈局中使用React Native，例如Bottom Sheets或Tab Layouts。

First add a `<FrameLayout>` with an id, width and height to your Activity's layout (e.g. `main_activity.xml` in the `res/layouts` folder). This is the layout you will find to render your React Native Fragment.

```xml
<FrameLayout
    android:id="@+id/react_native_fragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

### 3. 讓您的主Activity實現`DefaultHardwareBackBtnHandler`

由於您的主Activity不是`ReactActivity`，您需要實現`DefaultHardwareBackBtnHandler`接口來處理返回按鈕的按下事件。
這是React Native處理返回按鈕按下事件所必需的。

進入您的主Activity並確保它實現了`DefaultHardwareBackBtnHandler`接口：

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

### 4. 將React Native Fragment添加到FrameLayout中

最後，我們可以更新Activity以將React Native Fragment添加到FrameLayout中。
在此特定範例中，我們假設您的Activity有一個id為`sample_button`的按鈕，點擊後將在FrameLayout中渲染一個React Native Fragment。

更新您的Activity的`onCreate`方法如下：

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

在構建器中，您可以自定義Fragment的創建方式：

- `setComponentName`是您想要渲染的元件名稱。它與您在`index.js`中`registerComponent`方法指定的字符串相同。
- `setLaunchOptions`是一個可選方法，用於將初始props傳遞給您的元件。這是可選的，如果不使用可以移除。

### 5. 測試您的整合

確保您運行`yarn start`來啟動打包器，然後在Android Studio中運行您的Android應用。應用應從開發伺服器加載JavaScript/TypeScript代碼，並在Activity中的React Native Fragment中顯示。

您的應用應如下所示：

![截圖](/docs/assets/EmbeddedAppAndroidFragmentVideo.gif)