---
id: inputaccessoryview
title: InputAccessoryView
---

一個可在 iOS 上自訂鍵盤輸入附屬視圖的元件。當 `TextInput` 獲得焦點時，輸入附屬視圖會顯示於鍵盤上方。此元件可用於建立自訂工具列。

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

此元件亦可用於建立固定式文字輸入框（錨定在鍵盤頂部的文字輸入框）。操作方式是將 `TextInput` 用 `InputAccessoryView` 元件包裹，且不設定 `nativeID`。範例可參見 [InputAccessoryViewExample.js](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/examples/InputAccessoryView/InputAccessoryViewExample.js)。

---

# 參考文獻

## 屬性

### `backgroundColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `nativeID`

用於將此 `InputAccessoryView` 與指定 TextInput 關聯的識別碼。

| Type   |
| ------ |
| string |

---

### `style`

| Type                              |
| --------------------------------- |
| [View Style](view-style-props.md) |

# 已知問題

- [react-native#18997](https://github.com/facebook/react-native/issues/18997): 不支援多行 `TextInput`
- [react-native#20157](https://github.com/facebook/react-native/issues/20157): 無法與底部標籤欄共用