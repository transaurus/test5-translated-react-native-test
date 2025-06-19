---
id: view
title: View
---

作為構建使用者介面最基本的元件，`View` 是一個支援 [flexbox](flexbox.md) 佈局、[樣式](style.md)、[觸控處理](handling-touches.md) 和 [無障礙功能](accessibility.md) 控制的容器。`View` 會直接對應到 React Native 運行平台的原生視圖，無論是 `UIView`、`<div>` 還是 `android.view` 等。

`View` 設計用於嵌套在其他視圖中，可以包含 0 到多個任意類型的子元件。

此範例創建了一個 `View`，其中包裝了兩個帶有顏色的方塊和一個文字元件，並以行排列且具有內邊距。

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

> `View` 建議與 [`StyleSheet`](style.md) 搭配使用以提升清晰度和性能，但也支援行內樣式。

### 合成觸控事件

對於 `View` 的回應屬性（例如 `onResponderMove`），傳遞給它們的合成觸控事件形式為 [PressEvent](pressevent)。

---

# 參考

## 屬性

---

### `accessibilityActions`

無障礙操作允許輔助技術以程式化的方式呼叫元件的操作。`accessibilityActions` 屬性應包含一個操作物件列表，每個操作物件應包含名稱和標籤欄位。

詳見 [無障礙功能指南](accessibility.md#accessibility-actions) 以獲取更多資訊。

| Type  |
| ----- |
| array |

---

### `accessibilityElementsHidden` <div class="label ios">iOS</div>

表示此無障礙元素中包含的無障礙元素是否隱藏。預設值為 `false`。

詳見 [無障礙功能指南](accessibility.md#accessibilityelementshidden-ios) 以獲取更多資訊。

| Type |
| ---- |
| bool |

---

### `accessibilityHint`

無障礙提示幫助使用者理解當他們對無障礙元素執行操作時會發生什麼，尤其是當該結果無法從無障礙標籤中明確得知時。

| Type   |
| ------ |
| string |

---

### `accessibilityLanguage` <div class="label ios">iOS</div>

表示螢幕閱讀器在使用者與元素互動時應使用的語言。應遵循 [BCP 47 規範](https://www.rfc-editor.org/info/bcp47)。

詳見 [iOS `accessibilityLanguage` 文檔](https://developer.apple.com/documentation/objectivec/nsobject/1615192-accessibilitylanguage) 以獲取更多資訊。

| Type   |
| ------ |
| string |

---

### `accessibilityIgnoresInvertColors` <div class="label ios">iOS</div>

表示當顏色反轉開啟時，此視圖是否應被反轉。值為 `true` 時，即使顏色反轉開啟，視圖也不會被反轉。

詳見 [無障礙功能指南](accessibility.md#accessibilityignoresinvertcolors) 以獲取更多資訊。

| Type |
| ---- |
| bool |

---

### `accessibilityLabel`

覆蓋螢幕閱讀器在使用者與元素互動時朗讀的文字。預設情況下，標籤是通過遍歷所有子元件並累積所有以空格分隔的 `Text` 節點來構建的。

| Type   |
| ------ |
| string |

---

### `accessibilityLiveRegion` <div class="label android">Android</div>

指示無障礙服務是否應在該視圖變更時通知使用者。僅適用於 Android API >= 19。可能的值：

- `'none'` - 無障礙服務不應宣佈此視圖的變更。
- `'polite'` - 無障礙服務應宣佈此視圖的變更。
- `'assertive'` - 無障礙服務應中斷正在進行的語音以立即宣佈此視圖的變更。

詳見 [Android `View` 文檔](https://developer.android.com/reference/android/view/View.html#attr_android:accessibilityLiveRegion)參考。

| Type                                |
| ----------------------------------- |
| enum('none', 'polite', 'assertive') |

---

### `accessibilityRole`

`accessibilityRole` 向輔助技術的使用者傳達組件的用途。

`accessibilityRole` 可以是以下之一：

- `'none'` - 當元素沒有角色時使用。
- `'button'` - 當元素應被視為按鈕時使用。
- `'link'` - 當元素應被視為連結時使用。
- `'search'` - 當文字輸入框元素也應被視為搜尋欄時使用。
- `'image'` - 當元素應被視為圖像時使用。可與按鈕或連結等組合使用。
- `'keyboardkey'` - 當元素充當鍵盤按鍵時使用。
- `'text'` - 當元素應被視為不可變的靜態文字時使用。
- `'adjustable'` - 當元素可被「調整」（例如滑桿）時使用。
- `'imagebutton'` - 當元素應被視為按鈕且同時是圖像時使用。
- `'header'` - 當元素充當內容區段的標題（例如導覽列的標題）時使用。
- `'summary'` - 當元素可用於在應用程式首次啟動時提供當前條件的快速摘要時使用。
- `'alert'` - 當元素包含需要向使用者展示的重要文字時使用。
- `'checkbox'` - 當元素代表可勾選、取消勾選或具有混合勾選狀態的核取方塊時使用。
- `'combobox'` - 當元素代表允許使用者在多個選項中選擇的組合框時使用。
- `'menu'` - 當組件是選單時使用。
- `'menubar'` - 當組件是多個選單的容器時使用。
- `'menuitem'` - 用於表示選單中的項目。
- `'progressbar'` - 用於表示指示任務進度的組件。
- `'radio'` - 用於表示單選按鈕。
- `'radiogroup'` - 用於表示一組單選按鈕。
- `'scrollbar'` - 用於表示捲軸。
- `'spinbutton'` - 用於表示開啟選項列表的按鈕。
- `'switch'` - 用於表示可開啟和關閉的開關。
- `'tab'` - 用於表示標籤頁。
- `'tablist'` - 用於表示標籤頁列表。
- `'timer'` - 用於表示計時器。
- `'toolbar'` - 用於表示工具列（動作按鈕或組件的容器）。
- `'grid'` - 與 ScrollView、VirtualizedList、FlatList 或 SectionList 一起使用以表示網格。向 android GridView 添加網格內/外的宣佈。

| Type   |
| ------ |
| string |

---

### `accessibilityState`

向輔助技術的使用者描述組件的當前狀態。

詳見 [無障礙指南](accessibility.md#accessibilitystate-ios-android)獲取更多資訊。

| Type                                                                                             |
| ------------------------------------------------------------------------------------------------ |
| object: `{disabled: bool, selected: bool, checked: bool or 'mixed', busy: bool, expanded: bool}` |

---

### `accessibilityValue`

表示組件的當前值。它可以是組件值的文字描述，或對於基於範圍的組件（如滑桿和進度條），它包含範圍資訊（最小值、當前值和最大值）。

詳見 [無障礙指南](accessibility.md#accessibilityvalue-ios-android)獲取更多資訊。

| Type                                                            |
| --------------------------------------------------------------- |
| object: `{min: number, max: number, now: number, text: string}` |

---

### `accessibilityViewIsModal` <div class="label ios">iOS</div>

表示 VoiceOver 是否應忽略接收器同層級視圖中的元素。預設值為 `false`。

詳見[無障礙功能指南](accessibility.md#accessibilityviewismodal-ios)獲取更多資訊。

| Type |
| ---- |
| bool |

---

### `accessible`

當設為 `true` 時，表示該視圖是一個無障礙元素，可被螢幕閱讀器和硬體鍵盤等輔助技術發現。預設情況下，所有可觸控元素都是可訪問的。

詳見[無障礙功能指南](accessibility.md#accessible)獲取更多資訊。

---

### `aria-busy`

表示元素正在被修改，輔助技術可能需要等待更改完成後再通知用戶更新。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-checked`

表示可勾選元素的狀態。此屬性可以接受布林值或 "mixed" 字串來表示混合狀態的複選框。

| Type             | Default |
| ---------------- | ------- |
| boolean, 'mixed' | false   |

---

### `aria-disabled`

表示元素是可感知的但已被禁用，因此不可編輯或操作。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-expanded`

表示可展開元素當前是展開還是折疊狀態。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-hidden`

表示此無障礙元素中包含的子元素是否被隱藏。

For example, in a window that contains sibling views `A` and `B`, setting `aria-hidden` to `true` on view `B` causes VoiceOver to ignore the elements in the view `B`.

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-label`

定義一個字串值，用於標記互動式元素。

| Type   |
| ------ |
| string |

---

### `aria-labelledby` <div class="label android">Android</div>

標識用於標記當前元素的關聯元素。`aria-labelledby` 的值應與關聯元素的 [`nativeID`](view.md#nativeid) 相匹配：

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

表示元素將被更新，並描述用戶代理、輔助技術和用戶可以從實時區域獲得的更新類型。

- **off** 輔助技術不應宣佈此視圖的變更。
- **polite** 輔助技術應宣佈此視圖的變更。
- **assertive** 輔助技術應中斷當前語音以立即宣佈此視圖的變更。

| Type                                     | Default |
| ---------------------------------------- | ------- |
| enum(`'assertive'`, `'off'`, `'polite'`) | `'off'` |

---

### `aria-modal` <div class="label ios">iOS</div>

布林值，表示 VoiceOver 是否應忽略接收器同層級視圖中的元素。此屬性優先於 [`accessibilityViewIsModal`](#accessibilityviewismodal-ios) 屬性。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-selected`

表示可選元素當前是否被選中。

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

僅用於佈局其子元件或不繪製任何內容的視圖可能會作為優化被自動從原生層級中移除。將此屬性設為 `false` 可禁用此優化，確保此 `View` 存在於原生視圖層級中。

| Type    | Default |
| ------- | ------- |
| boolean | true    |

---

### `collapsableChildren`

設為 false 可防止視圖的直接子元件從原生視圖層級中移除，效果類似於在每個子元件上設定 `collapsable={false}`。

| Type    | Default |
| ------- | ------- |
| boolean | true    |

---

### `focusable` <div class="label android">Android</div>

此 `View` 是否應可透過非觸控輸入設備（如硬體鍵盤）獲取焦點。

| Type    |
| ------- |
| boolean |

---

### `hitSlop`

定義觸控事件可以從視圖多遠的距離開始。典型的介面指南建議觸控目標至少為 30 - 40 點/密度無關像素。

For example, if a touchable view has a height of 20 the touchable height can be extended to 40 with `hitSlop={{top: 10, bottom: 10, left: 0, right: 0}}`

> 觸控區域永遠不會超出父視圖邊界，且如果觸控命中兩個重疊的視圖，兄弟視圖的 Z-index 永遠優先。

| Type                                                                 |
| -------------------------------------------------------------------- |
| object: `{top: number, left: number, bottom: number, right: number}` |

---

### `id`

用於從原生類別中定位此視圖。優先於 `nativeID` 屬性。

> 這會禁用此視圖的「僅佈局視圖移除」優化！

| Type   |
| ------ |
| string |

---

### `importantForAccessibility` <div class="label android">Android</div>

控制視圖對無障礙功能的重要性，即是否觸發無障礙事件以及是否報告給查詢螢幕的無障礙服務。僅適用於 Android。

可能的值：

- `'auto'` - 由系統決定該視圖是否對無障礙功能重要 - 預設值（推薦使用）。
- `'yes'` - 該視圖對無障礙功能重要。
- `'no'` - 該視圖對無障礙功能不重要。
- `'no-hide-descendants'` - 該視圖及其所有子視圖對無障礙功能都不重要。

詳見 [Android `importantForAccessibility` 文檔](https://developer.android.com/reference/android/R.attr.html#importantForAccessibility) 作為參考。

| Type                                             |
| ------------------------------------------------ |
| enum('auto', 'yes', 'no', 'no-hide-descendants') |

---

### `nativeID`

用於從原生類別中定位此視圖。

> 這會禁用該視圖的「僅佈局視圖移除」優化！

| Type   |
| ------ |
| string |

---

### `needsOffscreenAlphaCompositing`

此 `View` 是否需要離屏渲染並以 alpha 值合成，以保留 100% 正確的顏色和混合行為。預設值 (`false`) 會改為在繪製每個元素時，對使用的繪製工具應用 alpha 值來繪製組件及其子元素，而非將整個組件離屏渲染後再以 alpha 值合成。在您設置透明度的 `View` 包含多個重疊元素（例如多個重疊的 `View`，或文字與背景）的情況下，這種預設行為可能會引起注意且不符合預期。

離屏渲染以保留正確的 alpha 行為非常耗費資源且對非原生開發者難以調試，因此預設不啟用。如果您確實需要為動畫啟用此屬性，考慮將其與 renderToHardwareTextureAndroid 結合使用（如果視圖**內容**是靜態的，即不需要每幀重新繪製）。若啟用該屬性，此視圖將被離屏渲染一次，保存為硬體紋理，然後每幀以 alpha 值合成到屏幕上，無需在 GPU 上切換渲染目標。

| Type |
| ---- |
| bool |

---

### `nextFocusDown` <div class="label android">Android</div>

指定用戶向下導航時下一個獲得焦點的視圖。詳見 [Android 文檔](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusDown)。

| Type   |
| ------ |
| number |

---

### `nextFocusForward` <div class="label android">Android</div>

指定用戶向前導航時下一個獲得焦點的視圖。詳見 [Android 文檔](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusForward)。

| Type   |
| ------ |
| number |

---

### `nextFocusLeft` <div class="label android">Android</div>

指定用戶向左導航時下一個獲得焦點的視圖。詳見 [Android 文檔](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusLeft)。

| Type   |
| ------ |
| number |

---

### `nextFocusRight` <div class="label android">Android</div>

指定用戶向右導航時下一個獲得焦點的視圖。詳見 [Android 文檔](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusRight)。

| Type   |
| ------ |
| number |

---

### `nextFocusUp` <div class="label android">Android</div>

指定用戶向上導航時下一個獲得焦點的視圖。詳見 [Android 文檔](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusUp)。

| Type   |
| ------ |
| number |

---

### `onAccessibilityAction`

當使用者執行無障礙操作時觸發。此函數的唯一參數是一個包含要執行操作名稱的事件物件。

詳見[無障礙功能指南](accessibility.md#accessibility-actions)以獲取更多資訊。

| Type     |
| -------- |
| function |

---

### `onAccessibilityEscape` <div class="label ios">iOS</div>

當`accessible`設為`true`時，系統會在用戶執行「返回」手勢時調用此函數。

| Type     |
| -------- |
| function |

---

### `onAccessibilityTap` <div class="label ios">iOS</div>

當`accessible`為真時，系統會嘗試在用戶執行無障礙點擊手勢時調用此函數。

| Type     |
| -------- |
| function |

---

### `onLayout`

在組件掛載或佈局變更時觸發。

此事件會在佈局計算完成後立即觸發，但新的佈局可能尚未在屏幕上顯示，特別是在進行佈局動畫時。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)}) => void` |

---

### `onMagicTap` <div class="label ios">iOS</div>

當`accessible`設為`true`時，系統會在用戶執行「魔法點擊」手勢時調用此函數。

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

該視圖現已取得觸控事件響應權。此時應高亮顯示並向用戶反饋互動狀態。

在Android平台上，從此回調返回true可阻止其他原生組件在此響應者終止前取得響應權。

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

- `'auto'`：視圖可以成為觸摸事件的目標。
- `'none'`：視圖永遠不會成為觸摸事件的目標。
- `'box-none'`：視圖本身永遠不會成為觸摸事件的目標，但其子視圖可以。行為類似於在 CSS 中設置以下類別：

```css
.box-none {
  pointer-events: none;
}
.box-none * {
  pointer-events: auto;
}
```

- `'box-only'`：視圖可以成為觸摸事件的目標，但其子視圖不能。行為類似於在 CSS 中設置以下類別：

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

這是 `RCTView` 提供的一個保留性能屬性，對於包含許多子視圖且大多數子視圖位於屏幕外的滾動內容非常有用。要使此屬性生效，必須將其應用於包含許多超出其邊界的子視圖的視圖。子視圖還必須設置 `overflow: hidden`，包含視圖（或其父視圖之一）也應如此設置。

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

`role` 向輔助技術用戶傳達組件的用途。優先級高於 [`accessibilityRole`](view#accessibilityrole) 屬性。

| Type                       |
| -------------------------- |
| [Role](accessibility#role) |

---

### `shouldRasterizeIOS` <div class="label ios">iOS</div>

Whether this `View` should be rendered as a bitmap before compositing.

在 iOS 上，這對於不修改此組件尺寸或其子視圖的動畫和交互非常有用；例如，當平移靜態視圖的位置時，柵格化允許渲染器重用靜態視圖的緩存位圖，並在每一幀中快速合成。

柵格化會導致離屏繪製過程，且位圖會消耗內存。使用此屬性時應進行測試和測量。

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

> 這會停用此視圖的「僅佈局視圖移除」優化！

| Type   |
| ------ |
| string |