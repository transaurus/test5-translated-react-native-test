---
id: interactionmanager
title: InteractionManager
---

InteractionManager 允許在互動/動畫完成後安排長時間運行的任務。這特別能讓 JavaScript 動畫流暢運行。

應用程式可以透過以下方式安排互動後執行的任務：

```jsx
InteractionManager.runAfterInteractions(() => {
  // ...long-running synchronous task...
});
```

與其他排程方案的比較：

- requestAnimationFrame(): 用於隨時間推移動畫化視圖的代碼。
- setImmediate/setTimeout(): 稍後運行代碼，注意這可能會延遲動畫。
- runAfterInteractions(): 稍後運行代碼，且不會延遲正在進行的動畫。

觸控處理系統將一個或多個活躍觸控視為「互動」，並會延遲 `runAfterInteractions()` 回調，直到所有觸控結束或被取消。

InteractionManager 還允許應用程式通過在動畫開始時創建互動「句柄」，並在完成時清除它來註冊動畫：

```jsx
var handle = InteractionManager.createInteractionHandle();
// run animation... (`runAfterInteractions` tasks are queued)
// later, on animation completion:
InteractionManager.clearInteractionHandle(handle);
// queued tasks run if all handles were cleared
```

`runAfterInteractions` takes either a plain callback function, or a `PromiseTask` object with a `gen` method that returns a `Promise`. If a `PromiseTask` is supplied, then it is fully resolved (including asynchronous dependencies that also schedule more tasks via `runAfterInteractions`) before starting on the next task that might have been queued up synchronously earlier.

默認情況下，排隊的任務會在一個 `setImmediate` 批次中一起執行。如果使用正數調用 `setDeadline`，則任務只會在截止時間（以 js 事件循環運行時間為準）接近時執行，此時執行會通過 setTimeout 讓步，允許觸控等事件開始互動並阻止排隊任務執行，使應用更具響應性。

---

## 範例

### 基礎

```SnackPlayer name=InteractionManager%20Function%20Component%20Basic%20Example&supportedPlatforms=ios,android
import React, { useEffect } from "react";
import {
  Alert,
  Animated,
  InteractionManager,
  Platform,
  StyleSheet,
  Text,
  View,
  useAnimatedValue,
} from "react-native";

const instructions = Platform.select({
  ios: "Press Cmd+R to reload,\n" + "Cmd+D or shake for dev menu",
  android:
    "Double tap R on your keyboard to reload,\n" +
    "Shake or press menu button for dev menu",
});

const useMount = func => useEffect(() => func(), []);

const useFadeIn = (duration = 5000) => {
  const opacity = useAnimatedValue(0);

  // Running the animation when the component is mounted
  useMount(() => {
    // Animated.timing() create a interaction handle by default, if you want to disabled that
    // behaviour you can set isInteraction to false to disabled that.
    Animated.timing(opacity, {
      toValue: 1,
      duration,
    }).start();
  });

  return opacity;
};

const Ball = ({ onShown }) => {
  const opacity = useFadeIn();

  // Running a method after the animation
  useMount(() => {
    const interactionPromise = InteractionManager.runAfterInteractions(() => onShown());
    return () => interactionPromise.cancel();
  });

  return <Animated.View style={[styles.ball, { opacity }]} />;
};

const App = () => {
  return (
    <View style={styles.container}>
      <Text>{instructions}</Text>
      <Ball onShown={() => Alert.alert("Animation is done")} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", alignItems: "center" },
  ball: {
    width: 100,
    height: 100,
    backgroundColor: "salmon",
    borderRadius: 100,
  },
});

export default App;
```

### 進階

```SnackPlayer name=InteractionManager%20Function%20Component%20Advanced%20Example&supportedPlatforms=ios,android
import React, { useEffect } from "react";
import {
  Alert,
  Animated,
  InteractionManager,
  Platform,
  StyleSheet,
  Text,
  View,
} from "react-native";

const instructions = Platform.select({
  ios: "Press Cmd+R to reload,\n" + "Cmd+D or shake for dev menu",
  android:
    "Double tap R on your keyboard to reload,\n" +
    "Shake or press menu button for dev menu",
});

const useMount = func => useEffect(() => func(), []);

// You can create a custom interaction/animation and add
// support for InteractionManager
const useCustomInteraction = (timeLocked = 2000) => {
  useMount(() => {
    const handle = InteractionManager.createInteractionHandle();

    setTimeout(
      () => InteractionManager.clearInteractionHandle(handle),
      timeLocked
    );

    return () => InteractionManager.clearInteractionHandle(handle);
  });
};

const Ball = ({ onInteractionIsDone }) => {
  useCustomInteraction();

  // Running a method after the interaction
  useMount(() => {
    InteractionManager.runAfterInteractions(() => onInteractionIsDone());
  });

  return <Animated.View style={[styles.ball]} />;
};

const App = () => {
  return (
    <View style={styles.container}>
      <Text>{instructions}</Text>
      <Ball onInteractionIsDone={() => Alert.alert("Interaction is done")} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", alignItems: "center" },
  ball: {
    width: 100,
    height: 100,
    backgroundColor: "salmon",
    borderRadius: 100,
  },
});

export default App;
```

# 參考

## 方法

### `runAfterInteractions()`

```jsx
static runAfterInteractions(task)
```

安排一個在所有互動完成後運行的函數。返回一個可取消的「promise」。

---

### `createInteractionHandle()`

```jsx
static createInteractionHandle()
```

通知管理器互動已開始。

---

### `clearInteractionHandle()`

```jsx
static clearInteractionHandle(handle)
```

通知管理器互動已完成。

---

### `setDeadline()`

```jsx
static setDeadline(deadline)
```

正數將使用 setTimeout 在 eventLoopRunningTime 達到截止值後安排任何任務，否則所有任務將在一個 setImmediate 批次中執行（默認）。