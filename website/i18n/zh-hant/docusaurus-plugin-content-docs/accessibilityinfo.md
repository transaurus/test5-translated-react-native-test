---
id: accessibilityinfo
title: AccessibilityInfo
---

有時了解裝置目前是否有啟用螢幕閱讀器相當實用。`AccessibilityInfo` API 正是為此目的設計，可用於查詢螢幕閱讀器的當前狀態，並註冊在狀態變更時接收通知。

## 範例

```SnackPlayer name=AccessibilityInfo%20Example&supportedPlatforms=android,ios
import React, {useState, useEffect} from 'react';
import {AccessibilityInfo, Text, StyleSheet} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  const [reduceMotionEnabled, setReduceMotionEnabled] = useState(false);
  const [screenReaderEnabled, setScreenReaderEnabled] = useState(false);

  useEffect(() => {
    const reduceMotionChangedSubscription = AccessibilityInfo.addEventListener(
      'reduceMotionChanged',
      isReduceMotionEnabled => {
        setReduceMotionEnabled(isReduceMotionEnabled);
      },
    );
    const screenReaderChangedSubscription = AccessibilityInfo.addEventListener(
      'screenReaderChanged',
      isScreenReaderEnabled => {
        setScreenReaderEnabled(isScreenReaderEnabled);
      },
    );

    AccessibilityInfo.isReduceMotionEnabled().then(isReduceMotionEnabled => {
      setReduceMotionEnabled(isReduceMotionEnabled);
    });
    AccessibilityInfo.isScreenReaderEnabled().then(isScreenReaderEnabled => {
      setScreenReaderEnabled(isScreenReaderEnabled);
    });

    return () => {
      reduceMotionChangedSubscription.remove();
      screenReaderChangedSubscription.remove();
    };
  }, []);

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <Text style={styles.status}>
          The reduce motion is {reduceMotionEnabled ? 'enabled' : 'disabled'}.
        </Text>
        <Text style={styles.status}>
          The screen reader is {screenReaderEnabled ? 'enabled' : 'disabled'}.
        </Text>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  status: {
    margin: 30,
  },
});

export default App;
```

---

# 參考文獻

## 方法

### `addEventListener()`

```tsx
static addEventListener(
  eventName: AccessibilityChangeEventName | AccessibilityAnnouncementEventName,
  handler: (
    event: AccessibilityChangeEvent | AccessibilityAnnouncementFinishedEvent,
  ) => void,
): EmitterSubscription;
```

新增事件處理函式。支援的事件類型：

| Event name                                                                           | Description                                                                                                                                                                                                                                                                                              |
| ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `accessibilityServiceChanged`<br/><div class="label two-lines android">Android</div> | Fires when some services such as TalkBack, other Android assistive technologies, and third-party accessibility services are enabled. The argument to the event handler is a boolean. The boolean is `true` when a some accessibility services is enabled and `false` otherwise.                          |
| `announcementFinished`<br/><div class="label two-lines ios">iOS</div>                | Fires when the screen reader has finished making an announcement. The argument to the event handler is a dictionary with these keys:<ul><li>`announcement`: The string announced by the screen reader.</li><li>`success`: A boolean indicating whether the announcement was successfully made.</li></ul> |
| `boldTextChanged`<br/><div class="label two-lines ios">iOS</div>                     | Fires when the state of the bold text toggle changes. The argument to the event handler is a boolean. The boolean is `true` when bold text is enabled and `false` otherwise.                                                                                                                             |
| `grayscaleChanged`<br/><div class="label two-lines ios">iOS</div>                    | Fires when the state of the gray scale toggle changes. The argument to the event handler is a boolean. The boolean is `true` when a gray scale is enabled and `false` otherwise.                                                                                                                         |
| `invertColorsChanged`<br/><div class="label two-lines ios">iOS</div>                 | Fires when the state of the invert colors toggle changes. The argument to the event handler is a boolean. The boolean is `true` when invert colors is enabled and `false` otherwise.                                                                                                                     |
| `reduceMotionChanged`                                                                | Fires when the state of the reduce motion toggle changes. The argument to the event handler is a boolean. The boolean is `true` when a reduce motion is enabled (or when "Transition Animation Scale" in "Developer options" is "Animation off") and `false` otherwise.                                  |
| `reduceTransparencyChanged`<br/><div class="label two-lines ios">iOS</div>           | Fires when the state of the reduce transparency toggle changes. The argument to the event handler is a boolean. The boolean is `true` when reduce transparency is enabled and `false` otherwise.                                                                                                         |
| `screenReaderChanged`                                                                | Fires when the state of the screen reader changes. The argument to the event handler is a boolean. The boolean is `true` when a screen reader is enabled and `false` otherwise.                                                                                                                          |

---

### `announceForAccessibility()`

```tsx
static announceForAccessibility(announcement: string);
```

發佈字串供螢幕閱讀器朗讀。

---

### `announceForAccessibilityWithOptions()`

```tsx
static announceForAccessibilityWithOptions(
  announcement: string,
  options: options: {queue?: boolean},
);
```

發佈字串供螢幕閱讀器朗讀，並可設定選項調整行為。預設會中斷當前語音，但在 iOS 上可透過選項物件設定 `queue` 為 `true` 來將朗讀排隊於現有語音之後。

**參數：**

| Name                                                          | Type   | Description                                                                              |
| ------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------------------- |
| announcement <div class="label basic required">Required</div> | string | The string to be announced                                                               |
| options <div class="label basic required">Required</div>      | object | `queue` - queue the announcement behind existing speech <div class="label ios">iOS</div> |

---

### `getRecommendedTimeoutMillis()` <div class="label android">Android</div>

```tsx
static getRecommendedTimeoutMillis(originalTimeout: number): Promise<number>;
```

取得使用者所需的逾時毫秒數。
此數值由「無障礙設定」中的「操作時間限制（無障礙逾時）」設定決定。

**參數：**

| Name                                                             | Type   | Description                                                                           |
| ---------------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------- |
| originalTimeout <div class="label basic required">Required</div> | number | The timeout to return if "Accessibility timeout" is not set. Specify in milliseconds. |

---

### `isAccessibilityServiceEnabled()` <div class="label android">Android</div>

```tsx
static isAccessibilityServiceEnabled(): Promise<boolean>;
```

檢查是否有啟用任何無障礙服務。包含 TalkBack 及可能安裝的第三方無障礙應用程式。若僅需檢查 TalkBack 狀態，請使用 [isScreenReaderEnabled](#isscreenreaderenabled)。回傳 Promise 並解析為布林值，當有無障礙服務啟用時結果為 `true`，反之為 `false`。

> **注意**：若僅需檢查 TalkBack 狀態，請使用 [isScreenReaderEnabled](#isscreenreaderenabled)。

---

### `isBoldTextEnabled()` <div class="label ios">iOS</div>

```tsx
static isBoldTextEnabled(): Promise<boolean>:
```

查詢粗體文字功能是否啟用。回傳 Promise 並解析為布林值，啟用時結果為 `true`，反之為 `false`。

---

### `isGrayscaleEnabled()` <div class="label ios">iOS</div>

```tsx
static isGrayscaleEnabled(): Promise<boolean>;
```

查詢灰階模式是否啟用。回傳 Promise 並解析為布林值，啟用時結果為 `true`，反之為 `false`。

---

### `isInvertColorsEnabled()` <div class="label ios">iOS</div>

```tsx
static isInvertColorsEnabled(): Promise<boolean>;
```

查詢反轉顏色功能是否啟用。回傳 Promise 並解析為布林值，啟用時結果為 `true`，反之為 `false`。

---

### `isReduceMotionEnabled()`

```tsx
static isReduceMotionEnabled(): Promise<boolean>;
```

查詢減少動畫效果是否啟用。回傳 Promise 並解析為布林值，啟用時結果為 `true`，反之為 `false`。

---

### `isReduceTransparencyEnabled()` <div class="label ios">iOS</div>

```tsx
static isReduceTransparencyEnabled(): Promise<boolean>;
```

查詢是否當前啟用了降低透明度功能。返回一個解析為布林值的 Promise。當降低透明度功能啟用時結果為 `true`，否則為 `false`。

---

### `isScreenReaderEnabled()`

```tsx
static isScreenReaderEnabled(): Promise<boolean>;
```

查詢是否當前啟用了螢幕閱讀器。返回一個解析為布林值的 Promise。當螢幕閱讀器啟用時結果為 `true`，否則為 `false`。

---

### `prefersCrossFadeTransitions()` <div class="label ios">iOS</div>

```tsx
static prefersCrossFadeTransitions(): Promise<boolean>;
```

查詢是否當前啟用了降低動畫效果並偏好使用淡入淡出過渡的設定。返回一個解析為布林值的 Promise。當偏好淡入淡出過渡功能啟用時結果為 `true`，否則為 `false`。

---

### `setAccessibilityFocus()`

```tsx
static setAccessibilityFocus(reactTag: number);
```

將無障礙焦點設置到 React 元件上。

在 Android 上，這會調用 `UIManager.sendAccessibilityEvent` 方法，並傳遞 `reactTag` 和 `UIManager.AccessibilityEventTypes.typeViewFocused` 參數。

> **注意**：確保任何要接收無障礙焦點的 `View` 都設置了 `accessible={true}`。