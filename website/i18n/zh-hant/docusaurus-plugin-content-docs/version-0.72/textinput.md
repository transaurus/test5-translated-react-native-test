---
id: textinput
title: TextInput
---

一個基礎組件，用於通過鍵盤向應用程式輸入文本。屬性提供了多種功能的可配置性，例如自動校正、自動大寫、佔位文本以及不同的鍵盤類型（如數字鍵盤）。

最基本的用例是放置一個 `TextInput` 並訂閱 `onChangeText` 事件來讀取用戶輸入。還有其他事件，如 `onSubmitEditing` 和 `onFocus`，也可以訂閱。一個最小示例：

```SnackPlayer name=TextInput
import React from 'react';
import {SafeAreaView, StyleSheet, TextInput} from 'react-native';

const TextInputExample = () => {
  const [text, onChangeText] = React.useState('Useless Text');
  const [number, onChangeNumber] = React.useState('');

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

export default TextInputExample;
```

通過原生元素暴露的兩個方法是 .focus() 和 .blur()，可以以編程方式聚焦或模糊 TextInput。

Note that some props are only available with `multiline={true/false}`. Additionally, border styles that apply to only one side of the element (e.g., `borderBottomColor`, `borderLeftWidth`, etc.) will not be applied if `multiline=true`. To achieve the same effect, you can wrap your `TextInput` in a `View`:

```SnackPlayer name=TextInput
import React from 'react';
import {View, TextInput} from 'react-native';

const MultilineTextInputExample = () => {
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
      <TextInput
        editable
        multiline
        numberOfLines={4}
        maxLength={40}
        onChangeText={text => onChangeText(text)}
        value={value}
        style={{padding: 10}}
      />
    </View>
  );
};

export default MultilineTextInputExample;
```

`TextInput` 默認在其視圖底部有一個邊框。此邊框的填充由系統提供的背景圖像設置，無法更改。避免此問題的解決方案是要麼不顯式設置高度（在這種情況下，系統會負責在正確的位置顯示邊框），要麼通過將 `underlineColorAndroid` 設置為透明來不顯示邊框。

Note that on Android performing text selection in an input can change the app's activity `windowSoftInputMode` param to `adjustResize`. This may cause issues with components that have position: 'absolute' while the keyboard is active. To avoid this behavior either specify `windowSoftInputMode` in AndroidManifest.xml ( https://developer.android.com/guide/topics/manifest/activity-element.html ) or control this param programmatically with native code.

---

# 參考

## 屬性

### [View 屬性](view.md#props)

繼承 [View 屬性](view.md#props)。

---

### `allowFontScaling`

指定字體是否應縮放以尊重文本大小的無障礙設置。默認為 `true`。

| Type |
| ---- |
| bool |

---

### `autoCapitalize`

告訴 `TextInput` 自動大寫某些字符。此屬性不受某些鍵盤類型（如 `name-phone-pad`）支持。

- `characters`: all characters.
- `words`: first letter of each word.
- `sentences`: first letter of each sentence (_default_).
- `none`: don't auto capitalize anything.

| Type                                             |
| ------------------------------------------------ |
| enum('none', 'sentences', 'words', 'characters') |

---

### `autoComplete`

為系統指定自動完成提示，以便提供自動填充。在 Android 上，系統將始終嘗試通過使用啟發式方法來識別內容類型來提供自動填充。要禁用自動完成，請將 `autoComplete` 設置為 `off`。

以下值在所有平台上都有效：

- `additional-name`
- `address-line1`
- `address-line2`
- `cc-number`
- `country`
- `current-password`
- `email`
- `family-name`
- `given-name`
- `honorific-prefix`
- `honorific-suffix`
- `name`
- `new-password`
- `off`
- `one-time-code`
- `postal-code`
- `street-address`
- `tel`
- `username`

<div class="label basic ios">iOS</div>

以下值僅在 iOS 上有效：

- `nickname`
- `organization`
- `organization-title`
- `url`

<div class="label basic android">Android</div>

以下數值僅適用於 Android：

- `birthdate-day`
- `birthdate-full`
- `birthdate-month`
- `birthdate-year`
- `cc-csc`
- `cc-exp`
- `cc-exp-day`
- `cc-exp-month`
- `cc-exp-year`
- `gender`
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
- `sms-otp`
- `tel-country-code`
- `tel-national`
- `tel-device`
- `username-new`

| Type                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| enum('additional-name', 'address-line1', 'address-line2', 'birthdate-day', 'birthdate-full', 'birthdate-month', 'birthdate-year', 'cc-csc', 'cc-exp', 'cc-exp-day', 'cc-exp-month', 'cc-exp-year', 'cc-number', 'country', 'current-password', 'email', 'family-name', 'gender', 'given-name', 'honorific-prefix', 'honorific-suffix', 'name', 'name-family', 'name-given', 'name-middle', 'name-middle-initial', 'name-prefix', 'name-suffix', 'new-password', 'nickname', 'one-time-code', 'organization', 'organization-title', 'password', 'password-new', 'postal-address', 'postal-address-country', 'postal-address-extended', 'postal-address-extended-postal-code', 'postal-address-locality', 'postal-address-region', 'postal-code', 'street-address', 'sms-otp', 'tel', 'tel-country-code', 'tel-national', 'tel-device', 'url', 'username', 'username-new', 'off') |

---

### `autoCorrect`

若設為 `false`，則停用自動校正。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `autoFocus`

若設為 `true`，則在 `componentDidMount` 或 `useEffect` 時聚焦輸入框。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `blurOnSubmit`

若設為 `true`，文字欄位在提交時會失去焦點。單行欄位的預設值為 true，多行欄位則為 false。請注意，對於多行欄位，將 `blurOnSubmit` 設為 `true` 意味著按下返回鍵會使欄位失去焦點並觸發 `onSubmitEditing` 事件，而非在欄位中插入換行符號。

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

### `clearButtonMode` <div class="label ios">iOS</div>

決定清除按鈕何時出現在文字視圖的右側。此屬性僅支援單行 TextInput 元件。預設值為 `never`。

| Type                                                       |
| ---------------------------------------------------------- |
| enum('never', 'while-editing', 'unless-editing', 'always') |

---

### `clearTextOnFocus` <div class="label ios">iOS</div>

若設為 `true`，則在開始編輯時自動清除文字欄位。

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

### `dataDetectorTypes` <div class="label ios">iOS</div>

決定文字輸入中哪些類型的資料會被轉換為可點擊的 URL。僅在 `multiline={true}` 且 `editable={false}` 時有效。預設不檢測任何資料類型。

您可以提供單一類型或多種類型的陣列。

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

提供初始值，該值將在使用者開始輸入時改變。適用於您不想處理事件監聽並更新 value 屬性以保持受控狀態同步的使用情境。

| Type   |
| ------ |
| string |

---

### `cursorColor` <div class="label android">Android</div>

當提供此屬性時，它會設定元件中游標（或「插入符」）的顏色。與 `selectionColor` 的行為不同，游標顏色會獨立於文字選取框的顏色進行設定。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `disableFullscreenUI` <div class="label android">Android</div>

當設為 `false` 時，如果文字輸入周圍可用空間較少（例如手機橫向模式），作業系統可能會選擇讓使用者在全螢幕文字輸入模式中編輯文字。設為 `true` 時，此功能會被停用，使用者將始終直接在文字輸入框中編輯文字。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `editable`

若設為 `false`，文字將無法編輯。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `enablesReturnKeyAutomatically` <div class="label ios">iOS</div>

若設為 `true`，當沒有文字時鍵盤會停用返回鍵，並在有文字時自動啟用。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `enterKeyHint`

決定返回鍵應顯示的文字。此屬性優先於 `returnKeyType` 屬性。

以下值跨平台適用：

- `enter`
- `done`
- `next`
- `search`
- `send`

_僅限 Android_

以下值僅在 Android 上有效：

- `previous`

| Type                                                        |
| ----------------------------------------------------------- |
| enum('enter', 'done', 'next', 'previous', 'search', 'send') |

---

### `importantForAutofill` <div class="label android">Android</div>

告知作業系統應用程式中的個別欄位是否應包含在 Android API 26+ 的自動填充視圖結構中。可能的值為 `auto`、`no`、`noExcludeDescendants`、`yes` 和 `yesExcludeDescendants`。預設值為 `auto`。

- `auto`：讓 Android 系統使用其啟發式方法決定視圖對自動填充是否重要。
- `no`：此視圖對自動填充不重要。
- `noExcludeDescendants`：此視圖及其子視圖對自動填充不重要。
- `yes`：此視圖對自動填充重要。
- `yesExcludeDescendants`：此視圖對自動填充重要，但其子視圖不重要。

| Type                                                                       |
| -------------------------------------------------------------------------- |
| enum('auto', 'no', 'noExcludeDescendants', 'yes', 'yesExcludeDescendants') |

---

### `inlineImageLeft` <div class="label android">Android</div>

若定義此屬性，提供的圖片資源將顯示在左側。圖片資源必須位於 `/android/app/src/main/res/drawable` 目錄下，並以類似方式引用：

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

內嵌圖片（如有）與文字輸入框本身之間的間距。

| Type   |
| ------ |
| number |

---

### `inputAccessoryViewID` <div class="label ios">iOS</div>

一個可選標識符，用於將自訂的 [InputAccessoryView](inputaccessoryview.md) 連結到此文字輸入框。當此文字輸入框獲得焦點時，InputAccessoryView 會顯示在鍵盤上方。

| Type   |
| ------ |
| string |

---

### `inputMode`

功能類似於HTML中的`inputmode`屬性，它決定要開啟哪種鍵盤，例如`numeric`，且優先級高於`keyboardType`。

支援以下數值：

- `none`
- `text`
- `decimal`
- `numeric`
- `tel`
- `search`
- `email`
- `url`

| Type                                                                        |
| --------------------------------------------------------------------------- |
| enum('decimal', 'email', 'none', 'numeric', 'search', 'tel', 'text', 'url') |

---

### `keyboardAppearance` <div class="label ios">iOS</div>

決定鍵盤的顏色。

| Type                             |
| -------------------------------- |
| enum('default', 'light', 'dark') |

---

### `keyboardType`

決定要開啟哪種鍵盤，例如`numeric`。

所有類型的截圖可參見[這裡](http://lefkowitz.me/2018/04/30/visual-guide-to-react-native-textinput-keyboardtype-options/)。

以下數值跨平台通用：

- `default`
- `number-pad`
- `decimal-pad`
- `numeric`
- `email-address`
- `phone-pad`
- `url`

_iOS專用_

以下數值僅在iOS上有效：

- `ascii-capable`
- `numbers-and-punctuation`
- `name-phone-pad`
- `twitter`
- `web-search`

_Android專用_

以下數值僅在Android上有效：

- `visible-password`

| Type                                                                                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| enum('default', 'email-address', 'numeric', 'phone-pad', 'ascii-capable', 'numbers-and-punctuation', 'url', 'number-pad', 'name-phone-pad', 'decimal-pad', 'twitter', 'web-search', 'visible-password') |

---

### `maxFontSizeMultiplier`

指定當`allowFontScaling`啟用時，字體可達到的最大縮放比例。可能的數值：

- `null/undefined`（預設）：繼承父節點或全域預設值（0）
- `0`：無上限，忽略父節點/全域預設值
- `>= 1`：將此節點的`maxFontSizeMultiplier`設為該數值

| Type   |
| ------ |
| number |

---

### `maxLength`

限制可輸入的最大字元數。使用此屬性而非在JS中實作邏輯以避免閃爍。

| Type   |
| ------ |
| number |

---

### `multiline`

若為`true`，文字輸入框可支援多行。預設值為`false`。

:::note
需注意在iOS上會將文字對齊頂部，而在Android上會置中。若要兩平台行為一致，請將`textAlignVertical`設為`top`。
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

當文字輸入框失去焦點時觸發的回調函數。

> Note: If you are attempting to access the `text` value from `nativeEvent` keep in mind that the resulting value you get can be `undefined` which can cause unintended errors. If you are trying to find the last value of TextInput, you can use the [`onEndEditing`](textinput#onendediting) event, which is fired upon completion of editing.

| Type     |
| -------- |
| function |

---

### `onChange`

當文字輸入框的文字變更時呼叫的回調函式。

| Type                                                  |
| ----------------------------------------------------- |
| (`{nativeEvent: {eventCount, target, text}}`) => void |

---

### `onChangeText`

當文字輸入框的文字變更時呼叫的回調函式。變更後的文字會以單一字串參數的形式傳遞給回調處理函式。

| Type     |
| -------- |
| function |

---

### `onContentSizeChange`

當文字輸入框的內容大小變更時呼叫的回調函式。

僅適用於多行文字輸入框。

| Type                                                       |
| ---------------------------------------------------------- |
| (`{nativeEvent: {contentSize: {width, height} }}`) => void |

---

### `onEndEditing`

當文字輸入結束時呼叫的回調函式。

| Type     |
| -------- |
| function |

---

### `onPressIn`

當觸碰開始時呼叫的回調函式。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onPressOut`

當觸碰結束時呼叫的回調函式。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onFocus`

當文字輸入框獲得焦點時呼叫的回調函式。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)}) => void` |

---

### `onKeyPress`

當按鍵被按下時呼叫的回調函式。會傳遞一個物件，其中 `keyValue` 為 `'Enter'` 或 `'Backspace'`（分別對應 Enter 鍵和退格鍵），其他按鍵則為輸入的字元（包括空格 `' '`）。此回調會在 `onChange` 之前觸發。注意：在 Android 上僅處理軟鍵盤輸入，不處理硬體鍵盤輸入。

| Type                                        |
| ------------------------------------------- |
| (`{nativeEvent: {key: keyValue} }`) => void |

---

### `onLayout`

在掛載時或佈局變更時呼叫。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)}) => void` |

---

### `onScroll`

當內容滾動時呼叫。可能包含來自 `ScrollEvent` 的其他屬性，但在 Android 上由於效能考量不會提供 `contentSize`。

| Type                                                |
| --------------------------------------------------- |
| (`{nativeEvent: {contentOffset: {x, y} }}`) => void |

---

### `onSelectionChange`

當文字輸入框的選取範圍變更時呼叫的回調函式。

| Type                                                  |
| ----------------------------------------------------- |
| (`{nativeEvent: {selection: {start, end} }}`) => void |

---

### `onSubmitEditing`

當文字輸入框的提交按鈕被按下時呼叫的回調函式。

| Type                                                  |
| ----------------------------------------------------- |
| (`{nativeEvent: {text, eventCount, target}}`) => void |

注意：在 iOS 上，當使用 `keyboardType="phone-pad"` 時不會呼叫此方法。

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

### `readOnly`

若為 `true`，則文字不可編輯。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `returnKeyLabel` <div class="label android">Android</div>

設定返回鍵的標籤。可替代 `returnKeyType` 使用。

| Type   |
| ------ |
| string |

---

### `returnKeyType`

決定返回鍵的外觀。在 Android 上，您也可以使用 `returnKeyLabel`。

_跨平台_

以下值適用於所有平台：

- `done`
- `go`
- `next`
- `search`
- `send`

_僅限 Android_

以下值僅在 Android 上有效：

- `none`
- `previous`

_僅限 iOS_

以下值僅在 iOS 上有效：

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

如果設為 `true`，允許 TextInput 將觸控事件傳遞給父組件。這使得如 SwipeableListView 等組件可以從 TextInput 上滑動（在 Android 上預設即為如此）。如果設為 `false`，TextInput 總是會要求處理輸入（除非被禁用）。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `rows` <div class="label android">Android</div>

Sets the number of lines for a `TextInput`. Use it with multiline set to `true` to be able to fill the lines.

| Type   |
| ------ |
| number |

---

### `scrollEnabled` <div class="label ios">iOS</div>

如果設為 `false`，將禁用文本視圖的滾動。預設值為 `true`。僅在 `multiline={true}` 時有效。

| Type |
| ---- |
| bool |

---

### `secureTextEntry`

如果設為 `true`，文本輸入會隱藏輸入內容以保護敏感信息（如密碼）。預設值為 `false`。不適用於 `multiline={true}`。

| Type |
| ---- |
| bool |

---

### `selection`

文本輸入的選取範圍的起始和結束位置。將起始和結束設為相同值以定位游標。

| Type                                  |
| ------------------------------------- |
| object: `{start: number,end: number}` |

---

### `selectionColor`

文本輸入的高亮和游標顏色。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `selectTextOnFocus`

如果設為 `true`，所有文本將在獲得焦點時自動被選取。

| Type |
| ---- |
| bool |

---

### `showSoftInputOnFocus`

當設為 `false` 時，將阻止字段獲得焦點時顯示軟鍵盤。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `spellCheck` <div class="label ios">iOS</div>

如果設為 `false`，將禁用拼寫檢查樣式（如紅色底線）。預設值繼承自 `autoCorrect`。

| Type |
| ---- |
| bool |

---

### `textAlign`

將輸入文本對齊輸入字段的左側、居中或右側。

Possible values for `textAlign` are:

- `left`
- `center`
- `right`

| Type                            |
| ------------------------------- |
| enum('left', 'center', 'right') |

---

### `textContentType` <div class="label ios">iOS</div>

向鍵盤和系統提供關於使用者輸入內容預期語義的資訊。

:::note
[`autoComplete`](#autocomplete) 提供相同功能且支援全平台。您可使用 [`Platform.select`](/docs/next/platform#select) 處理不同平台的行為差異。

請避免同時使用 `textContentType` 和 `autoComplete`。為保持向後兼容性，當兩個屬性同時設定時，`textContentType` 會優先生效。
:::

在 iOS 11+ 上，可將 `textContentType` 設為 `username` 或 `password` 以啟用裝置鑰匙圈自動填入登入資訊的功能。

在 iOS 12+ 上，`newPassword` 可用於標示使用者可能想儲存至鑰匙圈的新密碼輸入欄位，而 `oneTimeCode` 則表示該欄位可透過簡訊收到的驗證碼自動填入。

若要停用自動填入功能，請將 `textContentType` 設為 `none`。

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

當在 iOS 上使用 `textContentType` 設為 `newPassword` 時，可透過此屬性讓系統知悉密碼的最低要求，以便生成符合條件的密碼。建立有效的 `PasswordRules` 字串請參閱 [Apple 官方文件](https://developer.apple.com/password-rules/)。

> If passwords generation dialog doesn't appear please make sure that:
>
> - AutoFill is enabled: **Settings** → **Passwords & Accounts** → toggle "On" the **AutoFill Passwords**,
> - iCloud Keychain is used: **Settings** → **Apple ID** → **iCloud** → **Keychain** → toggle "On" the **iCloud Keychain**.

| Type   |
| ------ |
| string |

---

### `style`

請注意並非所有文字樣式都受支援，以下為部分不支援的樣式清單：

- `borderLeftWidth`
- `borderTopWidth`
- `borderRightWidth`
- `borderBottomWidth`
- `borderTopLeftRadius`
- `borderTopRightRadius`
- `borderBottomRightRadius`
- `borderBottomLeftRadius`

詳見 [Issue#7070](https://github.com/facebook/react-native/issues/7070) 取得更多細節。

[樣式](style.md)

| Type                  |
| --------------------- |
| [Text](text.md#style) |

---

### `textBreakStrategy` <div class="label android">Android</div>

在 Android API Level 23+ 設定文字斷行策略，可能值為 `simple`、`highQuality`、`balanced`，預設值為 `highQuality`。

| Type                                      |
| ----------------------------------------- |
| enum('simple', 'highQuality', 'balanced') |

---

### `underlineColorAndroid` <div class="label android">Android</div>

設定 `TextInput` 底線的顏色。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `value`

文字輸入框顯示的值。`TextInput` 是一個受控元件，這意味著如果提供了這個值屬性，原生值將被強制匹配此值。在大多數情況下，這運作良好，但在某些情況下可能會導致閃爍——一個常見的原因是通過保持值不變來防止編輯。除了設置相同的值外，可以設置 `editable={false}`，或者設置/更新 `maxLength` 以防止不需要的編輯而不會閃爍。

| Type   |
| ------ |
| string |

---

### `lineBreakStrategyIOS` <div class="label ios">iOS</div>

在 iOS 14+ 上設置換行策略。可能的值有 `none`、`standard`、`hangul-word` 和 `push-out`。

| Type                                                        | Default  |
| ----------------------------------------------------------- | -------- |
| enum(`'none'`, `'standard'`, `'hangul-word'`, `'push-out'`) | `'none'` |

## 方法

### `.focus()`

```tsx
focus();
```

使原生輸入請求焦點。

### `.blur()`

```tsx
blur();
```

使原生輸入失去焦點。

### `clear()`

```tsx
clear();
```

清除 `TextInput` 中的所有文字。

---

### `isFocused()`

```tsx
isFocused(): boolean;
```

如果輸入目前有焦點，則返回 `true`；否則返回 `false`。

# 已知問題

- [react-native#19096](https://github.com/facebook/react-native/issues/19096): Doesn't support Android's `onKeyPreIme`.
- [react-native#19366](https://github.com/facebook/react-native/issues/19366): Calling .focus() after closing Android's keyboard via back button doesn't bring keyboard up again.
- [react-native#26799](https://github.com/facebook/react-native/issues/26799): Doesn't support Android's `secureTextEntry` when `keyboardType="email-address"` or `keyboardType="phone-pad"`.