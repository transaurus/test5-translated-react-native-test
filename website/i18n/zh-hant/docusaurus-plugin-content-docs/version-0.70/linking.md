---
id: linking
title: Linking
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

`Linking` 提供了一個通用介面來處理應用程式的進出連結。

每個連結(URL)都有一個URL Scheme，某些網站以`https://`或`http://`為前綴，其中的`http`就是URL Scheme。我們簡稱其為scheme。

除了`https`之外，您可能也熟悉`mailto` scheme。當您開啟帶有mailto scheme的連結時，作業系統會啟動已安裝的郵件應用程式。同樣地，也有用於撥打電話和發送簡訊的scheme。更多關於[內建URL](#built-in-url-schemes) scheme的資訊請參閱下文。

如同使用mailto scheme，透過自訂url scheme也可以連結到其他應用程式。例如，當您收到Slack的**魔術連結**郵件時，**啟動Slack**按鈕實際上是一個錨點標籤，其href屬性類似於：`slack://secret/magic-login/other-secret`。與Slack類似，您可以告訴作業系統您想處理某個自訂scheme。當Slack應用程式開啟時，它會收到用於開啟它的URL。這通常被稱為深度連結。更多關於如何[獲取深度連結](#get-the-deep-link)到您應用程式的資訊請參閱下文。

自訂URL scheme並非在行動裝置上開啟應用程式的唯一方式。您不會想在電子郵件中使用自訂URL scheme連結，因為這些連結在桌機上會失效。相反地，您應該使用常規的`https`連結，例如`https://www.myapp.io/records/1234546`，並在行動裝置上讓該連結開啟您的應用程式。Android稱之為**深度連結**(iOS上稱為通用連結)。

### 內建URL Schemes

如引言所述，每個平台都有一些用於核心功能的URL scheme。以下是一個非詳盡的清單，但涵蓋了最常用的scheme。

| Scheme           | Description                                | iOS | Android |
| ---------------- | ------------------------------------------ | --- | ------- |
| `mailto`         | Open mail app, eg: mailto: support@expo.io | ✅  | ✅      |
| `tel`            | Open phone app, eg: tel:+123456789         | ✅  | ✅      |
| `sms`            | Open SMS app, eg: sms:+123456789           | ✅  | ✅      |
| `https` / `http` | Open web browser app, eg: https://expo.io  | ✅  | ✅      |

### 啟用深度連結

<div className="banner-native-code-required">
  <h3>Projects with Native Code Only</h3>
  <p>The following section only applies to projects with native code exposed. If you are using the managed Expo workflow, see the guide on <a href="https://docs.expo.dev/guides/linking/">Linking</a> in the Expo documentation for the appropriate alternative.</p>
</div>

如果您想在應用程式中啟用深度連結，請閱讀以下指南：

<Tabs groupId="syntax" queryString defaultValue={constants.defaultPlatform} values={constants.platforms}>
<TabItem value="android">

> For instructions on how to add support for deep linking on Android, refer to [Enabling Deep Links for App Content - Add Intent Filters for Your Deep Links](http://developer.android.com/training/app-indexing/deep-linking.html#adding-filters).

If you wish to receive the intent in an existing instance of MainActivity, you may set the `launchMode` of MainActivity to `singleTask` in `AndroidManifest.xml`. See [`<activity>`](http://developer.android.com/guide/topics/manifest/activity-element.html) documentation for more information.

```xml
<activity
  android:name=".MainActivity"
  android:launchMode="singleTask">
```

</TabItem>
<TabItem value="ios">

> **NOTE:** On iOS, you'll need to add the `LinkingIOS` folder into your header search paths as described in step 3 [here](linking-libraries-ios#step-3). If you also want to listen to incoming app links during your app's execution, you'll need to add the following lines to your `*AppDelegate.m`:

```objectivec
// iOS 9.x or newer
#import <React/RCTLinkingManager.h>

- (BOOL)application:(UIApplication *)application
   openURL:(NSURL *)url
   options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
  return [RCTLinkingManager application:application openURL:url options:options];
}
```

If you're targeting iOS 8.x or older, you can use the following code instead:

```objectivec
// iOS 8.x or older
#import <React/RCTLinkingManager.h>

- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
  sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
  return [RCTLinkingManager application:application openURL:url
                      sourceApplication:sourceApplication annotation:annotation];
}
```

If your app is using [Universal Links](https://developer.apple.com/ios/universal-links/), you'll need to add the following code as well:

```objectivec
- (BOOL)application:(UIApplication *)application continueUserActivity:(nonnull NSUserActivity *)userActivity
 restorationHandler:(nonnull void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler
{
 return [RCTLinkingManager application:application
                  continueUserActivity:userActivity
                    restorationHandler:restorationHandler];
}
```

</TabItem>
</Tabs>

### 處理深度連結

有兩種方式可以處理開啟您應用程式的URL。

#### 1. 如果應用程式已經開啟，應用程式會被帶到前景並觸發Linking的'url'事件

您可以使用`Linking.addEventListener('url', callback)`來處理這些事件 - 它會呼叫`callback({ url })`並傳入連結的URL

#### 2. 如果應用程式尚未開啟，它會被開啟且url會作為initialURL傳入

您可以使用`Linking.getInitialURL()`來處理這些事件 - 它會回傳一個Promise，該Promise會解析為URL(如果有的話)。

---

## 範例

### 開啟連結與深度連結(通用連結)

```SnackPlayer name=Linking%20Function%20Component%20Example&supportedPlatforms=ios,android
import React, { useCallback } from "react";
import { Alert, Button, Linking, StyleSheet, View } from "react-native";

const supportedURL = "https://google.com";

const unsupportedURL = "slack://open?team=123456";

const OpenURLButton = ({ url, children }) => {
  const handlePress = useCallback(async () => {
    // Checking if the link is supported for links with custom URL scheme.
    const supported = await Linking.canOpenURL(url);

    if (supported) {
      // Opening the link with some app, if the URL scheme is "http" the web link should be opened
      // by some browser in the mobile
      await Linking.openURL(url);
    } else {
      Alert.alert(`Don't know how to open this URL: ${url}`);
    }
  }, [url]);

  return <Button title={children} onPress={handlePress} />;
};

const App = () => {
  return (
    <View style={styles.container}>
      <OpenURLButton url={supportedURL}>Open Supported URL</OpenURLButton>
      <OpenURLButton url={unsupportedURL}>Open Unsupported URL</OpenURLButton>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", alignItems: "center" },
});

export default App;
```

### 開啟自訂設定

```SnackPlayer name=Linking%20Function%20Component%20Example&supportedPlatforms=ios,android
import React, { useCallback } from "react";
import { Button, Linking, StyleSheet, View } from "react-native";

const OpenSettingsButton = ({ children }) => {
  const handlePress = useCallback(async () => {
    // Open the custom settings if the app has one
    await Linking.openSettings();
  }, []);

  return <Button title={children} onPress={handlePress} />;
};

const App = () => {
  return (
    <View style={styles.container}>
      <OpenSettingsButton>Open Settings</OpenSettingsButton>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", alignItems: "center" },
});

export default App;
```

### 獲取深度連結

```SnackPlayer name=Linking%20Function%20Component%20Example&supportedPlatforms=ios,android
import React, { useState, useEffect } from "react";
import { Linking, StyleSheet, Text, View } from "react-native";

const useMount = func => useEffect(() => func(), []);

const useInitialURL = () => {
  const [url, setUrl] = useState(null);
  const [processing, setProcessing] = useState(true);

  useMount(() => {
    const getUrlAsync = async () => {
      // Get the deep link used to open the app
      const initialUrl = await Linking.getInitialURL();

      // The setTimeout is just for testing purpose
      setTimeout(() => {
        setUrl(initialUrl);
        setProcessing(false);
      }, 1000);
    };

    getUrlAsync();
  });

  return { url, processing };
};

const App = () => {
  const { url: initialUrl, processing } = useInitialURL();

  return (
    <View style={styles.container}>
      <Text>
        {processing
          ? `Processing the initial url from a deep link`
          : `The deep link is: ${initialUrl || "None"}`}
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", alignItems: "center" },
});

export default App;
```

### 發送Intents(Android)

```SnackPlayer name=Linking%20Function%20Component%20Example&supportedPlatforms=android
import React, { useCallback } from "react";
import { Alert, Button, Linking, StyleSheet, View } from "react-native";

const SendIntentButton = ({ action, extras, children }) => {
  const handlePress = useCallback(async () => {
    try {
      await Linking.sendIntent(action, extras);
    } catch (e) {
      Alert.alert(e.message);
    }
  }, [action, extras]);

  return <Button title={children} onPress={handlePress} />;
};

const App = () => {
  return (
    <View style={styles.container}>
      <SendIntentButton action="android.intent.action.POWER_USAGE_SUMMARY">
        Power Usage Summary
      </SendIntentButton>
      <SendIntentButton
        action="android.settings.APP_NOTIFICATION_SETTINGS"
        extras={[
          { "android.provider.extra.APP_PACKAGE": "com.facebook.katana" },
        ]}
      >
        App Notification Settings
      </SendIntentButton>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", alignItems: "center" },
});

export default App;
```

# 參考

## 方法

### `addEventListener()`

```jsx
addEventListener(type, handler);
```

透過監聽`url`事件類型並提供處理函式，為Linking變更添加處理器。

---

### `canOpenURL()`

```jsx
canOpenURL(url);
```

判斷已安裝的應用程式是否可以處理給定的URL。

該方法返回一個 `Promise` 物件。當確定給定的 URL 是否能被處理時，Promise 會被解析，第一個參數表示該 URL 是否能被打開。

The `Promise` will reject on Android if it was impossible to check if the URL can be opened or when targeting Android 11 (SDK 30) if you didn't specify the relevant intent queries in `AndroidManifest.xml`. Similarly on iOS, the promise will reject if you didn't add the specific scheme in the `LSApplicationQueriesSchemes` key inside `Info.plist` (see bellow).

**參數：**

| Name                                                     | Type   | Description      |
| -------------------------------------------------------- | ------ | ---------------- |
| url <div className="label basic required">Required</div> | string | The URL to open. |

> 對於網頁 URL，必須正確設置協議（`"http://"`、`"https://"`）！

> This method has limitations on iOS 9+. From [the official Apple documentation](https://developer.apple.com/documentation/uikit/uiapplication/1622952-canopenurl):
>
> - If your app is linked against an earlier version of iOS but is running in iOS 9.0 or later, you can call this method up to 50 times. After reaching that limit, subsequent calls always resolve to `false`. If the user reinstalls or upgrades the app, iOS resets the limit.
>
> As of iOS 9, your app also needs to provide the `LSApplicationQueriesSchemes` key inside `Info.plist` or `canOpenURL()` will always resolve to `false`.

> 當目標為 Android 11 (SDK 30) 時，你必須在 `AndroidManifest.xml` 中指定要處理的 scheme 的 intent。常見的 intent 列表可以在 [這裡](https://developer.android.com/guide/components/intents-common) 找到。
>
> 例如，要處理 `https` scheme，需要在清單中添加以下內容：
>
> ```
> <manifest ...>
>     <queries>
>         <intent>
>             <action android:name="android.intent.action.VIEW" />
>             <data android:scheme="https"/>
>         </intent>
>     </queries>
> </manifest>
> ```

---

### `getInitialURL()`

```jsx
getInitialURL();
```

如果應用程式啟動是由應用程式鏈接觸發的，則會返回該鏈接的 URL，否則返回 `null`。

> 要在 Android 上支援深度鏈接，請參考 http://developer.android.com/training/app-indexing/deep-linking.html#handling-intents

> getInitialURL may return `null` while debugging is enabled. Disable the debugger to ensure it gets passed.

---

### `openSettings()`

```jsx
openSettings();
```

打開「設定」應用程式並顯示應用程式的自定義設定（如果有的話）。

---

### `openURL()`

```jsx
openURL(url);
```

嘗試使用任何已安裝的應用程式打開給定的 `url`。

You can use other URLs, like a location (e.g. "geo:37.484847,-122.148386" on Android or "http://maps.apple.com/?ll=37.484847,-122.148386" on iOS), a contact, or any other URL that can be opened with the installed apps.

該方法返回一個 `Promise` 物件。如果用戶確認打開對話框或 URL 自動打開，Promise 會被解析。如果用戶取消打開對話框或沒有註冊的應用程式可以處理該 URL，Promise 會被拒絕。

**參數：**

| Name                                                     | Type   | Description      |
| -------------------------------------------------------- | ------ | ---------------- |
| url <div className="label basic required">Required</div> | string | The URL to open. |

> 如果系統不知道如何打開指定的 URL，此方法將失敗。如果你傳入的是非 HTTP(S) URL，最好先檢查 `canOpenURL()`。

> 對於網頁 URL，必須正確設置協議（`"http://"`、`"https://"`）！

> 此方法在模擬器中的行為可能有所不同，例如 iOS 模擬器無法處理 `"tel:"` 連結，因為無法存取撥號應用程式。

---

### `removeEventListener()`

```jsx
removeEventListener(type, handler);
```

> **Deprecated.** Use the `remove()` method on the event subscription returned by [`addEventListener()`](#addeventlistener).

---

### `sendIntent()` <div class="label android">Android</div>

```jsx
sendIntent(action, extras);
```

啟動帶有附加資料的 Android intent。

**參數：**

| Name                                                        | Type                                                       |
| ----------------------------------------------------------- | ---------------------------------------------------------- |
| action <div className="label basic required">Required</div> | string                                                     |
| extras                                                      | `Array<{key: string, value: string ｜ number ｜ boolean}>` |