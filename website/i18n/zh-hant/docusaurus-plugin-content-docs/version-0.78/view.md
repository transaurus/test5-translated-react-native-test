---
id: view
title: View
---

作為構建使用者介面最基礎的元件，`View` 是一個支援 [flexbox](flexbox.md) 佈局、[樣式](style.md)、[觸控處理](handling-touches.md) 和 [無障礙功能](accessibility.md) 控制的容器。`View` 會直接對應到 React Native 運行平台的原生視圖，無論是 `UIView`、`<div>` 還是 `android.view` 等。

`View` 設計用於嵌套在其他視圖中，可以包含 0 到多個任意類型的子元件。

此範例建立了一個 `View`，其中包覆了兩個帶顏色的方塊和一個文字元件，並以行內排列且具有內邊距。

```SnackPlayer name=View%20Example
import React from 'react';
import {View, Text} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const ViewBoxesWithColorAndText = () => {
  return (
    <SafeAreaProvider>
      <SafeAreaView style={{height: 100, flexDirection: 'row'}}>
        <View style={{backgroundColor: 'blue', flex: 0.2}} />
        <View style={{backgroundColor: 'red', flex: 0.4}} />
        <Text>Hello World!</Text>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

export default ViewBoxesWithColorAndText;
```

> `View`s are designed to be used with [`StyleSheet`](style.md) for clarity and performance, although inline styles are also supported.

### 合成觸控事件

對於 `View` 的回應器屬性（例如 `onResponderMove`），傳遞給它們的合成觸控事件格式為 [PressEvent](pressevent)。

---

# 參考文獻

## 屬性

---

### `accessibilityActions`

無障礙操作允許輔助技術以程式化方式觸發元件的操作。`accessibilityActions` 屬性應包含一個操作物件列表，每個操作物件需包含名稱和標籤欄位。

詳見 [無障礙功能指南](accessibility.md#accessibility-actions)。

| Type  |
| ----- |
| array |

---

### `accessibilityElementsHidden` <div class="label ios">iOS</div>

表示此無障礙元素內包含的子元素是否隱藏，預設值為 `false`。

詳見 [無障礙功能指南](accessibility.md#accessibilityelementshidden-ios)。

| Type |
| ---- |
| bool |

---

### `accessibilityHint`

當無障礙標籤無法明確說明操作結果時，無障礙提示能幫助使用者理解觸發該元素操作後的預期行為。

| Type   |
| ------ |
| string |

---

### `accessibilityLanguage` <div class="label ios">iOS</div>

指定螢幕閱讀器與元素互動時應使用的語言，需符合 [BCP 47 規範](https://www.rfc-editor.org/info/bcp47)。

詳見 [iOS `accessibilityLanguage` 文件](https://developer.apple.com/documentation/objectivec/nsobject/1615192-accessibilitylanguage)。

| Type   |
| ------ |
| string |

---

### `accessibilityIgnoresInvertColors` <div class="label ios">iOS</div>

指定當色彩反轉功能啟用時，此視圖是否應被反轉。設為 `true` 表示即使啟用色彩反轉，該視圖也不會被反轉。

詳見 [無障礙功能指南](accessibility.md#accessibilityignoresinvertcolors)。

| Type |
| ---- |
| bool |

---

### `accessibilityLabel`

覆寫螢幕閱讀器在使用者與元素互動時朗讀的文字。預設情況下，標籤會透過遍歷所有子元件並以空格連接所有 `Text` 節點來生成。

| Type   |
| ------ |
| string |

---

### `accessibilityLiveRegion` <div class="label android">Android</div>

指示無障礙服務是否應在使用者檢視變更時發出通知。僅適用於 Android API >= 19。可能的值：

- `'none'` - 無障礙服務不應宣布此檢視的變更。
- `'polite'` - 無障礙服務應宣布此檢視的變更。
- `'assertive'` - 無障礙服務應中斷正在進行的語音，立即宣布此檢視的變更。

參考 [Android `View` 文件](https://developer.android.com/reference/android/view/View.html#attr_android:accessibilityLiveRegion)。

| Type                                |
| ----------------------------------- |
| enum('none', 'polite', 'assertive') |

---

### `accessibilityRole`

`accessibilityRole` 向輔助技術的使用者傳達元件的用途。

`accessibilityRole` 可以是以下之一：

- `'none'` - 當元素沒有角色時使用。
- `'button'` - 當元素應被視為按鈕時使用。
- `'link'` - 當元素應被視為連結時使用。
- `'search'` - 當文字欄位元素也應被視為搜尋欄位時使用。
- `'image'` - 當元素應被視為圖片時使用。例如，可以與按鈕或連結結合使用。
- `'keyboardkey'` - 當元素作為鍵盤按鍵時使用。
- `'text'` - 當元素應被視為無法變更的靜態文字時使用。
- `'adjustable'` - 當元素可以「調整」時使用（例如滑桿）。
- `'imagebutton'` - 當元素應被視為按鈕且同時是圖片時使用。
- `'header'` - 當元素作為內容區段的標題時使用（例如導覽列的標題）。
- `'summary'` - 當元素可用於在應用程式首次啟動時提供當前條件的快速摘要時使用。
- `'alert'` - 當元素包含需要向使用者展示的重要文字時使用。
- `'checkbox'` - 當元素代表可以勾選、取消勾選或具有混合勾選狀態的核取方塊時使用。
- `'combobox'` - 當元素代表組合框，允許使用者從多個選項中選擇時使用。
- `'menu'` - 當元件是選單時使用。
- `'menubar'` - 當元件是多個選單的容器時使用。
- `'menuitem'` - 用於表示選單中的項目。
- `'progressbar'` - 用於表示指示任務進度的元件。
- `'radio'` - 用於表示單選按鈕。
- `'radiogroup'` - 用於表示單選按鈕群組。
- `'scrollbar'` - 用於表示捲軸。
- `'spinbutton'` - 用於表示開啟選項清單的按鈕。
- `'switch'` - 用於表示可以開啟和關閉的開關。
- `'tab'` - 用於表示標籤頁。
- `'tablist'` - 用於表示標籤頁清單。
- `'timer'` - 用於表示計時器。
- `'toolbar'` - 用於表示工具列（動作按鈕或元件的容器）。
- `'grid'` - 與 ScrollView、VirtualizedList、FlatList 或 SectionList 一起使用，表示網格。向 android GridView 添加網格內/外公告。

| Type   |
| ------ |
| string |

---

### `accessibilityState`

向輔助技術的使用者描述元件的當前狀態。

詳見 [無障礙指南](accessibility.md#accessibilitystate-ios-android)。

| Type                                                                                             |
| ------------------------------------------------------------------------------------------------ |
| object: `{disabled: bool, selected: bool, checked: bool or 'mixed', busy: bool, expanded: bool}` |

---

### `accessibilityValue`

表示元件的當前值。可以是元件值的文字描述，或對於範圍型元件（如滑桿和進度條），包含範圍資訊（最小值、當前值和最大值）。

詳見 [無障礙指南](accessibility.md#accessibilityvalue-ios-android)。

| Type                                                            |
| --------------------------------------------------------------- |
| object: `{min: number, max: number, now: number, text: string}` |

---

### `accessibilityViewIsModal` <div class="label ios">iOS</div>

A value indicating whether VoiceOver should ignore the elements within views that are siblings of the receiver. Default is `false`.

See the [Accessibility guide](accessibility.md#accessibilityviewismodal-ios) for more information.

| Type |
| ---- |
| bool |

---

### `accessible`

When `true`, indicates that the view is an accessibility element and discoverable by assistive technologies such as screen readers and hardware keyboards. By default, all the touchable elements are accessible.

See the [Accessibility guide](accessibility.md#accessible) for more information.

---

### `aria-busy`

Indicates an element is being modified and that assistive technologies may want to wait until the changes are complete before informing the user about the update.

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-checked`

Indicates the state of a checkable element. This field can either take a boolean or the "mixed" string to represent mixed checkboxes.

| Type             | Default |
| ---------------- | ------- |
| boolean, 'mixed' | false   |

---

### `aria-disabled`

Indicates that the element is perceivable but disabled, so it is not editable or otherwise operable.

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-expanded`

Indicates whether an expandable element is currently expanded or collapsed.

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-hidden`

Indicates whether the accessibility elements contained within this accessibility element are hidden.

For example, in a window that contains sibling views `A` and `B`, setting `aria-hidden` to `true` on view `B` causes VoiceOver to ignore the elements in the view `B`.

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-label`

Defines a string value that labels an interactive element.

| Type   |
| ------ |
| string |

---

### `aria-labelledby` <div class="label android">Android</div>

Identifies the element that labels the element it is applied to. The value of `aria-labelledby` should match the [`nativeID`](view.md#nativeid) of the related element:

```tsx
<View>
  <Text nativeID="formLabel">Label for Input Field</Text>
  <TextInput aria-label="input" aria-labelledby="formLabel" />
</View>
```

| Type   |
| ------ |
| string |

---

### `aria-live` <div class="label android">Android</div>

Indicates that an element will be updated, and describes the types of updates the user agents, assistive technologies, and user can expect from the live region.

- **off** Accessibility services should not announce changes to this view.
- **polite** Accessibility services should announce changes to this view.
- **assertive** Accessibility services should interrupt ongoing speech to immediately announce changes to this view.

| Type                                     | Default |
| ---------------------------------------- | ------- |
| enum(`'assertive'`, `'off'`, `'polite'`) | `'off'` |

---

### `aria-modal` <div class="label ios">iOS</div>

Boolean value indicating whether VoiceOver should ignore the elements within views that are siblings of the receiver. Has precedence over the [`accessibilityViewIsModal`](#accessibilityviewismodal-ios) prop.

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-selected`

指示可選元素當前是否被選中。

| Type    |
| ------- |
| boolean |

### `aria-valuemax`

Represents the maximum value for range-based components, such as sliders and progress bars. Has precedence over the `max` value in the `accessibilityValue` prop.

| Type   |
| ------ |
| number |

---

### `aria-valuemin`

Represents the minimum value for range-based components, such as sliders and progress bars. Has precedence over the `min` value in the `accessibilityValue` prop.

| Type   |
| ------ |
| number |

---

### `aria-valuenow`

Represents the current value for range-based components, such as sliders and progress bars. Has precedence over the `now` value in the `accessibilityValue` prop.

| Type   |
| ------ |
| number |

---

### `aria-valuetext`

Represents the textual description of the component. Has precedence over the `text` value in the `accessibilityValue` prop.

| Type   |
| ------ |
| string |

---

### `collapsable`

僅用於佈局其子元件或不繪製任何內容的檢視可能會作為優化自動從原生層級結構中移除。將此屬性設為 `false` 可禁用此優化，並確保此 `View` 存在於原生檢視層級結構中。

| Type    | Default |
| ------- | ------- |
| boolean | true    |

---

### `collapsableChildren`

設為 false 可防止檢視的直接子元件從原生檢視層級結構中移除，類似於對每個子元件設定 `collapsable={false}` 的效果。

| Type    | Default |
| ------- | ------- |
| boolean | true    |

---

### `focusable` <div class="label android">Android</div>

此 `View` 是否應可透過非觸控輸入設備（例如硬體鍵盤）獲得焦點。

| Type    |
| ------- |
| boolean |

---

### `hitSlop`

定義觸控事件可以從檢視多遠的位置開始。典型的介面指南建議觸控目標至少為 30 - 40 點/密度無關像素。

For example, if a touchable view has a height of 20 the touchable height can be extended to 40 with `hitSlop={{top: 10, bottom: 10, left: 0, right: 0}}`

> 觸控區域永遠不會超出父檢視邊界，且如果觸控命中兩個重疊的檢視，兄弟檢視的 Z-index 始終優先。

| Type                                                                 |
| -------------------------------------------------------------------- |
| object: `{top: number, left: number, bottom: number, right: number}` |

---

### `id`

用於從原生類別定位此檢視。優先於 `nativeID` 屬性。

> 這會禁用此檢視的「僅佈局檢視移除」優化！

| Type   |
| ------ |
| string |

---

### `importantForAccessibility` <div class="label android">Android</div>

控制檢視對無障礙的重要性，即是否觸發無障礙事件以及是否報告給查詢螢幕的無障礙服務。僅適用於 Android。

可能的值：

- `'auto'` - 由系統自動判斷該視圖是否對無障礙功能重要 - 預設值（推薦使用）。
- `'yes'` - 該視圖對無障礙功能重要。
- `'no'` - 該視圖對無障礙功能不重要。
- `'no-hide-descendants'` - 該視圖及其所有子視圖對無障礙功能都不重要。

See the [Android `importantForAccessibility` docs](https://developer.android.com/reference/android/R.attr.html#importantForAccessibility) for reference.

| Type                                             |
| ------------------------------------------------ |
| enum('auto', 'yes', 'no', 'no-hide-descendants') |

---

### `nativeID`

用於從原生類別中定位此視圖。

> 這會停用該視圖的「僅佈局視圖移除」優化功能！

| Type   |
| ------ |
| string |

---

### `needsOffscreenAlphaCompositing`

Whether this `View` needs to rendered offscreen and composited with an alpha in order to preserve 100% correct colors and blending behavior. The default (`false`) falls back to drawing the component and its children with an alpha applied to the paint used to draw each element instead of rendering the full component offscreen and compositing it back with an alpha value. This default may be noticeable and undesired in the case where the `View` you are setting an opacity on has multiple overlapping elements (e.g. multiple overlapping `View`s, or text and a background).

為保持正確的 alpha 行為而進行離屏渲染的成本極高，且對非原生開發者難以除錯，因此預設不啟用此功能。若您確實需要為動畫啟用此屬性，請考慮在視圖**內容**為靜態時（即不需要每幀重繪）與 renderToHardwareTextureAndroid 搭配使用。若啟用該屬性，此視圖將被離屏渲染一次並儲存為硬體紋理，之後每幀會直接以 alpha 值合成到畫面上，無需切換 GPU 的渲染目標。

| Type |
| ---- |
| bool |

---

### `nextFocusDown` <div class="label android">Android</div>

指定使用者向下導航時下一個獲得焦點的視圖。詳見 [Android 官方文件](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusDown)。

| Type   |
| ------ |
| number |

---

### `nextFocusForward` <div class="label android">Android</div>

指定使用者向前導航時下一個獲得焦點的視圖。詳見 [Android 官方文件](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusForward)。

| Type   |
| ------ |
| number |

---

### `nextFocusLeft` <div class="label android">Android</div>

指定使用者向左導航時下一個獲得焦點的視圖。詳見 [Android 官方文件](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusLeft)。

| Type   |
| ------ |
| number |

---

### `nextFocusRight` <div class="label android">Android</div>

指定使用者向右導航時下一個獲得焦點的視圖。詳見 [Android 官方文件](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusRight)。

| Type   |
| ------ |
| number |

---

### `nextFocusUp` <div class="label android">Android</div>

指定使用者向上導航時下一個獲得焦點的視圖。詳見 [Android 官方文件](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusUp)。

| Type   |
| ------ |
| number |

---

### `onAccessibilityAction`

當用戶執行無障礙操作時調用此函數。該函數的唯一參數是一個包含要執行操作名稱的事件對象。

詳見[無障礙功能指南](accessibility.md#accessibility-actions)獲取更多資訊。

| Type     |
| -------- |
| function |

---

### `onAccessibilityEscape` <div class="label ios">iOS</div>

當`accessible`設為`true`時，用戶執行逃逸手勢時系統將調用此函數。

| Type     |
| -------- |
| function |

---

### `onAccessibilityTap` <div class="label ios">iOS</div>

當`accessible`為true時，系統會在用戶執行無障礙點擊手勢時嘗試調用此函數。

| Type     |
| -------- |
| function |

---

### `onLayout`

在組件掛載和佈局變化時觸發。

該事件在佈局計算完成後立即觸發，但新的佈局可能尚未在屏幕上呈現，特別是在佈局動畫進行期間。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)}) => void` |

---

### `onMagicTap` <div class="label ios">iOS</div>

當`accessible`設為`true`時，用戶執行魔法點擊手勢時系統將調用此函數。

| Type     |
| -------- |
| function |

---

### `onMoveShouldSetResponder`

Does this view want to "claim" touch responsiveness? This is called for every touch move on the `View` when it is not the responder.

| Type                                                      |
| --------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => boolean` |

---

### `onMoveShouldSetResponderCapture`

If a parent `View` wants to prevent a child `View` from becoming responder on a move, it should have this handler which returns `true`.

| Type                                                      |
| --------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => boolean` |

---

### `onResponderGrant`

該視圖現在開始響應觸控事件。此時應高亮顯示並向用戶展示正在發生的操作。

在Android上，從此回調返回true可阻止其他原生組件在此響應者終止前成為響應者。

| Type                                                              |
| ----------------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void ｜ boolean` |

---

### `onResponderMove`

用戶正在移動手指。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onResponderReject`

Another responder is already active and will not release it to that `View` asking to be the responder.

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onResponderRelease`

在觸控結束時觸發。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onResponderTerminate`

The responder has been taken from the `View`. Might be taken by other views after a call to `onResponderTerminationRequest`, or might be taken by the OS without asking (e.g., happens with control center/ notification center on iOS)

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onResponderTerminationRequest`

Some other `View` wants to become responder and is asking this `View` to release its responder. Returning `true` allows its release.

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onStartShouldSetResponder`

此視圖是否希望在觸摸開始時成為響應者？

| Type                                                      |
| --------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => boolean` |

---

### `onStartShouldSetResponderCapture`

If a parent `View` wants to prevent a child `View` from becoming responder on a touch start, it should have this handler which returns `true`.

| Type                                                      |
| --------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => boolean` |

---

### `pointerEvents`

Controls whether the `View` can be the target of touch events.

- `'auto'`：視圖可成為觸摸事件的目標。
- `'none'`：視圖永遠不會成為觸摸事件的目標。
- `'box-none'`：視圖本身不會成為觸摸事件的目標，但其子視圖可以。行為類似CSS中的以下類別：

```css
.box-none {
  pointer-events: none;
}
.box-none * {
  pointer-events: auto;
}
```

- `'box-only'`：視圖可成為觸摸事件的目標，但其子視圖不能。行為類似CSS中的以下類別：

```css
.box-only {
  pointer-events: auto;
}
.box-only * {
  pointer-events: none;
}
```

| Type                                         |
| -------------------------------------------- |
| enum('box-none', 'none', 'box-only', 'auto') |

---

### `removeClippedSubviews`

這是 `RCTView` 提供的保留性能屬性，適用於包含大量子視圖且多數位於畫面外的滾動內容。要使此屬性生效，必須將其應用於包含超出邊界子視圖的容器視圖，且子視圖與容器視圖（或其父視圖）均需設置 `overflow: hidden`。

| Type |
| ---- |
| bool |

---

### `renderToHardwareTextureAndroid` <div class="label android">Android</div>

Whether this `View` should render itself (and all of its children) into a single hardware texture on the GPU.

On Android, this is useful for animations and interactions that only modify opacity, rotation, translation, and/or scale: in those cases, the view doesn't have to be redrawn and display lists don't need to be re-executed. The texture can be re-used and re-composited with different parameters. The downside is that this can use up limited video memory, so this prop should be set back to false at the end of the interaction/animation.

| Type |
| ---- |
| bool |

---

### `role`

`role` 向輔助技術使用者傳達組件的用途，其優先級高於 [`accessibilityRole`](view#accessibilityrole) 屬性。

| Type                       |
| -------------------------- |
| [Role](accessibility#role) |

---

### `shouldRasterizeIOS` <div class="label ios">iOS</div>

Whether this `View` should be rendered as a bitmap before compositing.

在iOS上，此屬性對於不修改組件尺寸或其子視圖的動畫與互動非常有用（例如平移靜態視圖位置時）。光柵化允許渲染器重用靜態視圖的快取點陣圖，並在每幀快速合成。

光柵化會產生離屏繪製過程且點陣圖消耗記憶體，使用時需測試與評估效能。

| Type |
| ---- |
| bool |

---

### `style`

| Type                           |
| ------------------------------ |
| [View Style](view-style-props) |

---

### `tabIndex` <div class="label android">Android</div>

Whether this `View` should be focusable with a non-touch input device, eg. receive focus with a hardware keyboard.
Supports the following values:

- `0` - 視圖可獲取焦點
- `-1` - 視圖不可獲取焦點

| Type        |
| ----------- |
| enum(0, -1) |

---

### `testID`

用於在端到端測試中定位此視圖。

> 這會禁用此視圖的「僅佈局視圖移除」優化！

| Type   |
| ------ |
| string |