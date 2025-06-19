---
id: native-modules-android
title: Android Native Modules
---

import NativeDeprecated from '@site/versioned_docs/version-0.76/the-new-architecture/\_markdown_native_deprecation.mdx'
import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<NativeDeprecated />

歡迎使用 Android 原生模組。請先閱讀 [原生模組簡介](native-modules-intro) 了解原生模組的基本概念。

## 建立日曆原生模組

In the following guide you will create a native module, `CalendarModule`, that will allow you to access Android’s calendar APIs from JavaScript. By the end, you will be able to call `CalendarModule.createCalendarEvent('Dinner Party', 'My House');` from JavaScript, invoking a Java/Kotlin method that creates a calendar event.

### 環境設定

首先，請在 Android Studio 中開啟您的 React Native 應用程式內的 Android 專案。您可以在以下路徑找到 Android 專案：

<figure>
  <img src="/docs/assets/native-modules-android-open-project.png" width="500" alt="Image of opening up an Android project within a React Native app inside of Android Studio." />
  <figcaption>Image of where you can find your Android project</figcaption>
</figure>

We recommend using Android Studio to write your native code. Android studio is an IDE built for Android development and using it will help you resolve minor issues like code syntax errors quickly.

同時建議啟用 [Gradle Daemon](https://docs.gradle.org/2.9/userguide/gradle_daemon.html) 以加速 Java/Kotlin 程式碼的迭代建構。

### 建立自訂原生模組檔案

The first step is to create the (`CalendarModule.java` or `CalendarModule.kt`) Java/Kotlin file inside `android/app/src/main/java/com/your-app-name/` folder (the folder is the same for both Kotlin and Java). This Java/Kotlin file will contain your native module Java/Kotlin class.

<figure>
  <img src="/docs/assets/native-modules-android-add-class.png" width="700" alt="Image of adding a class called CalendarModule.java within the Android Studio." />
  <figcaption>Image of how to add the CalendarModuleClass</figcaption>
</figure>

接著加入以下內容：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
package com.your-apps-package-name; // replace your-apps-package-name with your app’s package name
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import java.util.Map;
import java.util.HashMap;

public class CalendarModule extends ReactContextBaseJavaModule {
   CalendarModule(ReactApplicationContext context) {
       super(context);
   }
}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
package com.your-apps-package-name; // replace your-apps-package-name with your app’s package name
import com.facebook.react.bridge.NativeModule
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.bridge.ReactContext
import com.facebook.react.bridge.ReactContextBaseJavaModule
import com.facebook.react.bridge.ReactMethod

class CalendarModule(reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext) {...}
```

</TabItem>
</Tabs>

如您所見，您的 `CalendarModule` 類別繼承了 `ReactContextBaseJavaModule`。在 Android 中，Java/Kotlin 原生模組需擴展 `ReactContextBaseJavaModule` 並實作 JavaScript 所需的功能。

> 需注意的是，技術上 Java/Kotlin 類別只需繼承 `BaseJavaModule` 或實作 `NativeModule` 介面，即可被 React Native 視為原生模組。

> However we recommend that you use `ReactContextBaseJavaModule`, as shown above. `ReactContextBaseJavaModule` gives access to the `ReactApplicationContext` (RAC), which is useful for Native Modules that need to hook into activity lifecycle methods. Using `ReactContextBaseJavaModule` will also make it easier to make your native module type-safe in the future. For native module type-safety, which is coming in future releases, React Native looks at each native module's JavaScript spec and generates an abstract base class that extends `ReactContextBaseJavaModule`.

### 模組名稱

所有 Android 的 Java/Kotlin 原生模組都需實作 `getName()` 方法。該方法返回的字串代表原生模組的名稱，JavaScript 可透過此名稱存取模組。例如下方程式碼片段中，`getName()` 返回 `"CalendarModule"`。

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
// add to CalendarModule.java
@Override
public String getName() {
   return "CalendarModule";
}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
// add to CalendarModule.kt
override fun getName() = "CalendarModule"
```

</TabItem>
</Tabs>

接著 JavaScript 便可如此存取該原生模組：

```tsx
const {CalendarModule} = ReactNative.NativeModules;
```

### 將原生方法導出至 JavaScript

接下來您需在原生模組中添加一個能建立日曆事件並可從 JavaScript 呼叫的方法。所有需從 JavaScript 呼叫的原生模組方法都必須標記 `@ReactMethod` 註解。

Set up a method `createCalendarEvent()` for `CalendarModule` that can be invoked in JS through `CalendarModule.createCalendarEvent()`. For now, the method will take in a name and location as strings. Argument type options will be covered shortly.

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
@ReactMethod
public void createCalendarEvent(String name, String location) {
}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
@ReactMethod fun createCalendarEvent(name: String, location: String) {}
```

</TabItem>
</Tabs>

在方法中添加除錯日誌，以確認從應用程式呼叫時是否成功觸發。以下是從 Android util 套件導入和使用 [Log](https://developer.android.com/reference/android/util/Log) 類別的範例：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
import android.util.Log;

@ReactMethod
public void createCalendarEvent(String name, String location) {
   Log.d("CalendarModule", "Create event called with name: " + name
   + " and location: " + location);
}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
import android.util.Log

@ReactMethod
fun createCalendarEvent(name: String, location: String) {
    Log.d("CalendarModule", "Create event called with name: $name and location: $location")
}
```

</TabItem>
</Tabs>

完成原生模組的實作並在 JavaScript 中掛接後，您可以按照[這些步驟](https://developer.android.com/studio/debug/am-logcat.html)查看應用程式的日誌。

### 同步方法

您可以將 `isBlockingSynchronousMethod = true` 傳遞給原生方法，將其標記為同步方法。

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
@ReactMethod(isBlockingSynchronousMethod = true)
```

</TabItem>
<TabItem value="kotlin">

```kotlin
@ReactMethod(isBlockingSynchronousMethod = true)
```

</TabItem>
</Tabs>

目前我們不建議這樣做，因為同步呼叫方法可能會導致嚴重的效能損失，並在您的原生模組中引入與線程相關的錯誤。此外，請注意，如果您選擇啟用 `isBlockingSynchronousMethod`，您的應用程式將無法再使用 Google Chrome 除錯工具。這是因為同步方法要求 JS VM 與應用程式共享記憶體。對於 Google Chrome 除錯工具，React Native 在 Google Chrome 的 JS VM 中運行，並通過 WebSockets 與移動設備進行非同步通訊。

### 註冊模組 (Android 專屬)

編寫完原生模組後，需要將其註冊到 React Native 中。為此，您需要將原生模組添加到 `ReactPackage` 中，並將 `ReactPackage` 註冊到 React Native。在初始化期間，React Native 會遍歷所有套件，並為每個 `ReactPackage` 註冊其中的每個原生模組。

React Native invokes the method `createNativeModules()` on a `ReactPackage` in order to get the list of native modules to register. For Android, if a module is not instantiated and returned in createNativeModules it will not be available from JavaScript.

To add your Native Module to `ReactPackage`, first create a new Java/Kotlin Class named (`MyAppPackage.java` or `MyAppPackage.kt`) that implements `ReactPackage` inside the `android/app/src/main/java/com/your-app-name/` folder:

然後添加以下內容：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
package com.your-app-name; // replace your-app-name with your app’s name
import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class MyAppPackage implements ReactPackage {

   @Override
   public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
       return Collections.emptyList();
   }

   @Override
   public List<NativeModule> createNativeModules(
           ReactApplicationContext reactContext) {
       List<NativeModule> modules = new ArrayList<>();

       modules.add(new CalendarModule(reactContext));

       return modules;
   }

}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
package com.your-app-name // replace your-app-name with your app’s name

import android.view.View
import com.facebook.react.ReactPackage
import com.facebook.react.bridge.NativeModule
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.uimanager.ReactShadowNode
import com.facebook.react.uimanager.ViewManager

class MyAppPackage : ReactPackage {

    override fun createViewManagers(
        reactContext: ReactApplicationContext
    ): MutableList<ViewManager<View, ReactShadowNode<*>>> = mutableListOf()

    override fun createNativeModules(
        reactContext: ReactApplicationContext
    ): MutableList<NativeModule> = listOf(CalendarModule(reactContext)).toMutableList()
}
```

</TabItem>
</Tabs>

This file imports the native module you created, `CalendarModule`. It then instantiates `CalendarModule` within the `createNativeModules()` function and returns it as a list of `NativeModules` to register. If you add more native modules down the line, you can also instantiate them and add them to the list returned here.

> 值得注意的是，這種註冊原生模組的方式會在應用程式啟動時急切初始化所有原生模組，這會增加應用程式的啟動時間。您可以使用 [TurboReactPackage](https://github.com/facebook/react-native/blob/main/packages/react-native/ReactAndroid/src/main/java/com/facebook/react/TurboReactPackage.java) 作為替代方案。與 `createNativeModules`（返回實例化的原生模組對象列表）不同，TurboReactPackage 實現了一個 `getModule(String name, ReactApplicationContext rac)` 方法，該方法在需要時創建原生模組對象。目前 TurboReactPackage 的實現稍微複雜一些。除了實現 `getModule()` 方法外，您還需要實現一個 `getReactModuleInfoProvider()` 方法，該方法返回套件可以實例化的所有原生模組的列表以及實例化它們的函數，示例[在此](https://github.com/facebook/react-native/blob/8ac467c51b94c82d81930b4802b2978c85539925/ReactAndroid/src/main/java/com/facebook/react/CoreModulesPackage.java#L86-L165)。再次強調，使用 TurboReactPackage 可以讓您的應用程式擁有更快的啟動時間，但目前編寫起來有點麻煩。因此，如果您選擇使用 TurboReactPackages，請謹慎行事。

要註冊 `CalendarModule` 套件，您必須將 `MyAppPackage` 添加到 ReactNativeHost 的 `getPackages()` 方法返回的套件列表中。打開您的 `MainApplication.java` 或 `MainApplication.kt` 文件，該文件可以在以下路徑中找到：`android/app/src/main/java/com/your-app-name/`。

找到 ReactNativeHost 的 `getPackages()` 方法，並將您的套件添加到 `getPackages()` 返回的套件列表中：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
@Override
protected List<ReactPackage> getPackages() {
    List<ReactPackage> packages = new PackageList(this).getPackages();
    // Packages that cannot be autolinked yet can be added manually here, for example:
    // packages.add(new MyReactNativePackage());
    packages.add(new MyAppPackage());
    return packages;
}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
override fun getPackages(): List<ReactPackage> =
    PackageList(this).packages.apply {
        // Packages that cannot be autolinked yet can be added manually here, for example:
        // add(MyReactNativePackage())
        add(MyAppPackage())
    }
```

</TabItem>
</Tabs>

您現在已成功為 Android 註冊了您的原生模組！

### 測試您所構建的內容

至此，您已經在 Android 中為原生模組設置了基本框架。請透過在 JavaScript 中存取原生模組並呼叫其匯出的方法來進行測試。

在應用程式中找到您想要新增呼叫原生模組 `createCalendarEvent()` 方法的位置。以下是一個範例元件 `NewModuleButton`，您可以將其新增至應用程式中。您可以在 `NewModuleButton` 的 `onPress()` 函數中呼叫原生模組。

```tsx
import React from 'react';
import {NativeModules, Button} from 'react-native';

const NewModuleButton = () => {
  const onPress = () => {
    console.log('We will invoke the native module here!');
  };

  return (
    <Button
      title="Click to invoke your native module!"
      color="#841584"
      onPress={onPress}
    />
  );
};

export default NewModuleButton;
```

為了從 JavaScript 存取您的原生模組，您需要先從 React Native 導入 `NativeModules`：

```tsx
import {NativeModules} from 'react-native';
```

You can then access the `CalendarModule` native module off of `NativeModules`.

```tsx
const {CalendarModule} = NativeModules;
```

Now that you have the CalendarModule native module available, you can invoke your native method `createCalendarEvent()`. Below it is added to the `onPress()` method in `NewModuleButton`:

```tsx
const onPress = () => {
  CalendarModule.createCalendarEvent('testName', 'testLocation');
};
```

最後一步是重新建置 React Native 應用程式，以便讓最新的原生程式碼（包含您的新原生模組！）可用。在您的命令列中，位於 react native 應用程式的目錄下，執行以下指令：

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

### 迭代開發時的建置

當您遵循這些指南並對原生模組進行迭代開發時，您需要對應用程式進行原生重建，以便從 JavaScript 存取您的最新變更。這是因為您編寫的程式碼位於應用程式的原生部分。雖然 React Native 的 metro bundler 可以監視 JavaScript 的變更並即時重建，但它不會對原生程式碼這樣做。因此，如果您想測試最新的原生變更，您需要使用上述指令進行重建。

### 回顧✨

您現在應該能夠在應用程式中呼叫原生模組的 `createCalendarEvent()` 方法。在我們的範例中，這是透過按下 `NewModuleButton` 來實現的。您可以透過查看您在 `createCalendarEvent()` 方法中設置的日誌來確認這一點。您可以按照[這些步驟](https://developer.android.com/studio/debug/am-logcat.html)來查看應用程式中的 ADB 日誌。然後，您應該能夠搜尋您的 `Log.d` 訊息（在我們的範例中為「Create event called with name: testName and location: testLocation」），並在每次呼叫原生模組方法時看到您的日誌訊息。

<figure>
  <img src="/docs/assets/native-modules-android-logs.png" width="1000" alt="Image of logs." />
  <figcaption>Image of ADB logs in Android Studio</figcaption>
</figure>

至此，您已經創建了一個 Android 原生模組，並從 React Native 應用程式的 JavaScript 中呼叫了它的原生方法。您可以繼續閱讀以了解更多關於原生模組方法可用的參數類型以及如何設置回調和 Promise 的資訊。

## 超越日曆原生模組

### 更好的原生模組匯出

像上面那樣從 `NativeModules` 中拉取您的原生模組來導入有點笨拙。

為了讓您的原生模組的使用者不必每次存取原生模組時都這樣做，您可以為模組創建一個 JavaScript 包裝器。創建一個名為 `CalendarModule.js` 的新 JavaScript 文件，內容如下：

```tsx
/**
* This exposes the native CalendarModule module as a JS module. This has a
* function 'createCalendarEvent' which takes the following parameters:

* 1. String name: A string representing the name of the event
* 2. String location: A string representing the location of the event
*/
import {NativeModules} from 'react-native';
const {CalendarModule} = NativeModules;
export default CalendarModule;
```

This JavaScript file also becomes a good location for you to add any JavaScript side functionality. For example, if you use a type system like TypeScript you can add type annotations for your native module here. While React Native does not yet support Native to JS type safety, all your JS code will be type safe. Doing so will also make it easier for you to switch to type-safe native modules down the line. Below is an example of adding type safety to the CalendarModule:

```tsx
/**
 * This exposes the native CalendarModule module as a JS module. This has a
 * function 'createCalendarEvent' which takes the following parameters:
 *
 * 1. String name: A string representing the name of the event
 * 2. String location: A string representing the location of the event
 */
import {NativeModules} from 'react-native';
const {CalendarModule} = NativeModules;
interface CalendarInterface {
  createCalendarEvent(name: string, location: string): void;
}
export default CalendarModule as CalendarInterface;
```

在其他 JavaScript 文件中，您可以像這樣存取原生模組並呼叫其方法：

```tsx
import CalendarModule from './CalendarModule';
CalendarModule.createCalendarEvent('foo', 'bar');
```

> 這假設您導入 `CalendarModule` 的位置與 `CalendarModule.js` 位於相同的目錄層級。請根據需要更新相對導入路徑。

### 參數類型

當在 JavaScript 中調用原生模組方法時，React Native 會將參數從 JS 物件轉換為對應的 Java/Kotlin 物件。舉例來說，如果你的 Java 原生模組方法接受一個 double 類型，在 JS 中你需要用數字來調用該方法。React Native 會自動處理轉換。以下是原生模組方法支持的參數類型及其對應的 JavaScript 等效類型列表。

| Java          | Kotlin        | JavaScript |
| ------------- | ------------- | ---------- |
| Boolean       | Boolean       | ?boolean   |
| boolean       |               | boolean    |
| Double        | Double        | ?number    |
| double        |               | number     |
| String        | String        | string     |
| Callback      | Callback      | Function   |
| Promise       | Promise       | Promise    |
| ReadableMap   | ReadableMap   | Object     |
| ReadableArray | ReadableArray | Array      |

> 以下類型目前支持，但在 TurboModules 中將不再支持，請避免使用：
>
> - Integer Java/Kotlin -> ?number
> - Float Java/Kotlin -> ?number
> - int Java -> number
> - float Java -> number

對於未列出的參數類型，你需要自行處理轉換。例如，在 Android 中，`Date` 類型的轉換並不原生支持。你可以在原生方法中自行處理 `Date` 類型的轉換，如下所示：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
    String dateFormat = "yyyy-MM-dd";
    SimpleDateFormat sdf = new SimpleDateFormat(dateFormat);
    Calendar eStartDate = Calendar.getInstance();
    try {
        eStartDate.setTime(sdf.parse(startDate));
    }

```

</TabItem>
<TabItem value="kotlin">

```kotlin
    val dateFormat = "yyyy-MM-dd"
    val sdf = SimpleDateFormat(dateFormat, Locale.US)
    val eStartDate = Calendar.getInstance()
    try {
        sdf.parse(startDate)?.let {
            eStartDate.time = it
        }
    }
```

</TabItem>
</Tabs>

### 導出常量

原生模組可以通過實現原生方法 `getConstants()` 來導出常量，該方法在 JS 中可用。以下你將實現 `getConstants()` 並返回一個包含 `DEFAULT_EVENT_NAME` 常量的 Map，該常量可以在 JavaScript 中訪問：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
@Override
public Map<String, Object> getConstants() {
   final Map<String, Object> constants = new HashMap<>();
   constants.put("DEFAULT_EVENT_NAME", "New Event");
   return constants;
}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
override fun getConstants(): MutableMap<String, Any> =
    hashMapOf("DEFAULT_EVENT_NAME" to "New Event")
```

</TabItem>
</Tabs>

然後可以通過在 JS 中調用原生模組的 `getConstants` 來訪問該常量：

```tsx
const {DEFAULT_EVENT_NAME} = CalendarModule.getConstants();
console.log(DEFAULT_EVENT_NAME);
```

從技術上講，可以直接從原生模組物件訪問 `getConstants()` 導出的常量。這在 TurboModules 中將不再支持，因此我們鼓勵社區轉向上述方法，以避免未來不必要的遷移。

> That currently constants are exported only at initialization time, so if you change getConstants values at runtime it won't affect the JavaScript environment. This will change with Turbomodules. With Turbomodules, `getConstants()` will become a regular native module method, and each invocation will hit the native side.

### 回調

原生模組還支持一種特殊的參數：回調。回調用於將數據從 Java/Kotlin 異步傳遞到 JavaScript。它們也可以用於從原生端異步執行 JavaScript。

要創建一個帶有回調的原生模組方法，首先導入 `Callback` 接口，然後在你的原生模組方法中添加一個類型為 `Callback` 的新參數。回調參數有幾個細節，這些細節將在 TurboModules 中得到改進。首先，你的函數參數中只能有兩個回調——一個 successCallback 和一個 failureCallback。此外，原生模組方法調用的最後一個參數（如果是函數）將被視為 successCallback，倒數第二個參數（如果是函數）將被視為 failureCallback。

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
import com.facebook.react.bridge.Callback;

@ReactMethod
public void createCalendarEvent(String name, String location, Callback callBack) {
}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
import com.facebook.react.bridge.Callback

@ReactMethod fun createCalendarEvent(name: String, location: String, callback: Callback) {}
```

</TabItem>
</Tabs>

你可以在你的 Java/Kotlin 方法中調用回調，並提供你想傳遞給 JavaScript 的任何數據。請注意，你只能從原生代碼傳遞可序列化的數據到 JavaScript。如果需要傳遞原生物件，可以使用 `WriteableMaps`；如果需要使用集合，可以使用 `WritableArrays`。還需要強調的是，回調不會在原生函數完成後立即調用。以下將之前調用中創建的事件 ID 傳遞給回調。

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
  @ReactMethod
   public void createCalendarEvent(String name, String location, Callback callBack) {
       Integer eventId = ...
       callBack.invoke(eventId);
   }
```

</TabItem>
<TabItem value="kotlin">

```kotlin
  @ReactMethod
  fun createCalendarEvent(name: String, location: String, callback: Callback) {
      val eventId = ...
      callback.invoke(eventId)
  }
```

</TabItem>
</Tabs>

然後可以在 JavaScript 中使用以下方式訪問該方法：

```tsx
const onPress = () => {
  CalendarModule.createCalendarEvent(
    'Party',
    'My House',
    eventId => {
      console.log(`Created a new event with id ${eventId}`);
    },
  );
};
```

另一個重要的細節是，原生模組方法只能調用一個回調，且只能調用一次。這意味著你可以調用成功回調或失敗回調，但不能同時調用兩者，且每個回調最多只能調用一次。不過，原生模組可以存儲回調並在稍後調用它。

回調的錯誤處理有兩種方法。第一種是遵循 Node 的慣例，將傳遞給回調的第一個參數視為錯誤物件。

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
  @ReactMethod
   public void createCalendarEvent(String name, String location, Callback callBack) {
       Integer eventId = ...
       callBack.invoke(null, eventId);
   }
```

</TabItem>
<TabItem value="kotlin">

```kotlin
  @ReactMethod
  fun createCalendarEvent(name: String, location: String, callback: Callback) {
      val eventId = ...
      callback.invoke(null, eventId)
  }
```

</TabItem>
</Tabs>

在 JavaScript 中，您可以檢查第一個參數來判斷是否有錯誤傳遞：

```tsx
const onPress = () => {
  CalendarModule.createCalendarEvent(
    'testName',
    'testLocation',
    (error, eventId) => {
      if (error) {
        console.error(`Error found! ${error}`);
      }
      console.log(`event id ${eventId} returned`);
    },
  );
};
```

另一個選項是使用 onSuccess 和 onFailure 回調：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
@ReactMethod
public void createCalendarEvent(String name, String location, Callback myFailureCallback, Callback mySuccessCallback) {
}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
@ReactMethod
  fun createCalendarEvent(
      name: String,
      location: String,
      myFailureCallback: Callback,
      mySuccessCallback: Callback
  ) {}
```

</TabItem>
</Tabs>

然後在 JavaScript 中，您可以為錯誤和成功響應分別添加回調：

```tsx
const onPress = () => {
  CalendarModule.createCalendarEvent(
    'testName',
    'testLocation',
    error => {
      console.error(`Error found! ${error}`);
    },
    eventId => {
      console.log(`event id ${eventId} returned`);
    },
  );
};
```

### Promise

原生模組也可以實現 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)，這可以簡化您的 JavaScript 程式碼，尤其是在使用 ES2016 的 [async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) 語法時。當原生模組 Java/Kotlin 方法的最後一個參數是 Promise 時，其對應的 JS 方法將返回一個 JS Promise 物件。

將上述程式碼重構為使用 Promise 而非回調的範例如下：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
import com.facebook.react.bridge.Promise;

@ReactMethod
public void createCalendarEvent(String name, String location, Promise promise) {
    try {
        Integer eventId = ...
        promise.resolve(eventId);
    } catch(Exception e) {
        promise.reject("Create Event Error", e);
    }
}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
import com.facebook.react.bridge.Promise

@ReactMethod
fun createCalendarEvent(name: String, location: String, promise: Promise) {
    try {
        val eventId = ...
        promise.resolve(eventId)
    } catch (e: Throwable) {
        promise.reject("Create Event Error", e)
    }
}
```

</TabItem>
</Tabs>

> 與回調類似，原生模組方法可以拒絕或解決一個 Promise（但不能同時進行），且最多只能執行一次。這意味著您可以調用成功回調或失敗回調，但不能同時調用，且每個回調最多只能調用一次。不過，原生模組可以儲存回調並稍後調用。

此方法的 JavaScript 對應部分會返回一個 Promise。這意味著您可以在 async 函數中使用 `await` 關鍵字來調用它並等待其結果：

```tsx
const onSubmit = async () => {
  try {
    const eventId = await CalendarModule.createCalendarEvent(
      'Party',
      'My House',
    );
    console.log(`Created a new event with id ${eventId}`);
  } catch (e) {
    console.error(e);
  }
};
```

reject 方法接受以下參數的組合：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
String code, String message, WritableMap userInfo, Throwable throwable
```

</TabItem>
<TabItem value="kotlin">

```kotlin
code: String, message: String, userInfo: WritableMap, throwable: Throwable
```

</TabItem>
</Tabs>

For more detail, you can find the `Promise.java` interface [here](https://github.com/facebook/react-native/blob/main/packages/react-native/ReactAndroid/src/main/java/com/facebook/react/bridge/Promise.java). If `userInfo` is not provided, ReactNative will set it to null. For the rest of the parameters React Native will use a default value. The `message` argument provides the error `message` shown at the top of an error call stack. Below is an example of the error message shown in JavaScript from the following reject call in Java/Kotlin.

Java/Kotlin 中的 reject 呼叫：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
promise.reject("Create Event error", "Error parsing date", e);
```

</TabItem>
<TabItem value="kotlin">

```kotlin
promise.reject("Create Event error", "Error parsing date", e)
```

</TabItem>
</Tabs>

當 Promise 被拒絕時，React Native 應用中顯示的錯誤訊息：

<figure>
  <img src="/docs/assets/native-modules-android-errorscreen.png" width="200" alt="Image of error message in React Native app." />
  <figcaption>Image of error message</figcaption>
</figure>

### 向 JavaScript 發送事件

原生模組可以在不被直接調用的情況下向 JavaScript 發出事件信號。例如，您可能希望向 JavaScript 發出信號，提醒原生 Android 日曆應用中的日曆事件即將發生。最簡單的方法是使用 `RCTDeviceEventEmitter`，可以從 `ReactContext` 中獲取，如下面的程式碼片段所示。

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
...
import com.facebook.react.modules.core.DeviceEventManagerModule;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.Arguments;
...
private void sendEvent(ReactContext reactContext,
                      String eventName,
                      @Nullable WritableMap params) {
 reactContext
     .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
     .emit(eventName, params);
}

private int listenerCount = 0;

@ReactMethod
public void addListener(String eventName) {
  if (listenerCount == 0) {
    // Set up any upstream listeners or background tasks as necessary
  }

  listenerCount += 1;
}

@ReactMethod
public void removeListeners(Integer count) {
  listenerCount -= count;
  if (listenerCount == 0) {
    // Remove upstream listeners, stop unnecessary background tasks
  }
}
...
WritableMap params = Arguments.createMap();
params.putString("eventProperty", "someValue");
...
sendEvent(reactContext, "EventReminder", params);
```

</TabItem>
<TabItem value="kotlin">

```kotlin
...
import com.facebook.react.bridge.WritableMap
import com.facebook.react.bridge.Arguments
import com.facebook.react.modules.core.DeviceEventManagerModule
...

private fun sendEvent(reactContext: ReactContext, eventName: String, params: WritableMap?) {
    reactContext
      .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter::class.java)
      .emit(eventName, params)
}

private var listenerCount = 0

@ReactMethod
fun addListener(eventName: String) {
  if (listenerCount == 0) {
    // Set up any upstream listeners or background tasks as necessary
  }

  listenerCount += 1
}

@ReactMethod
fun removeListeners(count: Int) {
  listenerCount -= count
  if (listenerCount == 0) {
    // Remove upstream listeners, stop unnecessary background tasks
  }
}
...
val params = Arguments.createMap().apply {
    putString("eventProperty", "someValue")
}
...
sendEvent(reactContext, "EventReminder", params)
```

</TabItem>
</Tabs>

JavaScript modules can then register to receive events by `addListener` on the [NativeEventEmitter](https://github.com/facebook/react-native/blob/main/packages/react-native/Libraries/EventEmitter/NativeEventEmitter.js) class.

```tsx
import {NativeEventEmitter, NativeModules} from 'react-native';
...
useEffect(() => {
    const eventEmitter = new NativeEventEmitter(NativeModules.ToastExample);
    let eventListener = eventEmitter.addListener('EventReminder', event => {
      console.log(event.eventProperty) // "someValue"
    });

    // Removes the listener once unmounted
    return () => {
      eventListener.remove();
    };
  }, []);
```

### 從 startActivityForResult 獲取活動結果

You'll need to listen to `onActivityResult` if you want to get results from an activity you started with `startActivityForResult`. To do this, you must extend `BaseActivityEventListener` or implement `ActivityEventListener`. The former is preferred as it is more resilient to API changes. Then, you need to register the listener in the module's constructor like so:

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
reactContext.addActivityEventListener(mActivityResultListener);
```

</TabItem>
<TabItem value="kotlin">

```kotlin
reactContext.addActivityEventListener(mActivityResultListener);
```

</TabItem>
</Tabs>

現在，您可以通過實現以下方法來監聽 `onActivityResult`：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
@Override
public void onActivityResult(
 final Activity activity,
 final int requestCode,
 final int resultCode,
 final Intent intent) {
 // Your logic here
}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
override fun onActivityResult(
    activity: Activity?,
    requestCode: Int,
    resultCode: Int,
    intent: Intent?
) {
    // Your logic here
}
```

</TabItem>
</Tabs>

讓我們實現一個基本的圖片選擇器來演示這一點。圖片選擇器將向 JavaScript 公開方法 `pickImage`，該方法在調用時會返回圖片的路径。

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```kotlin
public class ImagePickerModule extends ReactContextBaseJavaModule {

  private static final int IMAGE_PICKER_REQUEST = 1;
  private static final String E_ACTIVITY_DOES_NOT_EXIST = "E_ACTIVITY_DOES_NOT_EXIST";
  private static final String E_PICKER_CANCELLED = "E_PICKER_CANCELLED";
  private static final String E_FAILED_TO_SHOW_PICKER = "E_FAILED_TO_SHOW_PICKER";
  private static final String E_NO_IMAGE_DATA_FOUND = "E_NO_IMAGE_DATA_FOUND";

  private Promise mPickerPromise;

  private final ActivityEventListener mActivityEventListener = new BaseActivityEventListener() {

    @Override
    public void onActivityResult(Activity activity, int requestCode, int resultCode, Intent intent) {
      if (requestCode == IMAGE_PICKER_REQUEST) {
        if (mPickerPromise != null) {
          if (resultCode == Activity.RESULT_CANCELED) {
            mPickerPromise.reject(E_PICKER_CANCELLED, "Image picker was cancelled");
          } else if (resultCode == Activity.RESULT_OK) {
            Uri uri = intent.getData();

            if (uri == null) {
              mPickerPromise.reject(E_NO_IMAGE_DATA_FOUND, "No image data found");
            } else {
              mPickerPromise.resolve(uri.toString());
            }
          }

          mPickerPromise = null;
        }
      }
    }
  };

  ImagePickerModule(ReactApplicationContext reactContext) {
    super(reactContext);

    // Add the listener for `onActivityResult`
    reactContext.addActivityEventListener(mActivityEventListener);
  }

  @Override
  public String getName() {
    return "ImagePickerModule";
  }

  @ReactMethod
  public void pickImage(final Promise promise) {
    Activity currentActivity = getCurrentActivity();

    if (currentActivity == null) {
      promise.reject(E_ACTIVITY_DOES_NOT_EXIST, "Activity doesn't exist");
      return;
    }

    // Store the promise to resolve/reject when picker returns data
    mPickerPromise = promise;

    try {
      final Intent galleryIntent = new Intent(Intent.ACTION_PICK);

      galleryIntent.setType("image/*");

      final Intent chooserIntent = Intent.createChooser(galleryIntent, "Pick an image");

      currentActivity.startActivityForResult(chooserIntent, IMAGE_PICKER_REQUEST);
    } catch (Exception e) {
      mPickerPromise.reject(E_FAILED_TO_SHOW_PICKER, e);
      mPickerPromise = null;
    }
  }
}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
class ImagePickerModule(reactContext: ReactApplicationContext) :
    ReactContextBaseJavaModule(reactContext) {

    private var pickerPromise: Promise? = null

    private val activityEventListener =
        object : BaseActivityEventListener() {
            override fun onActivityResult(
                activity: Activity?,
                requestCode: Int,
                resultCode: Int,
                intent: Intent?
            ) {
                if (requestCode == IMAGE_PICKER_REQUEST) {
                    pickerPromise?.let { promise ->
                        when (resultCode) {
                            Activity.RESULT_CANCELED ->
                                promise.reject(E_PICKER_CANCELLED, "Image picker was cancelled")
                            Activity.RESULT_OK -> {
                                val uri = intent?.data

                                uri?.let { promise.resolve(uri.toString())}
                                    ?: promise.reject(E_NO_IMAGE_DATA_FOUND, "No image data found")
                            }
                        }

                        pickerPromise = null
                    }
                }
            }
        }

    init {
        reactContext.addActivityEventListener(activityEventListener)
    }

    override fun getName() = "ImagePickerModule"

    @ReactMethod
    fun pickImage(promise: Promise) {
        val activity = currentActivity

        if (activity == null) {
            promise.reject(E_ACTIVITY_DOES_NOT_EXIST, "Activity doesn't exist")
            return
        }

        pickerPromise = promise

        try {
            val galleryIntent = Intent(Intent.ACTION_PICK).apply { type = "image\/*" }

            val chooserIntent = Intent.createChooser(galleryIntent, "Pick an image")

            activity.startActivityForResult(chooserIntent, IMAGE_PICKER_REQUEST)
        } catch (t: Throwable) {
            pickerPromise?.reject(E_FAILED_TO_SHOW_PICKER, t)
            pickerPromise = null
        }
    }

    companion object {
        const val IMAGE_PICKER_REQUEST = 1
        const val E_ACTIVITY_DOES_NOT_EXIST = "E_ACTIVITY_DOES_NOT_EXIST"
        const val E_PICKER_CANCELLED = "E_PICKER_CANCELLED"
        const val E_FAILED_TO_SHOW_PICKER = "E_FAILED_TO_SHOW_PICKER"
        const val E_NO_IMAGE_DATA_FOUND = "E_NO_IMAGE_DATA_FOUND"
    }
}
```

</TabItem>
</Tabs>

### 監聽生命週期事件

監聽活動的生命週期事件（如 `onResume`、`onPause` 等）與實作 `ActivityEventListener` 的方式非常相似。模組必須實作 `LifecycleEventListener`，然後在模組的建構函式中註冊監聽器，如下所示：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
reactContext.addLifecycleEventListener(this);
```

</TabItem>
<TabItem value="kotlin">

```kotlin
reactContext.addLifecycleEventListener(this)
```

</TabItem>
</Tabs>

現在，您可以通過實作以下方法來監聽活動的生命週期事件：

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>
<TabItem value="java">

```java
@Override
public void onHostResume() {
   // Activity `onResume`
}
@Override
public void onHostPause() {
   // Activity `onPause`
}
@Override
public void onHostDestroy() {
   // Activity `onDestroy`
}
```

</TabItem>
<TabItem value="kotlin">

```kotlin
override fun onHostResume() {
    // Activity `onResume`
}

override fun onHostPause() {
    // Activity `onPause`
}

override fun onHostDestroy() {
    // Activity `onDestroy`
}
```

</TabItem>
</Tabs>

### 執行緒處理

截至目前，在 Android 上，所有原生模組的異步方法都在單一執行緒上執行。原生模組不應對其被呼叫的執行緒有任何假設，因為當前的分配方式未來可能會變更。如果需要進行阻塞呼叫，繁重的工作應分派到內部管理的工作執行緒，並從那裡分發任何回調。