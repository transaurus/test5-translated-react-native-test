---
id: i18nmanager
title: I18nManager
---

# I18nManager

The `I18nManager` module provides utilities for managing Right-to-Left (RTL) layout support for languages like Arabic, Hebrew, and others. It provides methods to control RTL behavior and check the current layout direction.

## 範例

### 根據 RTL 調整位置與動畫

若您使用絕對定位使元素與其他 flexbox 元素對齊，在 RTL 語言中可能無法正確對齊。可使用 `isRTL` 來調整對齊方式或動畫效果。

```SnackPlayer name=I18nManager%20Change%20Absolute%20Positions%20And%20Animations
import React from 'react';
import {I18nManager, Text, View} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  // Change to `true` to see the effect in a non-RTL language
  const isRTL = I18nManager.isRTL;
  return (
    <SafeAreaProvider>
      <SafeAreaView>
        <View style={{ position: 'absolute', left: isRTL ? undefined : 0, right: isRTL ? 0 : undefined }}>
          {isRTL ? (
            <Text>Back &gt;</Text>
          ) : (
            <Text>&lt; Back</Text>
          )}
        </View>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

export default App;
```

### 開發期間

```SnackPlayer name=I18nManager%20During%20Development
import React, {useState} from 'react';
import {Alert, I18nManager, StyleSheet, Switch, Text, View} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  const [rtl, setRTL] = useState(I18nManager.isRTL);
  return (
    <SafeAreaProvider>
      <SafeAreaView>
        <View style={styles.container}>
          <View style={styles.forceRtl}>
            <Text>Force RTL in Development:</Text>
            <Switch value={rtl} onValueChange={(value) => {
              setRTL(value);
              I18nManager.forceRTL(value);
              Alert.alert(
                'Reload this page',
                'Please reload this page to change the UI direction! ' +
                  'All examples in this app will be affected. ' +
                  'Check them out to see what they look like in RTL layout.',
              );
            }} />
          </View>
        </View>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 20,
  },
  forceRtl: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },
});

export default App;
```

# 參考文獻

## 屬性

### `isRTL`

```typescript
static isRTL: boolean;
```

布林值，表示應用程式當前是否處於 RTL 佈局模式。

The value of `isRTL` is determined by the following logic:

- If `forceRTL` is `true`, `isRTL` returns `true`
- If `allowRTL` is `false`, `isRTL` returns `false`
- Otherwise, `isRTL` will be `true` given the following:
  - **iOS:**
    - The user-preferred language on the device is an RTL language
    - The application-defined localizations include the user-chosen language (as defined in the Xcode project file (`knownRegions = (...)`)
  - **Android:**
    - The user-preferred language on the device is an RTL language
    - The application's `AndroidManifest.xml` defines `android:supportsRTL="true"` on the `<application>` element

### `doLeftAndRightSwapInRTL`

```typescript
static doLeftAndRightSwapInRTL: boolean;
```

布林值，表示在 RTL 模式下是否應自動交換左右樣式屬性。啟用時，在 RTL 佈局中 left 會變為 right，right 會變為 left。

## 方法

### `allowRTL()`

```typescript
static allowRTL: (allowRTL: boolean) => void;
```

啟用或禁用應用程式的 RTL 佈局支援。

**參數：**

- `allowRTL` (布林值)：是否允許 RTL 佈局

**重要注意事項：**

- 變更將在下次應用程式啟動時生效，而非立即
- 此設定會跨應用程式重啟持續保存

### `forceRTL()`

```typescript
static forceRTL: (forced: boolean) => void;
```

強制應用程式使用 RTL 佈局，無論裝置語言設定為何。主要用於開發期間測試 RTL 佈局。

避免在生產環境應用程式中強制使用 RTL，因為這需要完整重啟應用程式才能生效，會導致不佳的使用者體驗。

**參數：**

- `forced` (布林值)：是否強制使用 RTL 佈局

**重要注意事項：**

- 變更將在下次應用程式啟動時完全生效，而非立即
- 設定會跨應用程式重啟持續保存
- 僅適用於開發與測試。在生產環境中，您應完全禁用 RTL 或妥善處理（參見 `isRTL`）

### `swapLeftAndRightInRTL()`

```typescript
static swapLeftAndRightInRTL: (swapLeftAndRight: boolean) => void;
```

在 RTL 模式下交換左右樣式屬性。啟用時，在 RTL 佈局中 left 會變為 right，right 會變為 left。不影響 `isRTL` 的值。