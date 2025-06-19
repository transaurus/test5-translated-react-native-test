---
id: inputaccessoryview
title: InputAccessoryView
---

此元件可讓您自訂 iOS 鍵盤上方的輸入輔助視圖。當 `TextInput` 獲得焦點時，輸入輔助視圖會顯示在鍵盤上方。此元件可用來建立自訂工具列。

To use this component wrap your custom toolbar with the InputAccessoryView component, and set a `nativeID`. Then, pass that `nativeID` as the `inputAccessoryViewID` of whatever `TextInput` you desire. A basic example:

```SnackPlayer name=InputAccessoryView&supportedPlatforms=ios
import React, {useState} from 'react';
import {
  Button,
  InputAccessoryView,
  ScrollView,
  TextInput,
  StyleSheet,
} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const inputAccessoryViewID = 'uniqueID';
const initialText = '';

const App = () => {
  const [text, setText] = useState(initialText);

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <ScrollView keyboardDismissMode="interactive">
          <TextInput
            style={styles.textInput}
            inputAccessoryViewID={inputAccessoryViewID}
            onChangeText={setText}
            value={text}
            placeholder={'Please type here…'}
          />
        </ScrollView>
      </SafeAreaView>
      <InputAccessoryView nativeID={inputAccessoryViewID}>
        <Button onPress={() => setText(initialText)} title="Clear text" />
      </InputAccessoryView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    paddingHorizontal: 20,
  },
  textInput: {
    padding: 16,
    borderColor: 'black',
    borderWidth: 1,
  },
});

export default App;
```

This component can also be used to create sticky text inputs (text inputs which are anchored to the top of the keyboard). To do this, wrap a `TextInput` with the `InputAccessoryView` component, and don't set a `nativeID`. For an example, look at [InputAccessoryViewExample.js](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/examples/InputAccessoryView/InputAccessoryViewExample.js).

---

# 參考文件

## 屬性

### `backgroundColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `nativeID`

用於將此 `InputAccessoryView` 與指定 TextInput 關聯的 ID。

| Type   |
| ------ |
| string |

---

### `style`

| Type                              |
| --------------------------------- |
| [View Style](view-style-props.md) |

# 已知問題

- [react-native#18997](https://github.com/facebook/react-native/issues/18997)：不支援多行 `TextInput`
- [react-native#20157](https://github.com/facebook/react-native/issues/20157)：無法與底部標籤欄同時使用