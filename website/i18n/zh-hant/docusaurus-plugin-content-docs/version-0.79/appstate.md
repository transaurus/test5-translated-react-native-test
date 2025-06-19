---
id: appstate
title: AppState
---

`AppState` 可以告訴您應用程式是處於前景還是背景狀態，並在狀態變更時通知您。

AppState 經常被用來判斷處理推送通知時的意圖和適當行為。

### 應用程式狀態

- `active` - 應用程式正在前景執行
- `background` - 應用程式正在背景執行。使用者可能處於以下情況：
  - 使用其他應用程式
  - 在首頁畫面
  - [Android] 在其他 `Activity` 中（即使是由您的應用程式啟動）
- [iOS] `inactive` - 此狀態發生在前景與背景轉換期間，以及進入多工檢視、開啟通知中心或來電等非活動期間。

更多資訊請參閱 [Apple 官方文件](https://developer.apple.com/documentation/uikit/app_and_scenes/managing_your_app_s_life_cycle)

## 基本用法

要查看當前狀態，您可以檢查 `AppState.currentState`，該值會保持最新狀態。然而，在啟動時 `currentState` 會是 null，直到 `AppState` 透過橋接器取得狀態。

```SnackPlayer name=AppState%20Example
import React, {useRef, useState, useEffect} from 'react';
import {AppState, StyleSheet, Text} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

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
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <Text>Current state is: {appStateVisible}</Text>
      </SafeAreaView>
    </SafeAreaProvider>
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

此範例永遠只會顯示「當前狀態是：active」，因為應用程式只有在 `active` 狀態時對使用者可見，而 null 狀態只會短暫出現。如果想實驗程式碼，建議使用自己的設備而非嵌入式預覽。

---

# 參考資料

## 事件

### `change`

當應用程式狀態變更時會收到此事件。監聽器會傳入[當前應用程式狀態值](appstate#app-states)之一。

### `memoryWarning`

此事件用於需要發出記憶體警告或釋放記憶體時。

### `focus` <div class="label android">Android</div>

當應用程式獲得焦點時接收（使用者正在與應用程式互動）。

### `blur` <div class="label android">Android</div>

當使用者未積極與應用程式互動時接收。適用於使用者下拉[通知抽屜](https://developer.android.com/guide/topics/ui/notifiers/notifications#bar-and-drawer)的情況。`AppState` 不會改變，但會觸發 `blur` 事件。

## 方法

### `addEventListener()`

```tsx
static addEventListener(
  type: AppStateEvent,
  listener: (state: AppStateStatus) => void,
): NativeEventSubscription;
```

設定一個函式，當 AppState 上發生指定事件類型時呼叫。`eventType` 的有效值請參閱[上述事件列表](#events)。返回 `EventSubscription`。

## 屬性

### `currentState`

```tsx
static currentState: AppStateStatus;
```