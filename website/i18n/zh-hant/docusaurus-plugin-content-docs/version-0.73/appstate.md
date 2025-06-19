---
id: appstate
title: AppState
---

`AppState` 可告知您應用程式處於前景或背景狀態，並在狀態變更時發出通知。

AppState 常用於處理推送通知時判斷意圖與適當行為。

### 應用程式狀態

- `active` - 應用程式在前景運行
- `background` - 應用程式在背景運行。使用者可能處於：
  - 其他應用程式中
  - 主畫面
  - [Android] 其他 `Activity`（即使由您的應用程式啟動）
- [iOS] `inactive` - 此狀態發生於前景與背景轉換期間，以及非活動時段（如進入多工視圖、開啟通知中心或來電時）。

更多資訊請參閱 [Apple 官方文件](https://developer.apple.com/documentation/uikit/app_and_scenes/managing_your_app_s_life_cycle)

## 基本用法

要查看當前狀態，可檢查 `AppState.currentState`（該值會保持最新）。但啟動時 `currentState` 會暫時為 null，直到 `AppState` 透過橋接取得狀態。

```SnackPlayer name=AppState%20Example
import React, {useRef, useState, useEffect} from 'react';
import {AppState, StyleSheet, Text, View} from 'react-native';

const AppStateExample = () => {
  const appState = useRef(AppState.currentState);
  const [appStateVisible, setAppStateVisible] = useState(appState.current);

  useEffect(() => {
    const subscription = AppState.addEventListener('change', nextAppState => {
      if (
        appState.current.match(/inactive|background/) &&
        nextAppState === 'active'
      ) {
        console.log('App has come to the foreground!');
      }

      appState.current = nextAppState;
      setAppStateVisible(appState.current);
      console.log('AppState', appState.current);
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
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default AppStateExample;
```

此範例永遠只會顯示「當前狀態為: active」，因為應用程式僅在 `active` 狀態時對使用者可見，而 null 狀態僅短暫出現。建議在實機而非嵌入式預覽中測試此程式碼。

---

# 參考文獻

## 事件

### `change`

當應用程式狀態變更時觸發此事件。監聽器會收到[當前應用程式狀態值](appstate#app-states)之一。

### `memoryWarning`

此事件用於記憶體警告或釋放記憶體時觸發。

### `focus` <div class="label android">Android</div>

當應用程式獲得焦點時觸發（使用者正在與應用程式互動）。

### `blur` <div class="label android">Android</div>

當使用者未主動與應用程式互動時觸發。適用於使用者下拉[通知抽屜](https://developer.android.com/guide/topics/ui/notifiers/notifications#bar-and-drawer)的情境。此時 `AppState` 不會改變，但會觸發 `blur` 事件。

## 方法

### `addEventListener()`

```tsx
static addEventListener(
  type: AppStateEvent,
  listener: (state: AppStateStatus) => void,
): NativeEventSubscription;
```

設定當指定事件類型在 AppState 上觸發時呼叫的函式。有效 `eventType` 值請參見[上述事件列表](#events)。回傳 `EventSubscription`。

## 屬性

### `currentState`

```tsx
static currentState: AppStateStatus;
```