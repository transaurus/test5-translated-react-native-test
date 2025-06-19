---
id: accessibility
title: Accessibility
description: Create mobile apps accessible to assistive technology with React Native's suite of APIs designed to work with Android and iOS.
---

Android 和 iOS 都提供 API 來整合應用程式與輔助技術，例如內建的螢幕閱讀器 VoiceOver (iOS) 和 TalkBack (Android)。React Native 提供互補的 API，讓你的應用程式能服務所有使用者。

:::info
Android 和 iOS 的方法略有不同，因此 React Native 的實作可能會因平台而異。
:::

## 無障礙屬性

### `accessible`

當設為 `true` 時，表示該視圖是一個無障礙元素。當視圖是無障礙元素時，它會將其子元素分組為一個可選取的元件。預設情況下，所有可觸碰元素都是無障礙的。

在 Android 上，React Native View 的 `accessible={true}` 屬性會被轉換為原生的 `focusable={true}`。

```tsx
<View accessible={true}>
  <Text>text one</Text>
  <Text>text two</Text>
</View>
```

在上面的例子中，無障礙焦點僅適用於具有 `accessible` 屬性的父視圖，而不適用於「text one」和「text two」這兩個子元素。

### `accessibilityLabel`

當視圖被標記為無障礙時，最好在視圖上設置一個 `accessibilityLabel`，這樣使用 VoiceOver 或 TalkBack 的使用者可以知道他們選擇了什麼元素。螢幕閱讀器會在相關元素被選中時朗讀這個字串。

使用時，請在你的 View、Text 或 Touchable 上設置 `accessibilityLabel` 屬性為自訂字串：

```tsx
<TouchableOpacity
  accessible={true}
  accessibilityLabel="Tap me!"
  onPress={onPress}>
  <View style={styles.button}>
    <Text style={styles.buttonText}>Press me!</Text>
  </View>
</TouchableOpacity>
```

在上面的例子中，TouchableOpacity 元素的 `accessibilityLabel` 預設為「Press me!」。標籤是通過串接所有 Text 節點子元素並以空格分隔來構建的。

### `accessibilityLabelledBy` <div class="label android">Android</div>

用於構建複雜表單的參考其他元素的 [nativeID](view.md#nativeid)。
`accessibilityLabelledBy` 的值應與相關元素的 `nativeID` 匹配：

```tsx
<View>
  <Text nativeID="formLabel">Label for Input Field</Text>
  <TextInput
    accessibilityLabel="input"
    accessibilityLabelledBy="formLabel"
  />
</View>
```

In the above example, the screen reader announces `Input, Edit Box for Label for Input Field` when focusing on the TextInput.

### `accessibilityHint`

無障礙提示可用於在使用者無法僅從無障礙標籤中清楚了解操作結果時，提供額外的上下文。

在你的 View、Text 或 Touchable 上提供 `accessibilityHint` 屬性為自訂字串：

```tsx
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

在上面的例子中，TalkBack 會在標籤之後朗讀提示。目前，Android 上的提示無法關閉。

### `accessibilityLanguage` <div class="label ios">iOS</div>

通過使用 `accessibilityLanguage` 屬性，螢幕閱讀器會知道在朗讀元素的**標籤**、**值**和**提示**時應使用哪種語言。提供的字串值必須遵循 [BCP 47 規範](https://www.rfc-editor.org/info/bcp47)。

```tsx
<View
  accessible={true}
  accessibilityLabel="Pizza"
  accessibilityLanguage="it-IT">
  <Text>🍕</Text>
</View>
```

### `accessibilityIgnoresInvertColors` <div class="label ios">iOS</div>

反轉螢幕顏色是 iOS 和 iPadOS 中為色盲、低視力或視力障礙者提供的無障礙功能。如果有一個視圖你不想在開啟此設定時反轉（例如照片），請將此屬性設為 `true`。

### `accessibilityLiveRegion` <div class="label android">Android</div>

當元件動態變化時，我們希望 TalkBack 能通知終端使用者。這可以通過 `accessibilityLiveRegion` 屬性實現，可設置為 `none`、`polite` 和 `assertive`：

- **none** 輔助服務不應宣佈此視圖的變更。
- **polite** 輔助服務應宣佈此視圖的變更。
- **assertive** 輔助服務應中斷當前語音以立即宣佈此視圖的變更。

```tsx
<TouchableWithoutFeedback onPress={addOne}>
  <View style={styles.embedded}>
    <Text>Click me</Text>
  </View>
</TouchableWithoutFeedback>
<Text accessibilityLiveRegion="polite">
  Clicked {count} times
</Text>
```

上述範例中，方法 `addOne` 會改變狀態變數 `count`。當 TouchableWithoutFeedback 被觸發時，由於 Text 視圖設有 `accessibilityLiveRegion="polite"` 屬性，TalkBack 會朗讀其中的文字。

### `accessibilityRole`

`accessibilityRole` 向輔助技術使用者傳達元件的用途。

`accessibilityRole` 可為以下值之一：

- **adjustable** 用於可「調整」的元件（如滑桿）。
- **alert** 用於包含需向使用者展示重要文字的元件。
- **button** 用於應被視為按鈕的元件。
- **checkbox** 用於表示可勾選、取消勾選或處於混合勾選狀態的核取方塊。
- **combobox** 用於表示允許使用者從多個選項中選擇的組合框。
- **header** 用於作為內容區段標題的元件（如導覽列標題）。
- **image** 用於應被視為圖像的元件，可與按鈕或連結結合使用。
- **imagebutton** 用於應被視為按鈕且同時是圖像的元件。
- **keyboardkey** 用於作為鍵盤按鍵的元件。
- **link** 用於應被視為連結的元件。
- **menu** 用於作為選單的元件。
- **menubar** 用於作為多個選單容器的元件。
- **menuitem** 用於表示選單中的項目。
- **none** 用於沒有角色的元件。
- **progressbar** 用於表示任務進度的元件。
- **radio** 用於表示單選按鈕。
- **radiogroup** 用於表示單選按鈕群組。
- **scrollbar** 用於表示捲軸。
- **search** 用於應同時被視為搜尋欄的文字欄位元件。
- **spinbutton** 用於表示可開啟選項清單的按鈕。
- **summary** 用於在應用程式首次啟動時提供當前條件的快速摘要。
- **switch** 用於表示可開關的切換開關。
- **tab** 用於表示標籤頁。
- **tablist** 用於表示標籤頁清單。
- **text** 用於應被視為靜態不可變文字的元件。
- **timer** 用於表示計時器。
- **togglebutton** 用於表示切換按鈕，需搭配 accessibilityState checked 表示按鈕是否處於開啟狀態。
- **toolbar** 用於表示工具列（動作按鈕或元件的容器）。
- **grid** 與 ScrollView、VirtualizedList、FlatList 或 SectionList 搭配使用表示網格，會為 Android 的 GridView 添加網格內/外公告。

### `accessibilityState`

向輔助技術使用者描述元件的當前狀態。

`accessibilityState` 是一個物件，包含以下欄位：

| Name     | Description                                                                                                                           | Type               | Required |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | -------- |
| disabled | Indicates whether the element is disabled or not.                                                                                     | boolean            | No       |
| selected | Indicates whether a selectable element is currently selected or not.                                                                  | boolean            | No       |
| checked  | Indicates the state of a checkable element. This field can either take a boolean or the "mixed" string to represent mixed checkboxes. | boolean or 'mixed' | No       |
| busy     | Indicates whether an element is currently busy or not.                                                                                | boolean            | No       |
| expanded | Indicates whether an expandable element is currently expanded or collapsed.                                                           | boolean            | No       |

使用時，將 `accessibilityState` 設為具有特定定義的物件。

### `accessibilityValue`

表示元件的當前值。可以是元件值的文字描述，或針對範圍型元件（如滑桿和進度條）包含範圍資訊（最小值、當前值和最大值）。

`accessibilityValue` 是一個物件，包含以下欄位：

| Name | Description                                                                                    | Type    | Required                  |
| ---- | ---------------------------------------------------------------------------------------------- | ------- | ------------------------- |
| min  | The minimum value of this component's range.                                                   | integer | Required if `now` is set. |
| max  | The maximum value of this component's range.                                                   | integer | Required if `now` is set. |
| now  | The current value of this component's range.                                                   | integer | No                        |
| text | A textual description of this component's value. Will override `min`, `now`, and `max` if set. | string  | No                        |

### `accessibilityViewIsModal` <div class="label ios">iOS</div>

一個布林值，用於指示 VoiceOver 是否應忽略接收者同層級視圖中的元素。

For example, in a window that contains sibling views `A` and `B`, setting `accessibilityViewIsModal` to `true` on view `B` causes VoiceOver to ignore the elements in view `A`. On the other hand, if view `B` contains a child view `C` and you set `accessibilityViewIsModal` to `true` on view `C`, VoiceOver does not ignore the elements in view `A`.

### `accessibilityElementsHidden` <div class="label ios">iOS</div>

一個布林值，用於指示此無障礙元素內包含的元素是否隱藏。

For example, in a window that contains sibling views `A` and `B`, setting `accessibilityElementsHidden` to `true` on view `B` causes VoiceOver to ignore the elements in view `B`. This is similar to the Android property `importantForAccessibility="no-hide-descendants"`.

### `aria-valuemax`

表示範圍型元件（如滑塊和進度條）的最大值。

### `aria-valuemin`

表示範圍型元件（如滑塊和進度條）的最小值。

### `aria-valuenow`

表示範圍型元件（如滑塊和進度條）的當前值。

### `aria-valuetext`

表示元件的文字描述。

### `aria-busy`

表示元素正在被修改，輔助技術可能需要等待更改完成後再通知用戶更新。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

### `aria-checked`

表示可勾選元素的狀態。此字段可以接受布林值或字串 "mixed" 來表示混合狀態的複選框。

| Type             | Default |
| ---------------- | ------- |
| boolean, 'mixed' | false   |

### `aria-disabled`

表示元素是可感知的但已被禁用，因此不可編輯或操作。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

### `aria-expanded`

表示可展開元素當前是展開還是折疊狀態。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

### `aria-hidden`

表示此無障礙元素內包含的元素是否隱藏。

For example, in a window that contains sibling views `A` and `B`, setting `aria-hidden` to `true` on view `B` causes VoiceOver to ignore the elements in view `B`.

| Type    | Default |
| ------- | ------- |
| boolean | false   |

### `aria-label`

定義一個字串值，用於標記互動元素。

| Type   |
| ------ |
| string |

### `aria-labelledby` <div class="label android">Android</div>

標識用於標記應用此屬性的元素的元素。`aria-labelledby` 的值應與相關元素的 [`nativeID`](view.md#nativeid) 匹配：

```tsx
<View>
  <Text nativeID="formLabel">Label for Input Field</Text>
  <TextInput aria-label="input" aria-labelledby="formLabel" />
</View>
```

| Type   |
| ------ |
| string |

### `aria-live` <div class="label android">Android</div>

表示元素將被更新，並描述用戶代理、輔助技術和用戶可以從實時區域獲得的更新類型。

- **off** 無障礙服務不應宣讀此視圖的變更。
- **polite** 無障礙服務應宣讀此視圖的變更。
- **assertive** 無障礙服務應中斷當前語音，立即宣讀此視圖的變更。

| Type                                     | Default |
| ---------------------------------------- | ------- |
| enum(`'assertive'`, `'off'`, `'polite'`) | `'off'` |

---

### `aria-modal` <div class="label ios">iOS</div>

布林值，指示 VoiceOver 是否應忽略接收者同級視圖中的元素。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

### `aria-selected`

指示可選元素當前是否被選中。

| Type    |
| ------- |
| boolean |

### `importantForAccessibility` <div class="label android">Android</div>

當兩個具有相同父元素的 UI 組件重疊時，預設的無障礙焦點可能會有不可預測的行為。`importantForAccessibility` 屬性可以解決這個問題，它控制視圖是否觸發無障礙事件以及是否向無障礙服務報告。它可以設置為 `auto`、`yes`、`no` 和 `no-hide-descendants`（最後一個值會強制無障礙服務忽略該組件及其所有子元素）。

```tsx
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

在上面的例子中，`yellow` 佈局及其子元素對 TalkBack 和所有其他無障礙服務完全不可見。因此，我們可以使用具有相同父元素的重疊視圖，而不會混淆 TalkBack。

### `onAccessibilityEscape` <div class="label ios">iOS</div>

將此屬性分配給一個自定義函數，當有人執行「escape」手勢（即兩指 Z 形手勢）時，該函數將被調用。escape 函數應在用戶界面中按層次結構向上移動。這可能意味著在導航層次結構中向上或返回，或者關閉模態用戶界面。如果選定的元素沒有 `onAccessibilityEscape` 函數，系統將嘗試向上遍歷視圖層次結構，直到找到具有該函數的視圖，或者發出「bonk」聲表示找不到。

### `onAccessibilityTap` <div class="label ios">iOS</div>

使用此屬性分配一個自定義函數，當有人在選中可訪問元素時雙擊它時，該函數將被調用。

### `onMagicTap` <div class="label ios">iOS</div>

將此屬性分配給一個自定義函數，當有人執行「magic tap」手勢（即兩指雙擊）時，該函數將被調用。magic tap 函數應執行用戶在組件上可以採取的最相關操作。在 iPhone 的電話應用中，magic tap 可以接聽電話或結束當前通話。如果選定的元素沒有 `onMagicTap` 函數，系統將向上遍歷視圖層次結構，直到找到具有該函數的視圖。

### `role`

`role` 傳達組件的用途，並優先於 [`accessibilityRole`](accessibility#accessibilityrole) 屬性。

`role` 可以是以下之一：

- **alert** 當元素包含需要向用戶展示的重要文字時使用。
- **button** 當元素應被視為按鈕時使用。
- **checkbox** 當元素代表可勾選、取消勾選或處於混合勾選狀態的核取方塊時使用。
- **combobox** 當元素代表組合框，允許用戶從多個選項中選擇時使用。
- **grid** 與 ScrollView、VirtualizedList、FlatList 或 SectionList 一起使用時，代表網格。為 Android 的 GridView 添加進出網格的語音提示。
- **heading** 當元素作為內容區段的標題時使用（例如導覽列的標題）。
- **img** 當元素應被視為圖片時使用。可與按鈕或連結等結合使用。
- **link** 當元素應被視為連結時使用。
- **list** 用於識別項目列表。
- **listitem** 用於識別列表中的單一項目。
- **menu** 當組件是選單時使用。
- **menubar** 當組件是多個選單的容器時使用。
- **menuitem** 用於代表選單中的項目。
- **none** 當元素沒有角色時使用。
- **presentation** 當元素沒有角色時使用。
- **progressbar** 用於代表顯示任務進度的組件。
- **radio** 用於代表單選按鈕。
- **radiogroup** 用於代表一組單選按鈕。
- **scrollbar** 用於代表捲軸。
- **searchbox** 當文字輸入框應同時被視為搜尋欄位時使用。
- **slider** 當元素可被「調整」時使用（例如滑桿）。
- **spinbutton** 用於代表可展開選項列表的按鈕。
- **summary** 當元素可用於在應用程式首次啟動時提供當前條件的快速摘要時使用。
- **switch** 用於代表可開啟或關閉的開關。
- **tab** 用於代表標籤頁。
- **tablist** 用於代表標籤頁列表。
- **timer** 用於代表計時器。
- **toolbar** 用於代表工具列（動作按鈕或組件的容器）。

## 無障礙操作

無障礙操作允許輔助技術以程式化的方式觸發組件的操作。要支援無障礙操作，組件必須完成兩件事：

- 透過 `accessibilityActions` 屬性定義其支援的操作列表。
- 實作 `onAccessibilityAction` 函數以處理操作請求。

The `accessibilityActions` property should contain a list of action objects. Each action object should contain the following fields:

| Name  | Type   | Required |
| ----- | ------ | -------- |
| name  | string | Yes      |
| label | string | No       |

操作可以是標準操作（例如點擊按鈕或調整滑桿），也可以是特定組件的自訂操作（例如刪除電子郵件）。`name` 欄位對於標準和自訂操作都是必需的，但 `label` 對於標準操作是可選的。

當添加對標準操作的支援時，`name` 必須是以下之一：

- `'magicTap'` - 僅限 iOS - 當 VoiceOver 焦點位於元件內或元件上時，使用者用兩指雙擊。
- `'escape'` - 僅限 iOS - 當 VoiceOver 焦點位於元件內或元件上時，使用者執行兩指擦除手勢（左、右、左）。
- `'activate'` - 啟動元件。無論是否使用輔助技術，都應執行相同的操作。當螢幕閱讀器使用者雙擊元件時觸發。
- `'increment'` - 增加可調整元件。在 iOS 上，當元件具有 `'adjustable'` 角色且使用者將焦點放在其上並向上滑動時，VoiceOver 會生成此操作。在 Android 上，當使用者將輔助功能焦點放在元件上並按下音量增大按鈕時，TalkBack 會生成此操作。
- `'decrement'` - 減少可調整元件。在 iOS 上，當元件具有 `'adjustable'` 角色且使用者將焦點放在其上並向下滑動時，VoiceOver 會生成此操作。在 Android 上，當使用者將輔助功能焦點放在元件上並按下音量減小按鈕時，TalkBack 會生成此操作。
- `'longpress'` - 僅限 Android - 當使用者將輔助功能焦點放在元件上，然後雙擊並按住一根手指在螢幕上時，會生成此操作。無論是否使用輔助技術，都應執行相同的操作。

The `label` field is optional for standard actions and is often unused by assistive technologies. For custom actions, it is a localized string containing a description of the action to be presented to the user.

要處理操作請求，元件必須實現 `onAccessibilityAction` 函數。此函數的唯一參數是一個事件，其中包含要執行的操作名稱。以下來自 RNTester 的示例展示了如何創建一個定義並處理多個自定義操作的元件。

```tsx
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

Sometimes it is useful to trigger an accessibility event on a UI component (i.e. when a custom view appears on a screen or set accessibility focus to a view). Native UIManager module exposes a method ‘sendAccessibilityEvent’ for this purpose. It takes two arguments: a view tag and a type of event. The supported event types are `typeWindowStateChanged`, `typeViewFocused`, and `typeViewClicked`.

```tsx
import {Platform, UIManager, findNodeHandle} from 'react-native';

if (Platform.OS === 'android') {
  UIManager.sendAccessibilityEvent(
    findNodeHandle(this),
    UIManager.AccessibilityEventTypes.typeViewFocused,
  );
}
```

## 測試 TalkBack 支持 <div class="label android">Android</div>

要啟用 TalkBack，請前往 Android 設備或模擬器上的「設定」應用。點擊「輔助功能」，然後點擊「TalkBack」。切換「使用服務」開關以啟用或停用它。

Android 模擬器默認未安裝 TalkBack。您可以通過 Google Play 商店在模擬器上安裝 TalkBack。請確保選擇安裝了 Google Play 商店的模擬器。這些在 Android Studio 中可用。

您可以使用音量鍵快捷方式來切換 TalkBack。要開啟音量鍵快捷方式，請前往「設定」應用，然後點擊「輔助功能」。在頂部，開啟音量鍵快捷方式。

要使用音量鍵快捷方式，按住兩個音量鍵 3 秒以啟動輔助工具。

此外，如果您願意，也可以通過命令行切換 TalkBack：

```shell
# disable
adb shell settings put secure enabled_accessibility_services com.android.talkback/com.google.android.marvin.talkback.TalkBackService

# enable
adb shell settings put secure enabled_accessibility_services com.google.android.marvin.talkback/com.google.android.marvin.talkback.TalkBackService
```

## 測試 VoiceOver 支持 <div class="label ios">iOS</div>

要在 iOS 或 iPadOS 設備上啟用 VoiceOver，請前往「設定」應用，點擊「一般」，然後點擊「輔助使用」。在那裡，您會找到許多工具，包括 VoiceOver，這些工具可以幫助使用者使設備更易用。要啟用 VoiceOver，請點擊「視覺」下的「VoiceOver」，然後切換頂部的開關。

在「輔助使用」設定的最底部，有一個「輔助使用快捷鍵」。您可以使用此功能通過三擊主畫面按鈕來切換 VoiceOver。

VoiceOver 無法透過模擬器使用，但你可以使用 Xcode 的「輔助功能檢查器」來透過應用程式使用 macOS 的 VoiceOver。請注意，最好還是使用實體裝置進行測試，因為 macOS 的 VoiceOver 可能會導致不同的體驗。

## 其他資源

- [打造無障礙的 React Native 應用程式](https://engineering.fb.com/ios/making-react-native-apps-accessible/)