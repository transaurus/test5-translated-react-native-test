---
id: accessibility
title: Accessibility
description: Create mobile apps accessible to assistive technology with React Native's suite of APIs designed to work with Android and iOS.
---

Android 和 iOS 都提供 API 讓應用程式能與輔助技術（如內建的螢幕閱讀器 VoiceOver（iOS）和 TalkBack（Android））整合。React Native 提供相應的 API，讓您的應用程式能服務所有使用者。

:::info
Android 和 iOS 的實現方式略有不同，因此 React Native 的實作可能因平台而異。
:::

## 無障礙屬性

### `accessible`

當設為 `true` 時，表示該視圖是一個無障礙元素。當視圖是無障礙元素時，它會將其子元素分組為一個可選取的元件。預設情況下，所有可觸碰元素都是無障礙的。

在 Android 上，react-native View 的 `accessible={true}` 屬性會被轉換為原生的 `focusable={true}`。

```jsx
<View accessible={true}>
  <Text>text one</Text>
  <Text>text two</Text>
</View>
```

在上面的例子中，我們無法分別在「text one」和「text two」上獲得無障礙焦點。相反，我們會在具有「accessible」屬性的父視圖上獲得焦點。

### `accessibilityLabel`

當視圖被標記為無障礙時，最好在視圖上設置一個 accessibilityLabel，這樣使用 VoiceOver 的人就知道他們選擇了什麼元素。當用戶選擇相關元素時，VoiceOver 會讀出這個字串。

要使用此功能，請在您的 View、Text 或 Touchable 上將 `accessibilityLabel` 屬性設為自訂字串：

```jsx
<TouchableOpacity
  accessible={true}
  accessibilityLabel="Tap me!"
  onPress={onPress}>
  <View style={styles.button}>
    <Text style={styles.buttonText}>Press me!</Text>
  </View>
</TouchableOpacity>
```

在上面的例子中，TouchableOpacity 元素的 `accessibilityLabel` 預設為「Press me!」。標籤是通過將所有 Text 節點子元素以空格分隔連接起來構建的。

### `accessibilityLabelledBy` <div class="label android">Android</div>

用於構建複雜表單的對另一個元素 [nativeID](view.md#nativeid) 的引用。`accessibilityLabelledBy` 的值應與相關元素的 `nativeID` 匹配：

```jsx
<View>
  <Text nativeID="formLabel">Label for Input Field</Text>
  <TextInput
    accessibilityLabel="input"
    accessibilityLabelledBy="formLabel"
  />
</View>
```

In the above example, the screenreader announces `Input, Edit Box for Label for Input Field` when focusing on the TextInput.

### `accessibilityHint`

當無障礙標籤無法清楚表達執行操作後的結果時，無障礙提示能幫助使用者理解在無障礙元素上執行操作時會發生什麼。

要使用此功能，請在您的 View、Text 或 Touchable 上將 `accessibilityHint` 屬性設為自訂字串：

```jsx
<TouchableOpacity
  accessible={true}
  accessibilityLabel="Go back"
  accessibilityHint="Navigates to the previous screen"
  onPress={onPress}>
  <View style={styles.button}>
    <Text style={styles.buttonText}>Back</Text>
  </View>
</TouchableOpacity>
```

<div class="label ios basic">iOS</div>

In the above example, VoiceOver will read the hint after the label, if the user has hints enabled in the device's VoiceOver settings. Read more about guidelines for `accessibilityHint` in the [iOS Developer Docs](https://developer.apple.com/documentation/objectivec/nsobject/1615093-accessibilityhint)

<div class="label android basic">Android</div>

在上面的例子中，TalkBack 會在標籤後讀出提示。目前，Android 上的提示無法關閉。

### `accessibilityLanguage` <div class="label ios">iOS</div>

通過使用 `accessibilityLanguage` 屬性，螢幕閱讀器會理解在讀取元素的**標籤**、**值**和**提示**時應使用哪種語言。提供的字串值必須遵循 [BCP 47 規範](https://www.rfc-editor.org/info/bcp47)。

```jsx
<View
  accessible={true}
  accessibilityLabel="Pizza"
  accessibilityLanguage="it-IT">
  <Text>🍕</Text>
</View>
```

### `accessibilityIgnoresInvertColors` <div class="label ios">iOS</div>

反轉螢幕顏色是一項無障礙功能，能讓iPhone和iPad對光線敏感的使用者更舒適、色盲使用者更容易辨識，以及低視力使用者更易於觀看。但有時您會有些不想被反轉的視圖（例如照片）。此時可將此屬性設為`true`，使這些特定視圖的顏色不被反轉。

### `accessibilityLiveRegion` <div class="label android">Android</div>

當元件動態變化時，我們希望TalkBack能通知終端使用者。透過`accessibilityLiveRegion`屬性可實現此功能，其值可設為`none`、`polite`和`assertive`：

- **none** 無障礙服務不應通報此視圖的變更。
- **polite** 無障礙服務應通報此視圖的變更。
- **assertive** 無障礙服務應中斷當前語音，立即通報此視圖的變更。

```jsx
<TouchableWithoutFeedback onPress={addOne}>
  <View style={styles.embedded}>
    <Text>Click me</Text>
  </View>
</TouchableWithoutFeedback>
<Text accessibilityLiveRegion="polite">
  Clicked {count} times
</Text>
```

在上述範例中，方法`addOne`會改變狀態變數`count`。當終端使用者點擊TouchableWithoutFeedback時，由於Text視圖設有`accessibilityLiveRegion="polite"`屬性，TalkBack會朗讀其中的文字。

### `accessibilityRole`

`accessibilityRole`向輔助技術使用者傳達元件的用途。

`accessibilityRole`可為下列值之一：

- **adjustable** 用於可「調整」的元素（如滑桿）。
- **alert** 用於包含需向使用者呈現重要文字的元素。
- **button** 用於應被視為按鈕的元素。
- **checkbox** 用於代表可勾選、取消勾選或混合勾選狀態的核取方塊。
- **combobox** 用於代表可讓使用者從多個選項中選擇的下拉式方塊。
- **header** 用於作為內容區段標題的元素（如導覽列標題）。
- **image** 用於應被視為圖像的元素。可與按鈕或連結等組合使用。
- **imagebutton** 用於應被視為按鈕且同時是圖像的元素。
- **keyboardkey** 用於作為鍵盤按鍵的元素。
- **link** 用於應被視為連結的元素。
- **menu** 用於代表選單的元件。
- **menubar** 用於作為多個選單容器的元件。
- **menuitem** 用於代表選單中的項目。
- **none** 用於沒有角色的元素。
- **progressbar** 用於代表顯示任務進度的元件。
- **radio** 用於代表單選按鈕。
- **radiogroup** 用於代表一組單選按鈕。
- **scrollbar** 用於代表捲軸。
- **search** 用於應同時被視為搜尋欄位的文字欄位元素。
- **spinbutton** 用於代表可開啟選項清單的按鈕。
- **summary** 用於在應用程式首次啟動時提供當前狀態快速摘要的元素。
- **switch** 用於代表可開啟/關閉的開關。
- **tab** 用於代表分頁標籤。
- **tablist** 用於代表分頁標籤清單。
- **text** 用於應被視為靜態不可變文字的元件。
- **timer** 用於代表計時器。
- **togglebutton** 用於代表切換按鈕。需搭配accessibilityState的checked屬性來指示按鈕是否處於切換狀態。
- **toolbar** 用於代表工具列（動作按鈕或元件的容器）。

### `accessibilityState`

向輔助技術使用者描述元件的當前狀態。

`accessibilityState`是一個物件，包含以下欄位：

| Name     | Description                                                                                                                           | Type               | Required |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | -------- |
| disabled | Indicates whether the element is disabled or not.                                                                                     | boolean            | No       |
| selected | Indicates whether a selectable element is currently selected or not.                                                                  | boolean            | No       |
| checked  | Indicates the state of a checkable element. This field can either take a boolean or the "mixed" string to represent mixed checkboxes. | boolean or 'mixed' | No       |
| busy     | Indicates whether an element is currently busy or not.                                                                                | boolean            | No       |
| expanded | Indicates whether an expandable element is currently expanded or collapsed.                                                           | boolean            | No       |

使用時，需將`accessibilityState`設為具有特定定義的物件。

### `accessibilityValue`

代表元件當前值。可以是元件值的文字描述，對於範圍型元件（如滑桿和進度條），則包含範圍資訊（最小值、當前值和最大值）。

`accessibilityValue` 是一個物件，包含以下欄位：

| Name | Description                                                                                    | Type    | Required                  |
| ---- | ---------------------------------------------------------------------------------------------- | ------- | ------------------------- |
| min  | The minimum value of this component's range.                                                   | integer | Required if `now` is set. |
| max  | The maximum value of this component's range.                                                   | integer | Required if `now` is set. |
| now  | The current value of this component's range.                                                   | integer | No                        |
| text | A textual description of this component's value. Will override `min`, `now`, and `max` if set. | string  | No                        |

### `accessibilityViewIsModal` <div class="label ios">iOS</div>

布林值，表示 VoiceOver 是否應忽略接收器同層級視圖中的元素。

For example, in a window that contains sibling views `A` and `B`, setting `accessibilityViewIsModal` to `true` on view `B` causes VoiceOver to ignore the elements in the view `A`. On the other hand, if view `B` contains a child view `C` and you set `accessibilityViewIsModal` to `true` on view `C`, VoiceOver does not ignore the elements in view `A`.

### `accessibilityElementsHidden` <div class="label ios">iOS</div>

布林值，表示此無障礙元素內包含的子元素是否隱藏。

For example, in a window that contains sibling views `A` and `B`, setting `accessibilityElementsHidden` to `true` on view `B` causes VoiceOver to ignore the elements in the view `B`. This is similar to the Android property `importantForAccessibility="no-hide-descendants"`.

### `importantForAccessibility` <div class="label android">Android</div>

當兩個同父元件的 UI 元件重疊時，預設無障礙焦點可能出現不可預測的行為。`importantForAccessibility` 屬性可透過控制視圖是否觸發無障礙事件及是否回報給無障礙服務來解決此問題。可設為 `auto`、`yes`、`no` 或 `no-hide-descendants`（最後一個值會強制無障礙服務忽略該元件及其所有子元件）。

```jsx
<View style={styles.container}>
  <View
    style={[styles.layout, {backgroundColor: 'green'}]}
    importantForAccessibility="yes">
    <Text>First layout</Text>
  </View>
  <View
    style={[styles.layout, {backgroundColor: 'yellow'}]}
    importantForAccessibility="no-hide-descendants">
    <Text>Second layout</Text>
  </View>
</View>
```

上述範例中，`yellow` 佈局及其子元件對 TalkBack 和其他無障礙服務完全不可見。因此我們可以在同父元件中使用重疊視圖，而不會造成 TalkBack 混淆。

### `onAccessibilityEscape` <div class="label ios">iOS</div>

將此屬性賦予自訂函式，當使用者執行「escape」手勢（雙指 Z 形手勢）時會呼叫該函式。escape 函式應在使用者介面中依層級返回，例如在導覽階層中向上或返回，或關閉模態介面。若選定元件未定義 `onAccessibilityEscape` 函式，系統會嘗試向上遍歷視圖階層，直到找到符合的視圖或發出提示音表示找不到。

### `onAccessibilityTap` <div class="label ios">iOS</div>

使用此屬性賦予自訂函式，當使用者在選定無障礙元件上雙擊觸發時會呼叫該函式。

### `onMagicTap` <div class="label ios">iOS</div>

將此屬性賦予自訂函式，當使用者執行「magic tap」手勢（雙指雙擊）時會呼叫該函式。magic tap 函式應執行使用者對元件最相關的操作。例如 iPhone 上的電話應用中，magic tap 會接聽或掛斷電話。若選定元件未定義 `onMagicTap` 函式，系統會向上遍歷視圖階層直到找到符合的視圖。

## 無障礙操作

無障礙操作允許輔助技術以程式化方式觸發元件的操作。要支援無障礙操作，元件必須完成兩件事：

- 透過 `accessibilityActions` 屬性定義支援的操作清單。
- 實作 `onAccessibilityAction` 函式來處理操作請求。

The `accessibilityActions` property should contain a list of action objects. Each action object should contain the following fields:

| Name  | Type   | Required |
| ----- | ------ | -------- |
| name  | string | Yes      |
| label | string | No       |

動作可以代表標準操作（例如點擊按鈕或調整滑塊），或是特定元件的自訂動作（例如刪除電子郵件）。標準動作和自訂動作都必須提供 `name` 欄位，但標準動作的 `label` 欄位是選填的。

當新增對標準動作的支援時，`name` 必須是以下其中之一：

- `'magicTap'` - 僅限 iOS - 當 VoiceOver 焦點位於元件上或內部時，使用者用兩指雙擊。
- `'escape'` - 僅限 iOS - 當 VoiceOver 焦點位於元件上或內部時，使用者執行了兩指擦除手勢（左、右、左）。
- `'activate'` - 啟動元件。通常這應執行與使用者在未使用輔助技術時觸摸或點擊元件相同的動作。當螢幕閱讀器使用者雙擊元件時會產生此動作。
- `'increment'` - 增加可調整元件的值。在 iOS 上，當元件具有 `'adjustable'` 角色且使用者將焦點放在其上並向上滑動時，VoiceOver 會產生此動作。在 Android 上，當使用者將輔助焦點放在元件上並按下音量增大按鈕時，TalkBack 會產生此動作。
- `'decrement'` - 減少可調整元件的值。在 iOS 上，當元件具有 `'adjustable'` 角色且使用者將焦點放在其上並向下滑動時，VoiceOver 會產生此動作。在 Android 上，當使用者將輔助焦點放在元件上並按下音量減小按鈕時，TalkBack 會產生此動作。
- `'longpress'` - 僅限 Android - 當使用者將輔助焦點放在元件上並雙擊並按住螢幕上的一指時，會產生此動作。通常這應執行與使用者在未使用輔助技術時按住元件相同的動作。

The `label` field is optional for standard actions, and is often unused by assistive technologies. For custom actions, it is a localized string containing a description of the action to be presented to the user.

要處理動作請求，元件必須實作 `onAccessibilityAction` 函數。此函數的唯一參數是一個包含要執行動作名稱的事件。以下來自 RNTester 的範例展示了如何建立一個定義並處理多個自訂動作的元件。

```jsx
<View
  accessible={true}
  accessibilityActions={[
    {name: 'cut', label: 'cut'},
    {name: 'copy', label: 'copy'},
    {name: 'paste', label: 'paste'},
  ]}
  onAccessibilityAction={event => {
    switch (event.nativeEvent.actionName) {
      case 'cut':
        Alert.alert('Alert', 'cut action success');
        break;
      case 'copy':
        Alert.alert('Alert', 'copy action success');
        break;
      case 'paste':
        Alert.alert('Alert', 'paste action success');
        break;
    }
  }}
/>
```

## 檢查螢幕閱讀器是否啟用

The `AccessibilityInfo` API allows you to determine whether or not a screen reader is currently active. See the [AccessibilityInfo documentation](accessibilityinfo) for details.

## 發送輔助功能事件 <div class="label android">Android</div>

Sometimes it is useful to trigger an accessibility event on a UI component (i.e. when a custom view appears on a screen or set accessibility focus to a view). Native UIManager module exposes a method ‘sendAccessibilityEvent’ for this purpose. It takes two arguments: view tag and a type of an event. The supported event types are `typeWindowStateChanged`, `typeViewFocused` and `typeViewClicked`.

```jsx
import {Platform, UIManager, findNodeHandle} from 'react-native';

if (Platform.OS === 'android') {
  UIManager.sendAccessibilityEvent(
    findNodeHandle(this),
    UIManager.AccessibilityEventTypes.typeViewFocused,
  );
}
```

## 測試 TalkBack 支援 <div class="label android">Android</div>

要啟用 TalkBack，請前往 Android 裝置或模擬器上的「設定」應用程式。點擊「無障礙」，然後點擊「TalkBack」。切換「使用服務」開關以啟用或停用它。

Android 模擬器預設未安裝 TalkBack。您可以透過 Google Play 商店在模擬器上安裝 TalkBack。請確保選擇已安裝 Google Play 商店的模擬器。這些在 Android Studio 中可用。

您可以使用音量鍵快捷方式來切換 TalkBack。要開啟音量鍵快捷方式，請前往「設定」應用程式，然後點擊「無障礙」。在頂部，開啟「音量鍵快捷方式」。

要使用音量鍵快捷方式，按住兩個音量鍵 3 秒以啟動無障礙工具。

此外，如果您願意，也可以透過命令列切換 TalkBack：

```shell
# disable
adb shell settings put secure enabled_accessibility_services com.android.talkback/com.google.android.marvin.talkback.TalkBackService

# enable
adb shell settings put secure enabled_accessibility_services com.google.android.marvin.talkback/com.google.android.marvin.talkback.TalkBackService
```

## 測試 VoiceOver 支援 <div class="label ios">iOS</div>

要啟用 VoiceOver，請前往 iOS 裝置上的「設定」應用程式（模擬器不支援此功能）。點選「一般」，然後選擇「輔助使用」。在這裡您會找到許多讓裝置更易於使用的工具，例如粗體文字、增加對比度以及 VoiceOver。

要啟用 VoiceOver，請在「視覺」下方點選 VoiceOver，並切換頂部的開關。

在「輔助使用」設定的最底部，有一個「輔助使用快速鍵」。您可以使用此功能，透過三擊主畫面按鈕來切換 VoiceOver。

## 其他資源

- [讓 React Native 應用程式具備無障礙功能](https://engineering.fb.com/ios/making-react-native-apps-accessible/)