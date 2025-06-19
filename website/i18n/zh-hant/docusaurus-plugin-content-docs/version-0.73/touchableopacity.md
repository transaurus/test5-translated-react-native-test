---
id: touchableopacity
title: TouchableOpacity
---

> 如果您正在尋找更全面且面向未來的觸控輸入處理方式，請參閱 [Pressable](pressable.md) API。

一個用於使視圖正確響應觸控操作的包裝組件。按下時，被包裝視圖的不透明度會降低，使其變暗。

透明度是通過將子組件包裹在 `Animated.View` 中來控制的，該組件會被添加到視圖層級中。請注意這可能會影響佈局。

## 範例

```SnackPlayer name=TouchableOpacity%20Example
import React, {useState} from 'react';
import {StyleSheet, Text, TouchableOpacity, View} from 'react-native';

const App = () => {
  const [count, setCount] = useState(0);
  const onPress = () => setCount(prevCount => prevCount + 1);

  return (
    <View style={styles.container}>
      <View style={styles.countContainer}>
        <Text>Count: {count}</Text>
      </View>
      <TouchableOpacity style={styles.button} onPress={onPress}>
        <Text>Press Here</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    paddingHorizontal: 10,
  },
  button: {
    alignItems: 'center',
    backgroundColor: '#DDDDDD',
    padding: 10,
  },
  countContainer: {
    alignItems: 'center',
    padding: 10,
  },
});

export default App;
```

---

# 參考文檔

## 屬性

### [TouchableWithoutFeedback 屬性](touchablewithoutfeedback.md#props)

繼承 [TouchableWithoutFeedback 屬性](touchablewithoutfeedback.md#props)。

---

### `style`

| Type                           |
| ------------------------------ |
| [View.style](view-style-props) |

---

### `activeOpacity`

決定觸控激活時被包裝視圖的不透明度值。默認為 `0.2`。

| Type   |
| ------ |
| number |

---

### `tvParallaxProperties` <div class="label ios">IOS</div>

_(僅限 Apple TV)_ 用於控制 Apple TV 視差效果的物件。

- `enabled`: 若為 `true` 則啟用視差效果。默認為 `true`。
- `shiftDistanceX`: 默認為 `2.0`。
- `shiftDistanceY`: 默認為 `2.0`。
- `tiltAngle`: 默認為 `0.05`。
- `magnification`: 默認為 `1.0`。
- `pressMagnification`: 默認為 `1.0`。
- `pressDuration`: 默認為 `0.3`。
- `pressDelay`: 默認為 `0.0`。

| Type   |
| ------ |
| object |

---

### `hasTVPreferredFocus` <div class="label ios">iOS</div>

_(僅限 Apple TV)_ TV 首選聚焦設定（詳見 View 組件的文檔）。

| Type |
| ---- |
| bool |

---

### `nextFocusDown` <div class="label android">Android</div>

TV 向下聚焦設定（詳見 View 組件的文檔）。

| Type   |
| ------ |
| number |

---

### `nextFocusForward` <div class="label android">Android</div>

TV 向前聚焦設定（詳見 View 組件的文檔）。

| Type   |
| ------ |
| number |

---

### `nextFocusLeft` <div class="label android">Android</div>

TV 向左聚焦設定（詳見 View 組件的文檔）。

| Type   |
| ------ |
| number |

---

### `nextFocusRight` <div class="label android">Android</div>

TV 向右聚焦設定（詳見 View 組件的文檔）。

| Type   |
| ------ |
| number |

---

### `nextFocusUp` <div class="label android">Android</div>

TV 向上聚焦設定（詳見 View 組件的文檔）。

| Type   |
| ------ |
| number |