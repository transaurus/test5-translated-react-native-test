---
id: inputaccessoryview
title: InputAccessoryView
---

一個可在 iOS 上自訂鍵盤輸入附屬視圖的元件。當 `TextInput` 獲得焦點時，輸入附屬視圖會顯示在鍵盤上方。此元件可用於建立自訂工具列。

使用此元件時，請將您的自訂工具列用 InputAccessoryView 元件包裹，並設置一個 `nativeID`。然後，將該 `nativeID` 作為 `inputAccessoryViewID` 傳遞給您所需的任何 `TextInput`。基本範例：

```SnackPlayer name=InputAccessoryView&supportedPlatforms=ios
import React, { useState } from 'react';
import { Button, InputAccessoryView, ScrollView, TextInput } from 'react-native';

export default App = () => {
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
        <Button
          onPress={() => setText(initialText)}
          title="Clear text"
        />
      </InputAccessoryView>
    </>
  );
}
```

This component can also be used to create sticky text inputs (text inputs which are anchored to the top of the keyboard). To do this, wrap a `TextInput` with the `InputAccessoryView` component, and don't set a `nativeID`. For an example, look at [InputAccessoryViewExample.js](https://github.com/facebook/react-native/blob/0.70-stable/packages/rn-tester/js/examples/InputAccessoryView/InputAccessoryViewExample.js).

---

# 參考

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

- [react-native#18997](https://github.com/facebook/react-native/issues/18997): 不支援多行 `TextInput`
- [react-native#20157](https://github.com/facebook/react-native/issues/20157): 無法與底部標籤欄一起使用