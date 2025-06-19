---
id: checkbox
title: '🚧 CheckBox'
---

> **已移除。** 請改用其中一個[社群套件](https://reactnative.directory/?search=checkbox)。

渲染一個布林值輸入元件（僅限 Android）。

這是一個受控元件，需要透過 `onValueChange` 回調函數來更新 `value` 屬性，以反映使用者操作。若未更新 `value` 屬性，元件將持續顯示初始設定的 `value` 值，而非使用者操作後的預期結果。

## 範例

```SnackPlayer name=CheckBox%20Component%20Example&supportedPlatforms=android,web&ext=js
import React, {useState} from 'react';
import {CheckBox, Text, StyleSheet, View} from 'react-native';

const App = () => {
  const [isSelected, setSelection] = useState(false);

  return (
    <View style={styles.container}>
      <View style={styles.checkboxContainer}>
        <CheckBox
          value={isSelected}
          onValueChange={setSelection}
          style={styles.checkbox}
        />
        <Text style={styles.label}>Do you like React Native?</Text>
      </View>
      <Text>Is CheckBox selected: {isSelected ? '👍' : '👎'}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  checkboxContainer: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  checkbox: {
    alignSelf: 'center',
  },
  label: {
    margin: 8,
  },
});

export default App;
```

---

# 參考

## 屬性

繼承 [View 屬性](view#props)。

---

### `disabled`

If true the user won't be able to toggle the checkbox. Default value is `false`.

| Type | Required |
| ---- | -------- |
| bool | No       |

---

### `onChange`

當屬性變動導致元件被移除時使用。

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `onValueChange`

當值變更時，以新值觸發此回調函數。

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `testID`

用於在端對端測試中定位此視圖。

| Type   | Required |
| ------ | -------- |
| string | No       |

---

### `value`

The value of the checkbox. If true the checkbox will be turned on. Default value is `false`.

| Type | Required |
| ---- | -------- |
| bool | No       |