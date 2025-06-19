---
id: actionsheetios
title: ActionSheetIOS
---

顯示 iOS 原生的 [動作選單](https://developer.apple.com/design/human-interface-guidelines/ios/views/action-sheets/) 元件。

## 範例

```SnackPlayer name=ActionSheetIOS&supportedPlatforms=ios
import React, {useState} from 'react';
import {ActionSheetIOS, Button, StyleSheet, Text, View} from 'react-native';

const App = () => {
  const [result, setResult] = useState('🔮');

  const onPress = () =>
    ActionSheetIOS.showActionSheetWithOptions(
      {
        options: ['Cancel', 'Generate number', 'Reset'],
        destructiveButtonIndex: 2,
        cancelButtonIndex: 0,
        userInterfaceStyle: 'dark',
      },
      buttonIndex => {
        if (buttonIndex === 0) {
          // cancel action
        } else if (buttonIndex === 1) {
          setResult(String(Math.floor(Math.random() * 100) + 1));
        } else if (buttonIndex === 2) {
          setResult('🔮');
        }
      },
    );

  return (
    <View style={styles.container}>
      <Text style={styles.result}>{result}</Text>
      <Button onPress={onPress} title="Show Action Sheet" />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  result: {
    fontSize: 64,
    textAlign: 'center',
  },
});

export default App;
```

# 參考文件

## 方法

### `showActionSheetWithOptions()`

```tsx
static showActionSheetWithOptions: (
  options: ActionSheetIOSOptions,
  callback: (buttonIndex: number) => void,
);
```

顯示 iOS 動作選單。`options` 物件必須包含以下一或多個屬性：

- `options` (array of strings) - a list of button titles (required)
- `cancelButtonIndex` (int) - index of cancel button in `options`
- `cancelButtonTintColor` (string) - the [color](colors) used for the change the text color of the cancel button
- `destructiveButtonIndex` (int or array of ints) - indices of destructive buttons in `options`
- `title` (string) - a title to show above the action sheet
- `message` (string) - a message to show below the title
- `anchor` (number) - the node to which the action sheet should be anchored (used for iPad)
- `tintColor` (string) - the [color](colors) used for non-destructive button titles
- `disabledButtonIndices` (array of numbers) - a list of button indices which should be disabled
- `userInterfaceStyle` (string) - the interface style used for the action sheet, can be set to `light` or `dark`, otherwise the default system style will be used

'callback' 函式接受一個參數，即所選項目的從零開始的索引。

最小範例：

```tsx
ActionSheetIOS.showActionSheetWithOptions(
  {
    options: ['Cancel', 'Remove'],
    destructiveButtonIndex: 1,
    cancelButtonIndex: 0,
  },
  buttonIndex => {
    if (buttonIndex === 1) {
      /* destructive action */
    }
  },
);
```

---

### `dismissActionSheet()`

```tsx
static dismissActionSheet();
```

關閉最上層的 iOS 動作選單，如果沒有動作選單顯示則會顯示警告。

---

### `showShareActionSheetWithOptions()`

```tsx
static showShareActionSheetWithOptions: (
  options: ShareActionSheetIOSOptions,
  failureCallback: (error: Error) => void,
  successCallback: (success: boolean, method: string) => void,
);
```

顯示 iOS 分享選單。`options` 物件應包含 `message` 和/或 `url`，並可額外包含 `subject` 或 `excludedActivityTypes`：

- `url` (字串) - 要分享的 URL
- `message` (字串) - 要分享的訊息
- `subject` (字串) - 訊息的主旨
- `excludedActivityTypes` (陣列) - 要從動作選單中排除的活動類型

> **注意：** 如果 `url` 指向本地檔案或是 base64 編碼的 uri，則會直接載入並分享該檔案。透過這種方式，您可以分享圖片、影片、PDF 檔案等。如果 `url` 指向遠端檔案或地址，則必須符合 [RFC 2396](https://www.ietf.org/rfc/rfc2396.txt) 中描述的 URL 格式。例如，沒有正確協定 (HTTP/HTTPS) 的網頁 URL 將無法分享。

The 'failureCallback' function takes one parameter, an error object. The only property defined on this object is an optional `stack` property of type `string`.

'successCallback' 函式接受兩個參數：

- 表示成功或失敗的布林值
- 在成功情況下，表示分享方式的字串