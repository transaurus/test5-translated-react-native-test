---
id: appstate
title: AppState
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

`AppState` 可以告訴您應用程式是處於前景還是背景狀態，並在狀態改變時通知您。

AppState 經常被用來判斷處理推送通知時的意圖和適當行為。

### 應用程式狀態

- `active` - 應用程式正在前景運行
- `background` - 應用程式正在背景運行。使用者可能處於以下情況：
  - 正在使用其他應用程式
  - 在首頁畫面
  - [Android] 在其他 `Activity` 中（即使是由您的應用程式啟動的）
- [iOS] `inactive` - 這種狀態發生在前景與背景轉換期間，以及處於非活動狀態時，例如進入多工視圖、打開通知中心或來電時。

更多資訊請參閱 [Apple 官方文件](https://developer.apple.com/documentation/uikit/app_and_scenes/managing_your_app_s_life_cycle)

## 基本用法

要查看當前狀態，您可以檢查 `AppState.currentState`，該值會保持最新。然而，在啟動時，`currentState` 會暫時為 null，直到 `AppState` 通過橋接獲取到狀態。

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=AppState%20Function%20Component%20Example
import React, { useRef, useState, useEffect } from "react";
import { AppState, StyleSheet, Text, View } from "react-native";

const AppStateExample = () => {
  const appState = useRef(AppState.currentState);
  const [appStateVisible, setAppStateVisible] = useState(appState.current);

  useEffect(() => {
    const subscription = AppState.addEventListener("change", nextAppState => {
      if (
        appState.current.match(/inactive|background/) &&
        nextAppState === "active"
      ) {
        console.log("App has come to the foreground!");
      }

      appState.current = nextAppState;
      setAppStateVisible(appState.current);
      console.log("AppState", appState.current);
    });

    return () => {
      subscription.remove();
    };
  }, []);

  return (
    <View style={styles.container}>
      <Text>Current state is: {appStateVisible}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
  },
});

export default AppStateExample;
```

If you don't want to see the AppState update from `active` to `inactive` on iOS you can remove the state variable and use the `appState.current` value.

</TabItem>
<TabItem value="classical">

```SnackPlayer name=AppState%20Class%20Component%20Example
import React, { Component } from "react";
import { AppState, StyleSheet, Text, View } from "react-native";

class AppStateExample extends Component {
  state = {
    appState: AppState.currentState
  };

  componentDidMount() {
    this.appStateSubscription = AppState.addEventListener(
      "change",
      nextAppState => {
        if (
          this.state.appState.match(/inactive|background/) &&
          nextAppState === "active"
        ) {
          console.log("App has come to the foreground!");
        }
        this.setState({ appState: nextAppState });
      }
    );
  }

  componentWillUnmount() {
    this.appStateSubscription.remove();
  }

  render() {
    return (
      <View style={styles.container}>
        <Text>Current state is: {this.state.appState}</Text>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center"
  }
});

export default AppStateExample;
```

</TabItem>
</Tabs>

此範例永遠只會顯示「當前狀態是：active」，因為應用程式只有在 `active` 狀態時對使用者可見，而 null 狀態只會短暫出現。如果想實驗此程式碼，建議使用自己的設備而非嵌入式預覽。

---

# 參考

## 事件

### `change`

當應用程式狀態改變時會收到此事件。監聽器會收到[當前應用程式狀態值](appstate#app-states)之一。

### `memoryWarning`

此事件用於需要發出記憶體警告或釋放記憶體時。

### `focus` <div class="label android">Android</div>

當應用程式獲得焦點時接收（使用者正在與應用程式互動）。

### `blur` <div class="label android">Android</div>

當使用者未主動與應用程式互動時接收。適用於使用者下拉[通知抽屜](https://developer.android.com/guide/topics/ui/notifiers/notifications#bar-and-drawer)的情況。此時 `AppState` 不會改變，但會觸發 `blur` 事件。

## 方法

### `addEventListener()`

```jsx
addEventListener(eventType, listener);
```

設置一個函數，當 AppState 上發生指定事件類型時會被調用。有效的 `eventType` 值請參見[上方列表](#events)。返回 `EventSubscription`。

---

### `removeEventListener()`

```jsx
removeEventListener(eventType, listener);
```

> **Deprecated.** Use the `remove()` method on the `EventSubscription` returned by [`addEventListener()`](#addeventlistener).

## 屬性

### `currentState`

```jsx
AppState.currentState;
```