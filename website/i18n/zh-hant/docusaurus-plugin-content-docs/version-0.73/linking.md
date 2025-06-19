---
id: linking
title: Linking
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

`Linking` 提供了一個通用介面來處理應用程式的進出連結。

每個連結（URL）都有一個 URL 方案，某些網站以 `https://` 或 `http://` 為前綴，其中的 `http` 就是 URL 方案。我們簡稱它為「方案」。

除了 `https`，你可能也熟悉 `mailto` 方案。當你開啟一個帶有 mailto 方案的連結時，作業系統會開啟已安裝的郵件應用程式。同樣地，也有用於撥打電話和發送簡訊的方案。更多關於[內建 URL](#built-in-url-schemes) 方案的資訊請見下文。

就像使用 mailto 方案一樣，也可以透過自訂 URL 方案連結到其他應用程式。例如，當你從 Slack 收到一封 **Magic Link** 郵件時，**Launch Slack** 按鈕是一個錨點標籤，其 href 看起來像這樣：`slack://secret/magic-login/other-secret`。與 Slack 類似，你可以告訴作業系統你想要處理一個自訂方案。當 Slack 應用程式開啟時，它會接收到用來開啟它的 URL。這通常被稱為深度連結。更多關於如何[獲取深度連結](#get-the-deep-link)到你的應用程式的資訊請見下文。

自訂 URL 方案並不是在行動裝置上開啟應用程式的唯一方式。你不希望在電子郵件中的連結使用自訂 URL 方案，因為這樣在桌面上連結會失效。相反地，你希望使用常規的 `https` 連結，例如 `https://www.myapp.io/records/1234546`，並在行動裝置上讓該連結開啟你的應用程式。Android 稱之為**深度連結**（Universal Links - iOS）。

### 內建 URL 方案

如引言所述，有一些核心功能的 URL 方案存在於每個平台上。以下是一個非詳盡的清單，但涵蓋了最常用的方案。

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

如果你想在應用程式中啟用深度連結，請閱讀以下指南：

<Tabs groupId="syntax" queryString defaultValue={constants.defaultPlatform} values={constants.platforms}>
<TabItem value="android">

> For instructions on how to add support for deep linking on Android, refer to [Enabling Deep Links for App Content - Add Intent Filters for Your Deep Links](https://developer.android.com/training/app-indexing/deep-linking.html#adding-filters).

If you wish to receive the intent in an existing instance of MainActivity, you may set the `launchMode` of MainActivity to `singleTask` in `AndroidManifest.xml`. See [`<activity>`](https://developer.android.com/guide/topics/manifest/activity-element.html) documentation for more information.

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

有兩種方式可以處理開啟你應用程式的 URL。

#### 1. 如果應用程式已經開啟，應用程式會被帶到前景並觸發一個 Linking 'url' 事件

你可以使用 `Linking.addEventListener('url', callback)` 來處理這些事件——它會呼叫 `callback({url})` 並帶入連結的 URL。

#### 2. 如果應用程式尚未開啟，它會被開啟並且 URL 會作為 initialURL 傳入

你可以使用 `Linking.getInitialURL()` 來處理這些事件——它會回傳一個 Promise，該 Promise 會解析為 URL（如果有的話）。

---

## 範例

### 開啟連結和深度連結（Universal Links）

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=ios,android&ext=js
import React, {useCallback} from 'react';
import {Alert, Button, Linking, StyleSheet, View} from 'react-native';

const supportedURL = 'https://google.com';

const unsupportedURL = 'slack://open?team=123456';

const OpenURLButton = ({url, children}) => {
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
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=ios,android&ext=tsx
import React, {useCallback} from 'react';
import {Alert, Button, Linking, StyleSheet, View} from 'react-native';

const supportedURL = 'https://google.com';

const unsupportedURL = 'slack://open?team=123456';

type OpenURLButtonProps = {
  url: string;
  children: string;
};

const OpenURLButton = ({url, children}: OpenURLButtonProps) => {
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
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
</Tabs>

### 開啟自訂設定

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=ios,android&ext=js
import React, {useCallback} from 'react';
import {Button, Linking, StyleSheet, View} from 'react-native';

const OpenSettingsButton = ({children}) => {
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
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=ios,android&ext=tsx
import React, {useCallback} from 'react';
import {Button, Linking, StyleSheet, View} from 'react-native';

type OpenSettingsButtonProps = {
  children: string;
};

const OpenSettingsButton = ({children}: OpenSettingsButtonProps) => {
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
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
</Tabs>

### 獲取深度連結

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=ios,android&ext=js
import React, {useState, useEffect} from 'react';
import {Linking, StyleSheet, Text, View} from 'react-native';

const useInitialURL = () => {
  const [url, setUrl] = useState(null);
  const [processing, setProcessing] = useState(true);

  useEffect(() => {
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
  }, []);

  return {url, processing};
};

const App = () => {
  const {url: initialUrl, processing} = useInitialURL();

  return (
    <View style={styles.container}>
      <Text>
        {processing
          ? 'Processing the initial url from a deep link'
          : `The deep link is: ${initialUrl || 'None'}`}
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=ios,android&ext=tsx
import React, {useState, useEffect} from 'react';
import {Linking, StyleSheet, Text, View} from 'react-native';

const useInitialURL = () => {
  const [url, setUrl] = useState<string | null>(null);
  const [processing, setProcessing] = useState(true);

  useEffect(() => {
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
  }, []);

  return {url, processing};
};

const App = () => {
  const {url: initialUrl, processing} = useInitialURL();

  return (
    <View style={styles.container}>
      <Text>
        {processing
          ? 'Processing the initial url from a deep link'
          : `The deep link is: ${initialUrl || 'None'}`}
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
</Tabs>

### 發送 Intents（Android）

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=android&ext=js
import React, {useCallback} from 'react';
import {Alert, Button, Linking, StyleSheet, View} from 'react-native';

const SendIntentButton = ({action, extras, children}) => {
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
          {
            key: 'android.provider.extra.APP_PACKAGE',
            value: 'com.facebook.katana',
          },
        ]}>
        App Notification Settings
      </SendIntentButton>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Linking%20Example&ext=tsx
import React, {useCallback} from 'react';
import {Alert, Button, Linking, StyleSheet, View} from 'react-native';

type SendIntentButtonProps = {
  action: string;
  children: string;
  extras?: Array<{
    key: string;
    value: string | number | boolean;
  }>;
};

const SendIntentButton = ({
  action,
  extras,
  children,
}: SendIntentButtonProps) => {
  const handlePress = useCallback(async () => {
    try {
      await Linking.sendIntent(action, extras);
    } catch (e: any) {
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
          {
            key: 'android.provider.extra.APP_PACKAGE',
            value: 'com.facebook.katana',
          },
        ]}>
        App Notification Settings
      </SendIntentButton>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
</Tabs>

# 參考

## 方法

### `addEventListener()`

```tsx
static addEventListener(
  type: 'url',
  handler: (event: {url: string}) => void,
): EmitterSubscription;
```

透過監聽 `url` 事件類型並提供處理程序，來為 Linking 變更添加處理程序。

---

### `canOpenURL()`

```tsx
static canOpenURL(url: string): Promise<boolean>;
```

判斷已安裝的應用程式是否可以處理給定的 URL。

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

```tsx
static getInitialURL(): Promise<string | null>;
```

如果應用程式的啟動是由應用程式鏈接觸發的，則會返回該鏈接的 URL，否則返回 `null`。

> 要在 Android 上支持深度鏈接，請參考 https://developer.android.com/training/app-indexing/deep-linking.html#handling-intents

> getInitialURL may return `null` while debugging is enabled. Disable the debugger to ensure it gets passed.

---

### `openSettings()`

```tsx
static openSettings(): Promise<void>;
```

打開設置應用程式並顯示應用程式的自定義設置（如果有的話）。

---

### `openURL()`

```tsx
static openURL(url: string): Promise<any>;
```

嘗試使用任何已安裝的應用程式打開給定的 `url`。

You can use other URLs, like a location (e.g. "geo:37.484847,-122.148386" on Android or "https://maps.apple.com/?ll=37.484847,-122.148386" on iOS), a contact, or any other URL that can be opened with the installed apps.

該方法返回一個 `Promise` 物件。如果用戶確認打開對話框或 URL 自動打開，Promise 會被解析。如果用戶取消打開對話框或沒有註冊的應用程式可以處理該 URL，Promise 會被拒絕。

**參數：**

| Name                                                     | Type   | Description      |
| -------------------------------------------------------- | ------ | ---------------- |
| url <div className="label basic required">Required</div> | string | The URL to open. |

> 如果系統不知道如何打開指定的 URL，此方法將失敗。如果你傳入的是非 http(s) URL，最好先檢查 `canOpenURL()`。

> 對於網頁 URL，必須正確設置協議（`"http://"`、`"https://"`）！

> 此方法在模擬器中的行為可能有所不同，例如 iOS 模擬器無法處理 `"tel:"` 連結，因為無法存取撥號應用程式。

---

### `sendIntent()` <div class="label android">Android</div>

```tsx
static sendIntent(
  action: string,
  extras?: Array<{key: string; value: string | number | boolean}>,
): Promise<void>;
```

啟動帶有附加資料的 Android intent。

**參數：**

| Name                                                        | Type                                                       |
| ----------------------------------------------------------- | ---------------------------------------------------------- |
| action <div className="label basic required">Required</div> | string                                                     |
| extras                                                      | `Array<{key: string, value: string ｜ number ｜ boolean}>` |