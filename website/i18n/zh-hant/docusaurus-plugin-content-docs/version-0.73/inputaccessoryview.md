---
id: inputaccessoryview
title: InputAccessoryView
---

一個可在 iOS 上自訂鍵盤輸入附屬視圖的元件。當 `TextInput` 獲得焦點時，輸入附屬視圖會顯示於鍵盤上方。此元件可用於建立自訂工具列。

To use this component wrap your custom toolbar with the InputAccessoryView component, and set a `nativeID`. Then, pass that `nativeID` as the `inputAccessoryViewID` of whatever `TextInput` you desire. A basic example:

```SnackPlayer name=InputAccessoryView&supportedPlatforms=ios
import React, {useState} from 'react';
import {Button, InputAccessoryView, ScrollView, TextInput} from 'react-native';

const App = () => {
  const inputAccessoryViewID = 'uniqueID';
  const initialText = '';
  const [text, setText] = useState(initialText);

  return (
    <>
      <ScrollView keyboardDismissMode="interactive">
        <TextInput
          style={{
            padding: 16,
            marginTop: 50,
          }}
          inputAccessoryViewID={inputAccessoryViewID}
          onChangeText={setText}
          value={text}
          placeholder={'Please type here…'}
        />
      </ScrollView>
      <InputAccessoryView nativeID={inputAccessoryViewID}>
        <Button onPress={() => setText(initialText)} title="Clear text" />
      </InputAccessoryView>
    </>
  );
};

export default App;
```

This component can also be used to create sticky text inputs (text inputs which are anchored to the top of the keyboard). To do this, wrap a `TextInput` with the `InputAccessoryView` component, and don't set a `nativeID`. For an example, look at [InputAccessoryViewExample.js](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/examples/InputAccessoryView/InputAccessoryViewExample.js).

---

# 參考文獻

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
- [react-native#20157](https://github.com/facebook/react-native/issues/20157)：無法與底部標籤欄共用