---
id: view
title: View
---

作為構建使用者介面的最基礎元件，`View` 是一個支援 [flexbox](flexbox.md) 佈局、[樣式](style.md)、[觸控處理](handling-touches.md) 和 [無障礙功能](accessibility.md) 控制的容器。`View` 會直接對應到 React Native 運行平台的原生視圖，無論是 `UIView`、`<div>` 還是 `android.view` 等。

`View` 設計用於嵌套在其他視圖中，可以包含 0 到多個任意類型的子元件。

此範例建立了一個 `View`，其中包含兩個帶有顏色的方塊和一個文字元件，並以行排列方式加上內邊距。

```SnackPlayer name=View%20Example
import React from 'react';
import {View, Text} from 'react-native';

const ViewBoxesWithColorAndText = () => {
  return (
    <View
      style={{
        flexDirection: 'row',
        height: 100,
        padding: 20,
      }}>
      <View style={{backgroundColor: 'blue', flex: 0.3}} />
      <View style={{backgroundColor: 'red', flex: 0.5}} />
      <Text>Hello World!</Text>
    </View>
  );
};

export default ViewBoxesWithColorAndText;
```

> `View`s are designed to be used with [`StyleSheet`](style.md) for clarity and performance, although inline styles are also supported.

### 合成觸控事件

對於 `View` 的回應器屬性（例如 `onResponderMove`），傳遞給它們的合成觸控事件格式為 [PressEvent](pressevent)。

---

# 參考文件

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

表示此無障礙元素中包含的子無障礙元素是否隱藏。預設值為 `false`。

詳見 [無障礙功能指南](accessibility.md#accessibilityelementshidden-ios) 以獲取更多資訊。

| Type |
| ---- |
| bool |

---

### `accessibilityHint`

當操作結果無法從無障礙標籤中明確理解時，無障礙提示可幫助使用者理解對該元素執行操作後會發生的結果。

| Type   |
| ------ |
| string |

---

### `accessibilityLanguage` <div class="label ios">iOS</div>

指定螢幕閱讀器與元素互動時應使用的語言，需遵循 [BCP 47 規範](https://www.rfc-editor.org/info/bcp47)。

詳見 [iOS `accessibilityLanguage` 文件](https://developer.apple.com/documentation/objectivec/nsobject/1615192-accessibilitylanguage) 以獲取更多資訊。

| Type   |
| ------ |
| string |

---

### `accessibilityIgnoresInvertColors` <div class="label ios">iOS</div>

表示當顏色反轉功能開啟時，此視圖是否應被反轉。設為 `true` 時，即使開啟顏色反轉，視圖也不會被反轉。

詳見 [無障礙功能指南](accessibility.md#accessibilityignoresinvertcolors) 以獲取更多資訊。

| Type |
| ---- |
| bool |

---

### `accessibilityLabel`

覆寫螢幕閱讀器在使用者與元素互動時朗讀的文字。預設情況下，標籤會透過遍歷所有子元件並累積以空格分隔的 `Text` 節點來建構。

| Type   |
| ------ |
| string |

---

### `accessibilityLiveRegion` <div class="label android">Android</div>

向無障礙服務指示當此視圖變更時是否應通知使用者。僅適用於 Android API >= 19。可能的值包括：

- `'none'` - 無障礙服務不應宣佈此視圖的變更。
- `'polite'`- 無障礙服務應宣佈此視圖的變更。
- `'assertive'` - 無障礙服務應中斷正在進行的語音以立即宣佈此視圖的變更。

參考 [Android `View` 文檔](http://developer.android.com/reference/android/view/View.html#attr_android:accessibilityLiveRegion)。

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
- `'search'` - 當文字輸入元素也應被視為搜尋欄位時使用。
- `'image'` - 當元素應被視為圖像時使用。可以與按鈕或連結結合使用，例如。
- `'keyboardkey'` - 當元素充當鍵盤按鍵時使用。
- `'text'` - 當元素應被視為無法更改的靜態文字時使用。
- `'adjustable'` - 當元素可以「調整」時使用（例如滑動條）。
- `'imagebutton'` - 當元素應被視為按鈕且同時是圖像時使用。
- `'header'` - 當元素充當內容區段的標題時使用（例如導航欄的標題）。
- `'summary'` - 當元素可用於在應用首次啟動時提供當前條件的快速摘要時使用。
- `'alert'` - 當元素包含需要向使用者展示的重要文字時使用。
- `'checkbox'` - 當元素表示可以勾選、取消勾選或處於混合勾選狀態的核取方塊時使用。
- `'combobox'` - 當元素表示組合框，允許使用者在多個選項中選擇時使用。
- `'menu'` - 當組件是選單時使用。
- `'menubar'` - 當組件是多個選單的容器時使用。
- `'menuitem'` - 用於表示選單中的項目。
- `'progressbar'` - 用於表示指示任務進度的組件。
- `'radio'` - 用於表示單選按鈕。
- `'radiogroup'` - 用於表示一組單選按鈕。
- `'scrollbar'` - 用於表示滾動條。
- `'spinbutton'` - 用於表示打開選項列表的按鈕。
- `'switch'` - 用於表示可以開啟和關閉的開關。
- `'tab'` - 用於表示標籤頁。
- `'tablist'` - 用於表示標籤頁列表。
- `'timer'` - 用於表示計時器。
- `'toolbar'` - 用於表示工具欄（動作按鈕或組件的容器）。
- `'grid'` - 與 ScrollView、VirtualizedList、FlatList 或 SectionList 一起使用以表示網格。向 android GridView 添加網格內/外的宣佈。

| Type   |
| ------ |
| string |

---

### `accessibilityState`

向輔助技術的使用者描述組件的當前狀態。

詳見 [無障礙指南](accessibility.md#accessibilitystate-ios-android)。

| Type                                                                                             |
| ------------------------------------------------------------------------------------------------ |
| object: `{disabled: bool, selected: bool, checked: bool or 'mixed', busy: bool, expanded: bool}` |

---

### `accessibilityValue`

表示組件的當前值。它可以是組件值的文字描述，或者對於基於範圍的組件（如滑動條和進度條），它包含範圍信息（最小值、當前值和最大值）。

詳見 [無障礙指南](accessibility.md#accessibilityvalue-ios-android)。

| Type                                                            |
| --------------------------------------------------------------- |
| object: `{min: number, max: number, now: number, text: string}` |

---

### `accessibilityViewIsModal` <div class="label ios">iOS</div>

表示 VoiceOver 是否應忽略接收器同層級視圖中的元素。預設值為 `false`。

詳見 [無障礙功能指南](accessibility.md#accessibilityviewismodal-ios) 以獲取更多資訊。

| Type |
| ---- |
| bool |

---

### `accessible`

當設為 `true` 時，表示該視圖是一個無障礙元素。預設情況下，所有可觸控元素都具有無障礙功能。

---

### `aria-busy`

表示元素正在被修改，輔助技術可能需要等待變更完成後再通知使用者更新。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-checked`

表示可勾選元素的狀態。此屬性可接受布林值或 "mixed" 字串來表示混合狀態的核取方塊。

| Type             | Default |
| ---------------- | ------- |
| boolean, 'mixed' | false   |

---

### `aria-disabled`

表示元素可感知但處於禁用狀態，因此不可編輯或操作。

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

表示此無障礙元素內包含的無障礙元素是否隱藏。

For example, in a window that contains sibling views `A` and `B`, setting `aria-hidden` to `true` on view `B` causes VoiceOver to ignore the elements in the view `B`.

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-label`

定義標記互動元素的字串值。

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

表示元素將被更新，並描述使用者代理、輔助技術和使用者可以從動態區域預期的更新類型。

- **off** 無障礙服務不應宣佈此視圖的變更。
- **polite** 無障礙服務應宣佈此視圖的變更。
- **assertive** 無障礙服務應中斷當前語音以立即宣佈此視圖的變更。

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

### `collapsable` <div class="label android">Android</div>

僅用於佈局子元件或不繪製任何內容的視圖，可能會作為優化被自動從原生層級中移除。將此屬性設為 `false` 可禁用此優化，確保該 `View` 存在於原生視圖層級中。

| Type |
| ---- |
| bool |

---

### `focusable` <div class="label android">Android</div>

此 `View` 是否應可透過非觸控輸入設備（例如硬體鍵盤）獲取焦點。

| Type    |
| ------- |
| boolean |

---

### `hitSlop`

定義觸控事件可從視圖外多遠開始觸發。典型的介面指南建議觸控目標至少為 30-40 點/密度無關像素。

For example, if a touchable view has a height of 20 the touchable height can be extended to 40 with `hitSlop={{top: 10, bottom: 10, left: 0, right: 0}}`

> 觸控區域永遠不會超出父視圖邊界，且若觸控同時命中兩個重疊視圖，兄弟視圖的 Z-index 永遠優先。

| Type                                                                 |
| -------------------------------------------------------------------- |
| object: `{top: number, left: number, bottom: number, right: number}` |

---

### `id`

用於從原生類別定位此視圖。優先於 `nativeID` 屬性。

> 這會禁用此視圖的「僅佈局視圖移除」優化！

| Type   |
| ------ |
| string |

---

### `importantForAccessibility` <div class="label android">Android</div>

控制視圖對無障礙功能的重要性，即是否觸發無障礙事件及是否回報給查詢螢幕的無障礙服務。僅適用於 Android。

可能值：

- `'auto'` - 由系統決定視圖對無障礙功能是否重要 - 預設值（建議使用）。
- `'yes'` - 視圖對無障礙功能重要。
- `'no'` - 視圖對無障礙功能不重要。
- `'no-hide-descendants'` - 視圖及其所有子視圖對無障礙功能都不重要。

參考 [Android `importantForAccessibility` 文件](http://developer.android.com/reference/android/R.attr.html#importantForAccessibility)。

| Type                                             |
| ------------------------------------------------ |
| enum('auto', 'yes', 'no', 'no-hide-descendants') |

---

### `nativeID`

用於從原生類別定位此視圖。

> 這會停用此視圖的「僅佈局視圖移除」優化！

| Type   |
| ------ |
| string |

---

### `needsOffscreenAlphaCompositing`

此 `View` 是否需要離屏渲染並以 alpha 值合成，以保留 100% 正確的色彩和混合行為。預設值 (`false`) 會改為在繪製每個元素時對繪製使用的顏料套用 alpha 值，而非將整個元件離屏渲染後再以 alpha 值合成回來。當您設定透明度的 `View` 包含多個重疊元素（例如多個重疊的 `View` 或文字與背景）時，此預設行為可能會產生明顯且非預期的效果。

Rendering offscreen to preserve correct alpha behavior is extremely expensive and hard to debug for non-native developers, which is why it is not turned on by default. If you do need to enable this property for an animation, consider combining it with renderToHardwareTextureAndroid if the view **contents** are static (i.e. it doesn't need to be redrawn each frame). If that property is enabled, this View will be rendered off-screen once, saved in a hardware texture, and then composited onto the screen with an alpha each frame without having to switch rendering targets on the GPU.

| Type |
| ---- |
| bool |

---

### `nextFocusDown` <div class="label android">Android</div>

指定使用者向下導航時下一個獲得焦點的視圖。參閱 [Android 文件](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusDown)。

| Type   |
| ------ |
| number |

---

### `nextFocusForward` <div class="label android">Android</div>

指定使用者向前導航時下一個獲得焦點的視圖。參閱 [Android 文件](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusForward)。

| Type   |
| ------ |
| number |

---

### `nextFocusLeft` <div class="label android">Android</div>

指定使用者向左導航時下一個獲得焦點的視圖。參閱 [Android 文件](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusLeft)。

| Type   |
| ------ |
| number |

---

### `nextFocusRight` <div class="label android">Android</div>

指定使用者向右導航時下一個獲得焦點的視圖。參閱 [Android 文件](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusRight)。

| Type   |
| ------ |
| number |

---

### `nextFocusUp` <div class="label android">Android</div>

指定使用者向上導航時下一個獲得焦點的視圖。參閱 [Android 文件](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusUp)。

| Type   |
| ------ |
| number |

---

### `onAccessibilityAction`

當使用者執行無障礙操作時觸發。此函數的唯一參數是一個包含要執行操作名稱的事件。

詳見 [無障礙指南](accessibility.md#accessibility-actions) 以獲取更多資訊。

| Type     |
| -------- |
| function |

---

### `onAccessibilityEscape` <div class="label ios">iOS</div>

當 `accessible` 為 `true` 時，系統會在使用者執行 escape 手勢時呼叫此函數。

| Type     |
| -------- |
| function |

---

### `onAccessibilityTap` <div class="label ios">iOS</div>

當 `accessible` 設為 true 時，系統將在使用者執行無障礙點擊手勢時嘗試調用此函數。

| Type     |
| -------- |
| function |

---

### `onLayout`

在掛載時和佈局變更時觸發。

This event is fired immediately once the layout has been calculated, but the new layout may not yet be reflected on the screen at the time the event is received, especially if a layout animation is in progress.

| Type                                                     |
| -------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)}) => void` |

---

### `onMagicTap` <div class="label ios">iOS</div>

當 `accessible` 設為 `true` 時，系統會在使用者執行 magic tap 手勢時調用此函數。

| Type     |
| -------- |
| function |

---

### `onMoveShouldSetResponder`

此視圖是否要「聲明」觸控回應權？當該 `View` 不是當前回應者時，每次觸控移動都會觸發此判斷。

| Type                                                      |
| --------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => boolean` |

---

### `onMoveShouldSetResponderCapture`

若父 `View` 欲阻止子 `View` 在移動時成為回應者，應設置此處理函數並返回 `true`。

| Type                                                      |
| --------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => boolean` |

---

### `onResponderGrant`

該視圖現已取得觸控事件回應權。此時應高亮顯示並向使用者反饋互動狀態。

在 Android 上，從此回調返回 true 可阻止其他原生組件在此回應者終止前取得控制權。

| Type                                                              |
| ----------------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void ｜ boolean` |

---

### `onResponderMove`

使用者正在移動手指。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onResponderReject`

已有其他回應者處於活動狀態，且不願釋放控制權給請求成為回應者的該 `View`。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onResponderRelease`

觸控結束時觸發。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onResponderTerminate`

回應者權限已被從該 `View` 收回。可能因 `onResponderTerminationRequest` 調用後被其他視圖取得，也可能被系統直接收回（例如 iOS 控制中心/通知中心觸發時）。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onResponderTerminationRequest`

其他 `View` 請求成為回應者，並要求當前 `View` 釋放權限。返回 `true` 表示允許釋放。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onStartShouldSetResponder`

該視圖是否要在觸控開始時成為回應者？

| Type                                                      |
| --------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => boolean` |

---

### `onStartShouldSetResponderCapture`

若父 `View` 欲阻止子 `View` 在觸控開始時成為回應者，應設置此處理函數並返回 `true`。

| Type                                                      |
| --------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => boolean` |

---

### `pointerEvents`

控制該 `View` 是否能成為觸控事件的目標。

- `'auto'`: 該視圖可以成為觸控事件的目標。
- `'none'`: 該視圖永遠不會成為觸控事件的目標。
- `'box-none'`: 該視圖本身不會成為觸控事件的目標，但其子視圖可以。其行為類似於在 CSS 中為該視圖添加以下類別：

```css
.box-none {
  pointer-events: none;
}
.box-none * {
  pointer-events: auto;
}
```

- `'box-only'`: 該視圖可以成為觸控事件的目標，但其子視圖不能。其行為類似於在 CSS 中為該視圖添加以下類別：

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

這是一個由 `RCTView` 提供的保留性能屬性，對於包含大量子視圖且多數位於畫面外的滾動內容非常有用。要使此屬性生效，必須將其應用於包含許多超出其邊界的子視圖的視圖上。子視圖必須具有 `overflow: hidden` 樣式，包含視圖（或其某個父視圖）也應如此。

| Type |
| ---- |
| bool |

---

### `renderToHardwareTextureAndroid` <div class="label android">Android</div>

此 `View` 是否應將自身（及其所有子視圖）渲染為 GPU 上的單一硬體紋理。

On Android, this is useful for animations and interactions that only modify opacity, rotation, translation, and/or scale: in those cases, the view doesn't have to be redrawn and display lists don't need to be re-executed. The texture can be re-used and re-composited with different parameters. The downside is that this can use up limited video memory, so this prop should be set back to false at the end of the interaction/animation.

| Type |
| ---- |
| bool |

---

### `role`

`role` 向輔助技術的使用者傳達組件的用途。其優先級高於 [`accessibilityRole`](view#accessibilityrole) 屬性。

| Type                       |
| -------------------------- |
| [Role](accessibility#role) |

---

### `shouldRasterizeIOS` <div class="label ios">iOS</div>

此 `View` 是否應在合成前渲染為點陣圖。

在 iOS 上，這對於不修改此組件尺寸或其子視圖的動畫和互動非常有用；例如，當平移靜態視圖的位置時，點陣化允許渲染器重用靜態視圖的快取點陣圖，並在每一幀中快速合成。

點陣化會導致一次離屏繪製過程，且點陣圖會消耗記憶體。使用此屬性時應進行測試和測量。

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

此 `View` 是否應可透過非觸控輸入設備（例如硬體鍵盤）獲得焦點。
支援以下值：

- `0` - 視圖可獲得焦點
- `-1` - 視圖不可獲得焦點

| Type        |
| ----------- |
| enum(0, -1) |

---

### `testID`

用於在端到端測試中定位此視圖。

> 這會禁用對此視圖的「僅佈局視圖移除」優化！

| Type   |
| ------ |
| string |