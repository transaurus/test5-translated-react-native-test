---
id: segmentedcontrolios
title: '🚧 SegmentedControlIOS'
---

> **已棄用。** 請改用 [社群套件](https://reactnative.directory/?search=segmentedcontrol) 中的替代方案。

使用 `SegmentedControlIOS` 來渲染 iOS 的 UISegmentedControl。

#### 以程式方式變更選取索引

可以透過將 selectedIndex 屬性賦值給一個狀態變數，然後更改該變數來動態變更選取的索引。請注意，當用戶選擇一個值並變更索引時，需要更新狀態變數，如下例所示。

## 範例

```SnackPlayer name=SegmentedControlIOS%20Example&supportedPlatforms=ios
import React, { useState } from "react";
import { SegmentedControlIOS, StyleSheet, Text, View } from "react-native";

export default App = () => {
  const [index, setIndex] = useState(0);
  return (
    <View style={styles.container}>
      <SegmentedControlIOS
        values={['One', 'Two']}
        selectedIndex={index}
        onChange={(event) => {
          setIndex(event.nativeEvent.selectedSegmentIndex);
        }}
      />
      <Text style={styles.text}>
        Selected index: {index}
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
    justifyContent: "center"
  },
  text: {
    marginTop: 24
  }
});
```

---

# 參考

## 屬性

繼承 [View 屬性](view.md#props)。

### `enabled`

若為 false，用戶將無法與控制項互動。預設值為 true。

| Type | Required |
| ---- | -------- |
| bool | No       |

---

### `momentary`

若為 true，則選擇分段時不會持續顯示視覺效果。`onValueChange` 回調仍會如預期工作。

| Type | Required |
| ---- | -------- |
| bool | No       |

---

### `onChange`

當用戶點擊分段時觸發的回調函數；將事件作為參數傳遞

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `onValueChange`

當用戶點擊分段時觸發的回調函數；將分段的值作為參數傳遞

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `selectedIndex`

The index in `props.values` of the segment to be (pre)selected.

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `tintColor`

> **注意：** iOS 13+ 不支援 `tintColor`。

控制項的強調色。

| Type   | Required |
| ------ | -------- |
| string | No       |

---

### `values`

控制項分段按鈕的標籤，按順序排列。

| Type            | Required |
| --------------- | -------- |
| array of string | No       |