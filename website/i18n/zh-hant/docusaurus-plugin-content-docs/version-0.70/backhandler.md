---
id: backhandler
title: BackHandler
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

Backhandler API 用於偵測 Android 裝置的實體返回按鈕操作，允許您註冊系統返回動作的事件監聽器，並控制應用程式的回應行為。此功能僅限 Android 平台使用。

事件監聽器的觸發順序為反向（最後註冊的監聽器會最先被呼叫）。

- **若某個監聽器返回 true**，則先前註冊的監聽器將不會被呼叫。
- **若沒有任何監聽器返回 true 或未註冊任何監聽器**，系統將預設執行返回按鈕功能來關閉應用程式。

> **Warning for modal users:** If your app shows an opened `Modal`, `BackHandler` will not publish any events ([see `Modal` docs](modal#onrequestclose)).

## 使用模式

```jsx
BackHandler.addEventListener('hardwareBackPress', function () {
  /**
   * this.onMainScreen and this.goBack are just examples,
   * you need to use your own implementation here.
   *
   * Typically you would use the navigator here to go to the last state.
   */

  if (!this.onMainScreen()) {
    this.goBack();
    /**
     * When true is returned the event will not be bubbled up
     * & no other back action will execute
     */
    return true;
  }
  /**
   * Returning false will let the event to bubble up & let other event listeners
   * or the system's default back action to be executed.
   */
  return false;
});
```

## 範例

以下範例實作確認使用者是否要退出應用程式的場景：

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=BackHandler&supportedPlatforms=android
import React, { useEffect } from "react";
import { Text, View, StyleSheet, BackHandler, Alert } from "react-native";

const App = () => {
  useEffect(() => {
    const backAction = () => {
      Alert.alert("Hold on!", "Are you sure you want to go back?", [
        {
          text: "Cancel",
          onPress: () => null,
          style: "cancel"
        },
        { text: "YES", onPress: () => BackHandler.exitApp() }
      ]);
      return true;
    };

    const backHandler = BackHandler.addEventListener(
      "hardwareBackPress",
      backAction
    );

    return () => backHandler.remove();
  }, []);

  return (
    <View style={styles.container}>
      <Text style={styles.text}>Click Back button!</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center"
  },
  text: {
    fontSize: 18,
    fontWeight: "bold"
  }
});

export default App;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=BackHandler&supportedPlatforms=android
import React, { Component } from "react";
import { Text, View, StyleSheet, BackHandler, Alert } from "react-native";

class App extends Component {
  backAction = () => {
    Alert.alert("Hold on!", "Are you sure you want to go back?", [
      {
        text: "Cancel",
        onPress: () => null,
        style: "cancel"
      },
      { text: "YES", onPress: () => BackHandler.exitApp() }
    ]);
    return true;
  };

  componentDidMount() {
    this.backHandler = BackHandler.addEventListener(
      "hardwareBackPress",
      this.backAction
    );
  }

  componentWillUnmount() {
    this.backHandler.remove();
  }

  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.text}>Click Back button!</Text>
      </View>
    );
  }
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center"
  },
  text: {
    fontSize: 18,
    fontWeight: "bold"
  }
});

export default App;
```

</TabItem>
</Tabs>

`BackHandler.addEventListener` 會建立事件監聽器並返回一個 `NativeEventSubscription` 物件，應使用 `NativeEventSubscription.remove` 方法清除該監聽器。

此外也可使用 `BackHandler.removeEventListener` 清除事件監聽器。請確保回調函數與 `addEventListener` 呼叫時使用的是相同函數參考，如下列範例所示：

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=BackHandler&supportedPlatforms=android
import React, { useEffect } from "react";
import { Text, View, StyleSheet, BackHandler, Alert } from "react-native";

const App = () => {
  const backAction = () => {
    Alert.alert("Hold on!", "Are you sure you want to go back?", [
      {
        text: "Cancel",
        onPress: () => null,
        style: "cancel"
      },
      { text: "YES", onPress: () => BackHandler.exitApp() }
    ]);
    return true;
  };

  useEffect(() => {
    BackHandler.addEventListener("hardwareBackPress", backAction);

    return () =>
      BackHandler.removeEventListener("hardwareBackPress", backAction);
  }, []);

  return (
    <View style={styles.container}>
      <Text style={styles.text}>Click Back button!</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center"
  },
  text: {
    fontSize: 18,
    fontWeight: "bold"
  }
});

export default App;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=BackHandler&supportedPlatforms=android
import React, { Component } from "react";
import { Text, View, StyleSheet, BackHandler, Alert } from "react-native";

class App extends Component {
  backAction = () => {
    Alert.alert("Hold on!", "Are you sure you want to go back?", [
      {
        text: "Cancel",
        onPress: () => null,
        style: "cancel"
      },
      { text: "YES", onPress: () => BackHandler.exitApp() }
    ]);
    return true;
  };

  componentDidMount() {
    BackHandler.addEventListener("hardwareBackPress", this.backAction);
  }

  componentWillUnmount() {
    BackHandler.removeEventListener("hardwareBackPress", this.backAction);
  }

  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.text}>Click Back button!</Text>
      </View>
    );
  }
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center"
  },
  text: {
    fontSize: 18,
    fontWeight: "bold"
  }
});

export default App;
```

</TabItem>
</Tabs>

## 與 React Navigation 搭配使用

若您使用 React Navigation 進行畫面導航，可參考其指南 [自訂 Android 返回按鈕行為](https://reactnavigation.org/docs/custom-android-back-button-handling/)

## Backhandler 鉤子

[React Native Hooks](https://github.com/react-native-community/hooks#usebackhandler) 提供了一個方便的 `useBackHandler` 鉤子，可簡化事件監聽器的設定流程。

---

# 參考文件

## 方法

### `addEventListener()`

```jsx
static addEventListener(eventName, handler)
```

---

### `exitApp()`

```jsx
static exitApp()
```

---

### `removeEventListener()`

```jsx
static removeEventListener(eventName, handler)
```