---
id: textinput
title: TextInput
---

一個基礎組件，用於透過鍵盤在應用程式中輸入文字。其屬性可配置多種功能，如自動校正、自動大寫、佔位文字，以及不同類型的鍵盤（例如數字鍵盤）。

最基本的用法是放置一個 `TextInput` 並訂閱 `onChangeText` 事件來讀取用戶輸入。還有其他事件如 `onSubmitEditing` 和 `onFocus` 可供訂閱。以下是一個簡單範例：

```SnackPlayer name=TextInput
import React from "react";
import { SafeAreaView, StyleSheet, TextInput } from "react-native";

const UselessTextInput = () => {
  const [text, onChangeText] = React.useState("Useless Text");
  const [number, onChangeNumber] = React.useState(null);

  return (
    <SafeAreaView>
      <TextInput
        style={styles.input}
        onChangeText={onChangeText}
        value={text}
      />
      <TextInput
        style={styles.input}
        onChangeText={onChangeNumber}
        value={number}
        placeholder="useless placeholder"
        keyboardType="numeric"
      />
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  input: {
    height: 40,
    margin: 12,
    borderWidth: 1,
    padding: 10,
  },
});

export default UselessTextInput;
```

透過原生元素暴露的兩個方法是 .focus() 和 .blur()，可用程式方式聚焦或模糊 TextInput。

Note that some props are only available with `multiline={true/false}`. Additionally, border styles that apply to only one side of the element (e.g., `borderBottomColor`, `borderLeftWidth`, etc.) will not be applied if `multiline=true`. To achieve the same effect, you can wrap your `TextInput` in a `View`:

```SnackPlayer name=TextInput
import React from 'react';
import { View, TextInput } from 'react-native';

const UselessTextInput = (props) => {
  return (
    <TextInput
      {...props} // Inherit any props passed to it; e.g., multiline, numberOfLines below
      editable
      maxLength={40}
    />
  );
}

const UselessTextInputMultiline = () => {
  const [value, onChangeText] = React.useState('Useless Multiline Placeholder');

  // If you type something in the text box that is a color, the background will change to that
  // color.
  return (
    <View
      style={{
        backgroundColor: value,
        borderBottomColor: '#000000',
        borderBottomWidth: 1,
      }}>
      <UselessTextInput
        multiline
        numberOfLines={4}
        onChangeText={text => onChangeText(text)}
        value={value}
        style={{padding: 10}}
      />
    </View>
  );
}

export default UselessTextInputMultiline;
```

`TextInput` 預設在其視圖底部有一個邊框。此邊框的內邊距由系統提供的背景圖片設定，無法更改。避免此問題的解決方案是：要麼不顯式設定高度（系統會自動正確顯示邊框），要麼透過將 `underlineColorAndroid` 設為透明來隱藏邊框。

Note that on Android performing text selection in an input can change the app's activity `windowSoftInputMode` param to `adjustResize`. This may cause issues with components that have position: 'absolute' while the keyboard is active. To avoid this behavior either specify `windowSoftInputMode` in AndroidManifest.xml ( https://developer.android.com/guide/topics/manifest/activity-element.html ) or control this param programmatically with native code.

---

# 參考

## 屬性

### [View 屬性](view.md#props)

繼承 [View 屬性](view.md#props)。

---

### `allowFontScaling`

指定字體是否應縮放以符合「文字大小」輔助設定。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `autoCapitalize`

指示 `TextInput` 自動大寫特定字元。此屬性不受某些鍵盤類型支援（如 `name-phone-pad`）。

- `characters`: 所有字元。
- `words`: 每個單詞的首字母。
- `sentences`: 每個句子的首字母（_預設_）。
- `none`: 不自動大寫任何內容。

| Type                                             |
| ------------------------------------------------ |
| enum('none', 'sentences', 'words', 'characters') |

---

### `autoComplete` <div class="label android">Android</div>

為系統指定自動完成提示，以便提供自動填充功能。在 Android 上，系統會始終嘗試透過啟發式識別內容類型來提供自動填充。要禁用自動完成，請將 `autoComplete` 設為 `off`。

Possible values for `autoComplete` are:

- `birthdate-day`
- `birthdate-full`
- `birthdate-month`
- `birthdate-year`
- `cc-csc`
- `cc-exp`
- `cc-exp-day`
- `cc-exp-month`
- `cc-exp-year`
- `cc-number`
- `email`
- `gender`
- `name`
- `name-family`
- `name-given`
- `name-middle`
- `name-middle-initial`
- `name-prefix`
- `name-suffix`
- `password`
- `password-new`
- `postal-address`
- `postal-address-country`
- `postal-address-extended`
- `postal-address-extended-postal-code`
- `postal-address-locality`
- `postal-address-region`
- `postal-code`
- `street-address`
- `sms-otp`
- `tel`
- `tel-country-code`
- `tel-national`
- `tel-device`
- `username`
- `username-new`
- `off`

| Type                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| enum('birthdate-day', 'birthdate-full', 'birthdate-month', 'birthdate-year', 'cc-csc', 'cc-exp', 'cc-exp-day', 'cc-exp-month', 'cc-exp-year', 'cc-number', 'email', 'gender', 'name', 'name-family', 'name-given', 'name-middle', 'name-middle-initial', 'name-prefix', 'name-suffix', 'password', 'password-new', 'postal-address', 'postal-address-country', 'postal-address-extended', 'postal-address-extended-postal-code', 'postal-address-locality', 'postal-address-region', 'postal-code', 'street-address', 'sms-otp', 'tel', 'tel-country-code', 'tel-national', 'tel-device', 'username', 'username-new', 'off') |

---

### `autoCorrect`

若設為 `false`，則停用自動校正功能。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `autoFocus`

若設為 `true`，則在 `componentDidMount` 或 `useEffect` 時自動聚焦輸入框。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `blurOnSubmit`

If `true`, the text field will blur when submitted. The default value is true for single-line fields and false for multiline fields. Note that for multiline fields, setting `blurOnSubmit` to `true` means that pressing return will blur the field and trigger the `onSubmitEditing` event instead of inserting a newline into the field.

| Type |
| ---- |
| bool |

---

### `caretHidden`

若設為 `true`，則隱藏游標。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `clearButtonMode` (清除按鈕模式) <div class="label ios">iOS</div>

決定清除按鈕何時出現在文字輸入框右側。此屬性僅支援單行 TextInput 元件。預設值為 `never` (從不顯示)。

| Type                                                       |
| ---------------------------------------------------------- |
| enum('never', 'while-editing', 'unless-editing', 'always') |

---

### `clearTextOnFocus` (聚焦時清除文字) <div class="label ios">iOS</div>

若設為 `true`，開始編輯時會自動清除文字框內容。

| Type |
| ---- |
| bool |

---

### `contextMenuHidden`

若設為 `true`，則隱藏上下文選單。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `dataDetectorTypes` (資料偵測類型) <div class="label ios">iOS</div>

決定文字輸入框中哪些類型的資料會被轉換為可點擊的連結。僅在 `multiline={true}` 且 `editable={false}` 時有效。預設不偵測任何資料類型。

可指定單一類型或多種類型組成的陣列。

Possible values for `dataDetectorTypes` are:

- `'phoneNumber'`
- `'link'`
- `'address'`
- `'calendarEvent'`
- `'none'`
- `'all'`

| Type                                                                                                                                                     |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| enum('phoneNumber', 'link', 'address', 'calendarEvent', 'none', 'all'), ,array of enum('phoneNumber', 'link', 'address', 'calendarEvent', 'none', 'all') |

---

### `defaultValue`

提供初始值，該值會在使用者開始輸入時改變。適用於不想處理事件監聽和更新值屬性來保持受控狀態同步的使用情境。

| Type   |
| ------ |
| string |

---

### `cursorColor` <div class="label android">Android</div>

當提供此屬性時，它會設定元件中游標（或「插入符號」）的顏色。與 `selectionColor` 的行為不同，游標顏色會獨立於文字選取框的顏色設定。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `disableFullscreenUI` <div class="label android">Android</div>

當設為 `false` 時，如果文字輸入周圍可用空間較少（例如手機的橫向模式），作業系統可能會選擇讓使用者在全螢幕文字輸入模式中編輯文字。當設為 `true` 時，此功能會被禁用，使用者將始終直接在文字輸入框中編輯文字。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `editable`

如果設為 `false`，文字將無法編輯。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `enablesReturnKeyAutomatically` <div class="label ios">iOS</div>

如果設為 `true`，鍵盤會在沒有文字時禁用返回鍵，並在有文字時自動啟用它。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `importantForAutofill` <div class="label android">Android</div>

告訴作業系統是否應將應用中的個別欄位包含在 Android API 26+ 的自動填充視圖結構中。可能的值為 `auto`、`no`、`noExcludeDescendants`、`yes` 和 `yesExcludeDescendants`。預設值為 `auto`。

- `auto`：讓 Android 系統使用其啟發式方法來判斷視圖是否對自動填充重要。
- `no`：此視圖對自動填充不重要。
- `noExcludeDescendants`：此視圖及其子視圖對自動填充不重要。
- `yes`：此視圖對自動填充重要。
- `yesExcludeDescendants`：此視圖對自動填充重要，但其子視圖不重要。

| Type                                                                       |
| -------------------------------------------------------------------------- |
| enum('auto', 'no', 'noExcludeDescendants', 'yes', 'yesExcludeDescendants') |

---

### `inlineImageLeft` <div class="label android">Android</div>

如果定義此屬性，提供的圖片資源將顯示在左側。圖片資源必須位於 `/android/app/src/main/res/drawable` 中，並以以下方式引用：

```
<TextInput
 inlineImageLeft='search_icon'
/>
```

| Type   |
| ------ |
| string |

---

### `inlineImagePadding` <div class="label android">Android</div>

內嵌圖片（如果有）與文字輸入框本身之間的間距。

| Type   |
| ------ |
| number |

---

### `inputAccessoryViewID` <div class="label ios">iOS</div>

一個可選的標識符，用於將自定義的 [InputAccessoryView](inputaccessoryview.md) 連結到此文字輸入框。當此文字輸入框獲得焦點時，InputAccessoryView 會顯示在鍵盤上方。

| Type   |
| ------ |
| string |

---

### `keyboardAppearance` <div class="label ios">iOS</div>

決定鍵盤的顏色。

| Type                             |
| -------------------------------- |
| enum('default', 'light', 'dark') |

---

### `keyboardType`

決定要開啟哪種鍵盤，例如 `numeric`。

所有類型的截圖可參見[此處](http://lefkowitz.me/2018/04/30/visual-guide-to-react-native-textinput-keyboardtype-options/)。

以下數值適用於跨平台：

- `default`
- `number-pad`
- `decimal-pad`
- `numeric`
- `email-address`
- `phone-pad`
- `url`

_iOS 專用_

以下數值僅適用於 iOS：

- `ascii-capable`
- `numbers-and-punctuation`
- `name-phone-pad`
- `twitter`
- `web-search`

_Android 專用_

以下數值僅適用於 Android：

- `visible-password`

| Type                                                                                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| enum('default', 'email-address', 'numeric', 'phone-pad', 'ascii-capable', 'numbers-and-punctuation', 'url', 'number-pad', 'name-phone-pad', 'decimal-pad', 'twitter', 'web-search', 'visible-password') |

---

### `maxFontSizeMultiplier`

指定當啟用 `allowFontScaling` 時，字體可達到的最大縮放比例。可能的值：

- `null/undefined`（預設）：繼承父節點或全域預設值（0）
- `0`：無最大值，忽略父級/全域預設
- `>= 1`：將此節點的 `maxFontSizeMultiplier` 設為此值

| Type   |
| ------ |
| number |

---

### `maxLength`

限制可輸入的最大字元數。使用此屬性代替在 JS 中實作邏輯以避免閃爍。

| Type   |
| ------ |
| number |

---

### `multiline`

若為 `true`，文字輸入可為多行。預設值為 `false`。

:::note
需注意在 iOS 上會將文字對齊頂部，而在 Android 上會置中。若要兩平台行為一致，請搭配將 `textAlignVertical` 設為 `top` 使用。
:::

| Type |
| ---- |
| bool |

---

### `numberOfLines` <div class="label android">Android</div>

Sets the number of lines for a `TextInput`. Use it with multiline set to `true` to be able to fill the lines.

| Type   |
| ------ |
| number |

---

### `onBlur`

當文字輸入失去焦點時呼叫的回調函式。

> Note: If you are attempting to access the `text` value from `nativeEvent` keep in mind that the resulting value you get can be `undefined` which can cause unintended errors. If you are trying to find the last value of TextInput, you can use the [`onEndEditing`](textinput#onendediting) event, which is fired upon completion of editing.

| Type     |
| -------- |
| function |

---

### `onChange`

當文字輸入內容變更時呼叫的回調函式。

| Type                                                     |
| -------------------------------------------------------- |
| (`{ nativeEvent: { eventCount, target, text} }`) => void |

---

### `onChangeText`

當文字輸入內容變更時呼叫的回調函式。變更後的文字會以單一字串參數傳遞給回調處理函式。

| Type     |
| -------- |
| function |

---

### `onContentSizeChange`

當文字輸入內容大小變更時呼叫的回調函式。

僅適用於多行文字輸入。

| Type                                                            |
| --------------------------------------------------------------- |
| (`{ nativeEvent: { contentSize: { width, height } } }`) => void |

---

### `onEndEditing`

當文字輸入結束時呼叫的回調函式。

| Type     |
| -------- |
| function |

---

### `onPressIn`

當觸碰開始時呼叫的回調函式。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({ nativeEvent: [PressEvent](pressevent) }) => void` |

---

### `onPressOut`

當觸碰被釋放時觸發的回調函數。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({ nativeEvent: [PressEvent](pressevent) }) => void` |

---

### `onFocus`

當文字輸入框獲得焦點時觸發的回調函數。

| Type                                                       |
| ---------------------------------------------------------- |
| `md ({ nativeEvent: [LayoutEvent](layoutevent) }) => void` |

---

### `onKeyPress`

當按鍵被按下時觸發的回調函數。會傳入一個物件，其中 `keyValue` 為 `'Enter'` 或 `'Backspace'` 對應特定按鍵，其他情況下則是輸入的字符（包括空格 `' '`）。此回調會在 `onChange` 之前觸發。注意：在 Android 上僅處理軟鍵盤輸入，不處理硬體鍵盤輸入。

| Type                                           |
| ---------------------------------------------- |
| (`{ nativeEvent: { key: keyValue } }`) => void |

---

### `onLayout`

在元件掛載或佈局變更時觸發。

| Type                                                       |
| ---------------------------------------------------------- |
| `md ({ nativeEvent: [LayoutEvent](layoutevent) }) => void` |

---

### `onScroll`

在內容滾動時觸發。可能包含來自 `ScrollEvent` 的其他屬性，但在 Android 上為效能考量不會提供 `contentSize`。

| Type                                                     |
| -------------------------------------------------------- |
| (`{ nativeEvent: { contentOffset: { x, y } } }`) => void |

---

### `onSelectionChange`

當文字輸入框的選取範圍變更時觸發的回調函數。

| Type                                                       |
| ---------------------------------------------------------- |
| (`{ nativeEvent: { selection: { start, end } } }`) => void |

---

### `onSubmitEditing`

當文字輸入框的提交按鈕被按下時觸發的回調函數。

| Type                                                     |
| -------------------------------------------------------- |
| (`{ nativeEvent: { text, eventCount, target }}`) => void |

注意：在 iOS 上使用 `keyboardType="phone-pad"` 時不會觸發此方法。

---

### `placeholder`

在文字輸入框尚未輸入內容時顯示的字串。

| Type   |
| ------ |
| string |

---

### `placeholderTextColor`

預留文字的字串顏色。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `returnKeyLabel` <div class="label android">Android</div>

設定返回鍵的標籤文字。可替代 `returnKeyType` 使用。

| Type   |
| ------ |
| string |

---

### `returnKeyType`

決定返回鍵的外觀樣式。在 Android 上也可使用 `returnKeyLabel`。

_跨平台支援_

以下數值適用於所有平台：

- `done`
- `go`
- `next`
- `search`
- `send`

_僅限 Android_

以下數值僅適用於 Android：

- `none`
- `previous`

_僅限 iOS_

以下數值僅適用於 iOS：

- `default`
- `emergency-call`
- `google`
- `join`
- `route`
- `yahoo`

| Type                                                                                                                              |
| --------------------------------------------------------------------------------------------------------------------------------- |
| enum('done', 'go', 'next', 'search', 'send', 'none', 'previous', 'default', 'emergency-call', 'google', 'join', 'route', 'yahoo') |

### `rejectResponderTermination` <div class="label ios">iOS</div>

若為 `true`，允許 TextInput 將觸碰事件傳遞給父元件。這使得如 SwipeableListView 等元件能從 TextInput 滑動（Android 預設即支援此行為）。若為 `false`，TextInput 始終要求處理輸入（除非被禁用）。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `scrollEnabled` <div class="label ios">iOS</div>

若設為 `false`，將禁用文字視窗的滾動功能。預設值為 `true`。僅在 `multiline={true}` 時有效。

| Type |
| ---- |
| bool |

---

### `secureTextEntry`

若設為 `true`，文字輸入會隱藏內容以保護敏感資訊（如密碼）。預設值為 `false`。不可與 `multiline={true}` 同時使用。

| Type |
| ---- |
| bool |

---

### `selection`

設定文字輸入選取範圍的起始與結束位置。將起始與結束設為相同值可定位游標。

| Type                                  |
| ------------------------------------- |
| object: `{start: number,end: number}` |

---

### `selectionColor`

文字輸入的高亮與游標顏色。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `selectTextOnFocus`

若設為 `true`，聚焦時將自動選取所有文字。

| Type |
| ---- |
| bool |

---

### `showSoftInputOnFocus`

設為 `false` 可防止聚焦時觸發軟鍵盤彈出。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `spellCheck` <div class="label ios">iOS</div>

若設為 `false`，將停用拼字檢查樣式（如紅色底線）。預設值繼承自 `autoCorrect`。

| Type |
| ---- |
| bool |

---

### `textAlign`

將輸入文字對齊輸入欄位的左側、居中或右側。

Possible values for `textAlign` are:

- `left`
- `center`
- `right`

| Type                            |
| ------------------------------- |
| enum('left', 'center', 'right') |

---

### `textContentType` <div class="label ios">iOS</div>

向鍵盤與系統提供預期輸入內容的語義資訊。

For iOS 11+ you can set `textContentType` to `username` or `password` to enable autofill of login details from the device keychain.

在 iOS 12+ 中，`newPassword` 表示使用者可能想儲存至鑰匙圈的新密碼輸入欄位，`oneTimeCode` 表示可透過簡訊驗證碼自動填入的欄位。

To disable autofill, set `textContentType` to `none`.

Possible values for `textContentType` are:

- `none`
- `URL`
- `addressCity`
- `addressCityAndState`
- `addressState`
- `countryName`
- `creditCardNumber`
- `emailAddress`
- `familyName`
- `fullStreetAddress`
- `givenName`
- `jobTitle`
- `location`
- `middleName`
- `name`
- `namePrefix`
- `nameSuffix`
- `nickname`
- `organizationName`
- `postalCode`
- `streetAddressLine1`
- `streetAddressLine2`
- `sublocality`
- `telephoneNumber`
- `username`
- `password`
- `newPassword`
- `oneTimeCode`

| Type                                                                                                                                                                                                                                                                                                                                                                                                       |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| enum('none', 'URL', 'addressCity', 'addressCityAndState', 'addressState', 'countryName', 'creditCardNumber', 'emailAddress', 'familyName', 'fullStreetAddress', 'givenName', 'jobTitle', 'location', 'middleName', 'name', 'namePrefix', 'nameSuffix', 'nickname', 'organizationName', 'postalCode', 'streetAddressLine1', 'streetAddressLine2', 'sublocality', 'telephoneNumber', 'username', 'password') |

---

### `passwordRules` <div class="label ios">iOS</div>

在 iOS 上使用 `textContentType` 設為 `newPassword` 時，我們可以讓作業系統知道密碼的最低要求，以便生成符合這些要求的密碼。要建立有效的 `PasswordRules` 字串，請參考 [Apple 官方文件](https://developer.apple.com/password-rules/)。

> 如果密碼生成對話框未出現，請確認以下事項：
>
> - 已啟用自動填充功能：**設定** → **密碼與帳號** → 開啟 **自動填充密碼**，
> - 使用 iCloud 鑰匙圈：**設定** → **Apple ID** → **iCloud** → **鑰匙圈** → 開啟 **iCloud 鑰匙圈**。

| Type   |
| ------ |
| string |

---

### `style`

請注意，並非所有文字樣式都受支援，以下是不支援的樣式部分列表：

- `borderLeftWidth`
- `borderTopWidth`
- `borderRightWidth`
- `borderBottomWidth`
- `borderTopLeftRadius`
- `borderTopRightRadius`
- `borderBottomRightRadius`
- `borderBottomLeftRadius`

詳情請參閱 [Issue#7070](https://github.com/facebook/react-native/issues/7070)。

[樣式](style.md)

| Type                  |
| --------------------- |
| [Text](text.md#style) |

---

### `textBreakStrategy` <div class="label android">Android</div>

在 Android API Level 23+ 上設定文字斷行策略，可能的值為 `simple`、`highQuality`、`balanced`，預設值為 `highQuality`。

| Type                                      |
| ----------------------------------------- |
| enum('simple', 'highQuality', 'balanced') |

---

### `underlineColorAndroid` <div class="label android">Android</div>

The color of the `TextInput` underline.

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `value`

文字輸入框顯示的值。`TextInput` 是一個受控元件，這意味著如果提供了此屬性，原生值將被強制匹配此值。在大多數情況下，這運作良好，但在某些情況下可能會導致閃爍——常見原因之一是通過保持值不變來防止編輯。除了設定相同的值外，還可以設定 `editable={false}`，或設定/更新 `maxLength` 以防止不必要的編輯而不產生閃爍。

| Type   |
| ------ |
| string |

## 方法

### `.focus()`

```jsx
focus();
```

讓原生輸入框請求焦點。

### `.blur()`

```jsx
blur();
```

讓原生輸入框失去焦點。

### `clear()`

```jsx
clear();
```

清除 `TextInput` 中的所有文字。

---

### `isFocused()`

```jsx
isFocused();
```

如果輸入框當前擁有焦點則返回 `true`；否則返回 `false`。

# 已知問題

- [react-native#19096](https://github.com/facebook/react-native/issues/19096): Doesn't support Android's `onKeyPreIme`.
- [react-native#19366](https://github.com/facebook/react-native/issues/19366): Calling .focus() after closing Android's keyboard via back button doesn't bring keyboard up again.
- [react-native#26799](https://github.com/facebook/react-native/issues/26799): Doesn't support Android's `secureTextEntry` when `keyboardType="email-address"` or `keyboardType="phone-pad"`.