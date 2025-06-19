---
id: appstate
title: AppState
---

`AppState` 可以告訴您應用程式目前處於前景或背景狀態，並在狀態變更時通知您。

AppState 經常被用來處理推送通知時，判斷意圖和決定適當的行為。

### 應用程式狀態

- `active` - 應用程式正在前景執行
- `background` - 應用程式正在背景執行。使用者可能處於以下情況：
  - 正在使用其他應用程式
  - 在主畫面
  - [Android] 在其他 `Activity` 中（即使是由您的應用程式啟動的）
- [iOS] `inactive` - 這種狀態發生在前景與背景之間的過渡期，或是處於非活動期間，例如進入多工視圖、開啟通知中心或有來電時。

更多資訊，請參閱 [Apple 的官方文件](https://developer.apple.com/documentation/uikit/app_and_scenes/managing_your_app_s_life_cycle)

## 基本用法

要查看當前狀態，您可以檢查 `AppState.currentState`，它會保持最新狀態。然而，在啟動時，`currentState` 會是 null，直到 `AppState` 透過橋接器取得狀態。

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

這個範例永遠只會顯示「Current state is: active」，因為應用程式只有在 `active` 狀態時對使用者可見，而 null 狀態只會短暫出現。如果您想實驗這段程式碼，建議使用自己的設備而非嵌入式預覽。

---

# 參考

## 事件

### `change`

當應用程式狀態變更時會收到此事件。監聽器會收到一個[當前應用程式狀態值](appstate#app-states)作為參數。

### `memoryWarning`

此事件用於需要發出記憶體警告或釋放記憶體時。

### `focus` <div class="label android">Android</div>

當應用程式獲得焦點時（使用者正在與應用程式互動）接收。

### `blur` <div class="label android">Android</div>

當使用者未積極與應用程式互動時接收。在使用者下拉[通知抽屜](https://developer.android.com/guide/topics/ui/notifiers/notifications#bar-and-drawer)時特別有用。`AppState` 不會改變，但會觸發 `blur` 事件。

## 方法

### `addEventListener()`

```tsx
static addEventListener(
  type: AppStateEvent,
  listener: (state: AppStateStatus) => void,
): NativeEventSubscription;
```

設定一個函式，當 AppState 上發生指定的事件類型時會被呼叫。`eventType` 的有效值[如上所列](#events)。回傳 `EventSubscription`。

## 屬性

### `currentState`

```tsx
static currentState: AppStateStatus;
```