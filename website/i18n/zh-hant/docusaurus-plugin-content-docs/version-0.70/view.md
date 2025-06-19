---
id: view
title: View
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

作為構建使用者介面最基本的元件，`View` 是一個支援 [flexbox](flexbox.md) 佈局、[樣式](style.md)、[觸控處理](handling-touches.md) 和 [無障礙功能](accessibility.md) 控制的容器。`View` 會直接對應到 React Native 運行平台的原生視圖，無論是 `UIView`、`<div>` 還是 `android.view` 等。

`View` 設計用於嵌套在其他視圖中，可以包含 0 到多個任意類型的子元件。

此範例創建了一個 `View`，其中包含兩個帶有顏色的方塊和一個文字元件，並以行排列並添加內邊距。

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=View%20Function%20Component%20Example
import React from "react";
import { View, Text } from "react-native";

const ViewBoxesWithColorAndText = () => {
  return (
    <View
      style={{
        flexDirection: "row",
        height: 100,
        padding: 20
      }}
    >
      <View style={{ backgroundColor: "blue", flex: 0.3 }} />
      <View style={{ backgroundColor: "red", flex: 0.5 }} />
      <Text>Hello World!</Text>
    </View>
  );
};

export default ViewBoxesWithColorAndText;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=View%20Class%20Component%20Example
import React, { Component } from "react";
import { View, Text } from "react-native";

class App extends Component {
  render() {
    return (
      <View
        style={{
          flexDirection: "row",
          height: 100,
          padding: 20
        }}
      >
        <View style={{ backgroundColor: "blue", flex: 0.3 }} />
        <View style={{ backgroundColor: "red", flex: 0.5 }} />
        <Text>Hello World!</Text>
      </View>
    );
  }
}

export default App;
```

</TabItem>
</Tabs>

> `View`s are designed to be used with [`StyleSheet`](style.md) for clarity and performance, although inline styles are also supported.

### 合成觸控事件

對於 `View` 的回應器屬性（例如 `onResponderMove`），傳遞給它們的合成觸控事件以 [PressEvent](pressevent) 的形式呈現。

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

當無障礙標籤無法清楚表達操作結果時，無障礙提示可幫助使用者理解在無障礙元素上執行操作時會發生什麼。

| Type   |
| ------ |
| string |

---

### `accessibilityLanguage` <div class="label ios">iOS</div>

表示螢幕閱讀器在使用者與元素互動時應使用的語言。應遵循 [BCP 47 規範](https://www.rfc-editor.org/info/bcp47)。

詳見 [iOS `accessibilityLanguage` 文件](https://developer.apple.com/documentation/objectivec/nsobject/1615192-accessibilitylanguage) 以獲取更多資訊。

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

覆寫螢幕閱讀器在使用者與元素互動時朗讀的文字。預設情況下，標籤是通過遍歷所有子元件並累積所有以空格分隔的 `Text` 節點來構建的。

| Type   |
| ------ |
| string |

---

### `accessibilityLiveRegion` <div class="label android">Android</div>

指示輔助服務是否應在使用者變更此視圖時通知使用者。僅適用於 Android API >= 19。可能的值：

- `'none'` - 輔助服務不應宣佈此視圖的變更。
- `'polite'` - 輔助服務應宣佈此視圖的變更。
- `'assertive'` - 輔助服務應中斷正在進行的語音，立即宣佈此視圖的變更。

詳見 [Android `View` 文檔](http://developer.android.com/reference/android/view/View.html#attr_android:accessibilityLiveRegion)。

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
- `'image'` - 當元素應被視為圖片時使用。可以與按鈕或連結結合使用。
- `'keyboardkey'` - 當元素作為鍵盤按鍵時使用。
- `'text'` - 當元素應被視為無法變更的靜態文字時使用。
- `'adjustable'` - 當元素可以「調整」時使用（例如滑桿）。
- `'imagebutton'` - 當元素應被視為按鈕且同時是圖片時使用。
- `'header'` - 當元素作為內容區段的標題時使用（例如導覽列的標題）。
- `'summary'` - 當元素可用於提供應用程式啟動時當前條件的快速摘要時使用。
- `'alert'` - 當元素包含需要向使用者展示的重要文字時使用。
- `'checkbox'` - 當元素代表可勾選、取消勾選或混合勾選狀態的核取方塊時使用。
- `'combobox'` - 當元素代表組合框，允許使用者從多個選項中選擇時使用。
- `'menu'` - 當組件是選單時使用。
- `'menubar'` - 當組件是多個選單的容器時使用。
- `'menuitem'` - 用於代表選單中的項目。
- `'progressbar'` - 用於代表指示任務進度的組件。
- `'radio'` - 用於代表單選按鈕。
- `'radiogroup'` - 用於代表一組單選按鈕。
- `'scrollbar'` - 用於代表捲軸。
- `'spinbutton'` - 用於代表開啟選項列表的按鈕。
- `'switch'` - 用於代表可開啟和關閉的開關。
- `'tab'` - 用於代表標籤頁。
- `'tablist'` - 用於代表標籤頁列表。
- `'timer'` - 用於代表計時器。
- `'toolbar'` - 用於代表工具列（動作按鈕或組件的容器）。

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

代表組件的當前值。可以是組件值的文字描述，或對於範圍型組件（如滑桿和進度條），包含範圍資訊（最小值、當前值和最大值）。

詳見 [無障礙指南](accessibility.md#accessibilityvalue-ios-android)。

| Type                                                            |
| --------------------------------------------------------------- |
| object: `{min: number, max: number, now: number, text: string}` |

---

### `accessibilityViewIsModal` <div class="label ios">iOS</div>

表示 VoiceOver 是否應忽略接收器同層級視圖中的元素。預設值為 `false`。

詳見[無障礙功能指南](accessibility.md#accessibilityviewismodal-ios)以獲取更多資訊。

| Type |
| ---- |
| bool |

---

### `accessible`

當設為 `true` 時，表示該視圖是一個無障礙元素。預設情況下，所有可觸控元素均具有無障礙特性。

---

### `collapsable` <div class="label android">Android</div>

僅用於佈局子元素或不繪製任何內容的視圖，可能會作為優化被自動從原生層級結構中移除。將此屬性設為 `false` 可禁用此優化，確保該 `View` 存在於原生視圖層級中。

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

定義觸控事件可從視圖外多遠開始觸發。典型的介面設計指南建議觸控目標至少為 30-40 點/與密度無關的像素。

For example, if a touchable view has a height of 20 the touchable height can be extended to 40 with `hitSlop={{top: 10, bottom: 10, left: 0, right: 0}}`

> 觸控區域永遠不會超出父視圖邊界，且當觸控點擊兩個重疊視圖時，兄弟視圖的 Z-index 永遠優先。

| Type                                                                 |
| -------------------------------------------------------------------- |
| object: `{top: number, left: number, bottom: number, right: number}` |

---

### `importantForAccessibility` <div class="label android">Android</div>

控制視圖對無障礙功能的重要性，即是否觸發無障礙事件及是否被查詢螢幕的無障礙服務回報。僅適用於 Android。

可能的值：

- `'auto'` - 由系統決定視圖對無障礙功能的重要性 - 預設值（建議使用）。
- `'yes'` - 視圖對無障礙功能重要。
- `'no'` - 視圖對無障礙功能不重要。
- `'no-hide-descendants'` - 視圖及其所有子視圖對無障礙功能均不重要。

參見 [Android `importantForAccessibility` 文檔](http://developer.android.com/reference/android/R.attr.html#importantForAccessibility)以獲取參考。

| Type                                             |
| ------------------------------------------------ |
| enum('auto', 'yes', 'no', 'no-hide-descendants') |

---

### `nativeID`

用於從原生類別中定位此視圖。

> 這會禁用此視圖的「僅佈局視圖移除」優化！

| Type   |
| ------ |
| string |

---

### `needsOffscreenAlphaCompositing`

此 `View` 是否需要離屏渲染並透過 alpha 合成以保留 100% 正確的色彩與混合行為。預設值 (`false`) 會改為在繪製每個元素時對繪製使用的顏料套用 alpha 值，而非將整個元件離屏渲染後再透過 alpha 值合成回來。當你設定透明度的 `View` 含有多個重疊元素（例如多個重疊的 `View`，或文字與背景）時，此預設行為可能會引起明顯且非預期的效果。

Rendering offscreen to preserve correct alpha behavior is extremely expensive and hard to debug for non-native developers, which is why it is not turned on by default. If you do need to enable this property for an animation, consider combining it with renderToHardwareTextureAndroid if the view **contents** are static (i.e. it doesn't need to be redrawn each frame). If that property is enabled, this View will be rendered off-screen once, saved in a hardware texture, and then composited onto the screen with an alpha each frame without having to switch rendering targets on the GPU.

| Type |
| ---- |
| bool |

---

### `nextFocusDown` <div class="label android">Android</div>

指定當用戶向下導航時下一個獲得焦點的視圖。參閱 [Android 官方文檔](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusDown)。

| Type   |
| ------ |
| number |

---

### `nextFocusForward` <div class="label android">Android</div>

指定當用戶向前導航時下一個獲得焦點的視圖。參閱 [Android 官方文檔](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusForward)。

| Type   |
| ------ |
| number |

---

### `nextFocusLeft` <div class="label android">Android</div>

指定當用戶向左導航時下一個獲得焦點的視圖。參閱 [Android 官方文檔](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusLeft)。

| Type   |
| ------ |
| number |

---

### `nextFocusRight` <div class="label android">Android</div>

指定當用戶向右導航時下一個獲得焦點的視圖。參閱 [Android 官方文檔](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusRight)。

| Type   |
| ------ |
| number |

---

### `nextFocusUp` <div class="label android">Android</div>

指定當用戶向上導航時下一個獲得焦點的視圖。參閱 [Android 官方文檔](https://developer.android.com/reference/android/view/View.html#attr_android:nextFocusUp)。

| Type   |
| ------ |
| number |

---

### `onAccessibilityAction`

當用戶執行無障礙操作時觸發。此函數的唯一參數是一個包含要執行操作名稱的事件對象。

詳見 [無障礙功能指南](accessibility.md#accessibility-actions)。

| Type     |
| -------- |
| function |

---

### `onAccessibilityEscape` <div class="label ios">iOS</div>

當 `accessible` 為 `true` 時，系統會在用戶執行 escape 手勢時調用此函數。

| Type     |
| -------- |
| function |

---

### `onAccessibilityTap` <div class="label ios">iOS</div>

當 `accessible` 為 true 時，系統會在用戶執行無障礙點擊手勢時嘗試調用此函數。

| Type     |
| -------- |
| function |

---

### `onLayout`

在組件掛載時和佈局變化時觸發。

此事件會在佈局計算完成後立即觸發，但新佈局可能尚未反映到螢幕上（尤其在佈局動畫進行期間）。

| Type                                                       |
| ---------------------------------------------------------- |
| `md ({ nativeEvent: [LayoutEvent](layoutevent) }) => void` |

---

### `onMagicTap` <div class="label ios">iOS</div>

當 `accessible` 設為 `true` 時，系統會在用戶執行魔法點擊手勢時調用此函數。

| Type     |
| -------- |
| function |

---

### `onMoveShouldSetResponder`

Does this view want to "claim" touch responsiveness? This is called for every touch move on the `View` when it is not the responder.

| Type                                                        |
| ----------------------------------------------------------- |
| `md ({ nativeEvent: [PressEvent](pressevent) }) => boolean` |

---

### `onMoveShouldSetResponderCapture`

If a parent `View` wants to prevent a child `View` from becoming responder on a move, it should have this handler which returns `true`.

| Type                                                        |
| ----------------------------------------------------------- |
| `md ({ nativeEvent: [PressEvent](pressevent) }) => boolean` |

---

### `onResponderGrant`

視圖開始響應觸控事件。此時應高亮顯示以向用戶反饋交互狀態。

在 Android 上，從此回調返回 true 可阻止其他原生組件在此響應者終止前獲取響應權。

| Type                                                              |
| ----------------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void ｜ boolean` |

---

### `onResponderMove`

用戶正在移動手指。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({ nativeEvent: [PressEvent](pressevent) }) => void` |

---

### `onResponderReject`

Another responder is already active and will not release it to that `View` asking to be the responder.

| Type                                                     |
| -------------------------------------------------------- |
| `md ({ nativeEvent: [PressEvent](pressevent) }) => void` |

---

### `onResponderRelease`

觸控結束時觸發。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({ nativeEvent: [PressEvent](pressevent) }) => void` |

---

### `onResponderTerminate`

The responder has been taken from the `View`. Might be taken by other views after a call to `onResponderTerminationRequest`, or might be taken by the OS without asking (e.g., happens with control center/ notification center on iOS)

| Type                                                     |
| -------------------------------------------------------- |
| `md ({ nativeEvent: [PressEvent](pressevent) }) => void` |

---

### `onResponderTerminationRequest`

Some other `View` wants to become responder and is asking this `View` to release its responder. Returning `true` allows its release.

| Type                                                     |
| -------------------------------------------------------- |
| `md ({ nativeEvent: [PressEvent](pressevent) }) => void` |

---

### `onStartShouldSetResponder`

視圖是否希望在觸控開始時成為響應者？

| Type                                                        |
| ----------------------------------------------------------- |
| `md ({ nativeEvent: [PressEvent](pressevent) }) => boolean` |

---

### `onStartShouldSetResponderCapture`

If a parent `View` wants to prevent a child `View` from becoming responder on a touch start, it should have this handler which returns `true`.

| Type                                                        |
| ----------------------------------------------------------- |
| `md ({ nativeEvent: [PressEvent](pressevent) }) => boolean` |

---

### `pointerEvents`

Controls whether the `View` can be the target of touch events.

- `'auto'`: 視圖可成為觸控事件目標
- `'none'`: 視圖絕不成為觸控事件目標
- `'box-none'`: 視圖本身不響應觸控，但其子視圖可以。行為等效於 CSS 類：

```css
.box-none {
  pointer-events: none;
}
.box-none * {
  pointer-events: auto;
}
```

- `'box-only'`: 視圖本身可響應觸控，但其子視圖不能。行為等效於 CSS 類：

```css
.box-only {
  pointer-events: auto;
}
.box-only * {
  pointer-events: none;
}
```

> 由於 `pointerEvents` 不影響佈局/外觀，且我們已通過添加額外模式偏離規範，因此選擇不將 `pointerEvents` 包含在 `style` 中。在某些平台上，我們無論如何都需要將其實現為 `className`。使用 `style` 與否是平台的實現細節。

| Type                                         |
| -------------------------------------------- |
| enum('box-none', 'none', 'box-only', 'auto') |

---

### `removeClippedSubviews`

這是由 `RCTView` 公開的保留性能屬性，對於包含許多子視圖（大多數位於屏幕外）的滾動內容非常有用。要使此屬性生效，必須將其應用於包含許多超出其邊界的子視圖的視圖。子視圖還必須具有 `overflow: hidden`，包含視圖（或其父視圖之一）也應如此。

| Type |
| ---- |
| bool |

---

### `renderToHardwareTextureAndroid` <div class="label android">Android</div>

此 `View` 是否應將自身（及其所有子視圖）渲染為 GPU 上的單個硬件紋理。

On Android, this is useful for animations and interactions that only modify opacity, rotation, translation, and/or scale: in those cases, the view doesn't have to be redrawn and display lists don't need to be re-executed. The texture can be re-used and re-composited with different parameters. The downside is that this can use up limited video memory, so this prop should be set back to false at the end of the interaction/animation.

| Type |
| ---- |
| bool |

---

### `shouldRasterizeIOS` <div class="label ios">iOS</div>

此 `View` 是否應在合成前渲染為位圖。

在 iOS 上，這對於不修改此組件或其子組件尺寸的動畫和交互非常有用；例如，當平移靜態視圖的位置時，柵格化允許渲染器重用靜態視圖的緩存位圖，並在每幀快速合成。

柵格化會導致離屏繪製過程，且位圖會消耗內存。使用此屬性時需進行測試和測量。

| Type |
| ---- |
| bool |

---

### `style`

| Type                           |
| ------------------------------ |
| [View Style](view-style-props) |

---

### `testID`

用於在端到端測試中定位此視圖。

> 這會禁用此視圖的「僅佈局視圖移除」優化！

| Type   |
| ------ |
| string |