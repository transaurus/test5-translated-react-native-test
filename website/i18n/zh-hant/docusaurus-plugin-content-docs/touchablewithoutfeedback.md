---
id: touchablewithoutfeedback
title: TouchableWithoutFeedback
---

> 如果您正在尋找更全面且面向未來的觸控輸入處理方式，請參閱 [Pressable](pressable.md) API。

除非有充分理由，否則請勿使用。所有響應按壓的元素在被觸碰時都應具有視覺反饋。

`TouchableWithoutFeedback` 僅支援單個子元素。若需包含多個子組件，請將其包裹在 View 中。需特別注意，`TouchableWithoutFeedback` 的工作原理是克隆其子元素並應用響應屬性，因此要求任何中介組件都必須將這些屬性傳遞給底層的 React Native 組件。

## 使用模式

```tsx
function MyComponent(props: MyComponentProps) {
  return (
    <View {...props} style={{flex: 1, backgroundColor: '#fff'}}>
      <Text>My Component</Text>
    </View>
  );
}

<TouchableWithoutFeedback onPress={() => alert('Pressed!')}>
  <MyComponent />
</TouchableWithoutFeedback>;
```

## 範例

```SnackPlayer name=TouchableWithoutFeedback
import React, {useState} from 'react';
import {StyleSheet, TouchableWithoutFeedback, Text, View} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const TouchableWithoutFeedbackExample = () => {
  const [count, setCount] = useState(0);

  const onPress = () => {
    setCount(count + 1);
  };

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <View style={styles.countContainer}>
          <Text style={styles.countText}>Count: {count}</Text>
        </View>
        <TouchableWithoutFeedback onPress={onPress}>
          <View style={styles.button}>
            <Text>Touch Here</Text>
          </View>
        </TouchableWithoutFeedback>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    paddingHorizontal: 10,
  },
  button: {
    alignItems: 'center',
    backgroundColor: '#DDDDDD',
    padding: 10,
  },
  countContainer: {
    alignItems: 'center',
    padding: 10,
  },
  countText: {
    color: '#FF00FF',
  },
});

export default TouchableWithoutFeedbackExample;
```

---

# 參考文獻

## 屬性

### `accessibilityIgnoresInvertColors` <div class="label ios">iOS</div>

指示當顏色反轉開啟時，此視圖是否應被反轉。設為 `true` 時，即使開啟顏色反轉，視圖也不會被反轉。

詳見 [無障礙指南](accessibility.md#accessibilityignoresinvertcolors) 以獲取更多資訊。

| Type    |
| ------- |
| Boolean |

---

### `accessible`

當設為 `true` 時，表示該視圖是一個無障礙元素。預設情況下，所有可觸碰元素均為無障礙元素。

| Type |
| ---- |
| bool |

---

### `accessibilityLabel`

覆寫螢幕閱讀器在使用者與元素互動時朗讀的文字。預設情況下，標籤由遍歷所有子元素並累積以空格分隔的 `Text` 節點構成。

| Type   |
| ------ |
| string |

---

### `accessibilityLanguage` <div class="label ios">iOS</div>

指定螢幕閱讀器在使用者與元素互動時應使用的語言，需遵循 [BCP 47 規範](https://www.rfc-editor.org/info/bcp47)。

詳見 [iOS `accessibilityLanguage` 文件](https://developer.apple.com/documentation/objectivec/nsobject/1615192-accessibilitylanguage) 以獲取更多資訊。

| Type   |
| ------ |
| string |

---

### `accessibilityHint`

當無障礙標籤無法明確表達操作結果時，無障礙提示可幫助使用者理解在無障礙元素上執行操作後會發生的結果。

| Type   |
| ------ |
| string |

---

### `accessibilityRole`

`accessibilityRole` 向輔助技術使用者傳達組件的用途。

`accessibilityRole` 可為下列值之一：

- `'none'` - 當元素沒有角色時使用。
- `'button'` - 當元素應被視為按鈕時使用。
- `'link'` - 當元素應被視為連結時使用。
- `'search'` - 當文字輸入框元素也應被視為搜尋欄位時使用。
- `'image'` - 當元素應被視為圖片時使用。可與按鈕或連結等組合使用。
- `'keyboardkey'` - 當元素作為鍵盤按鍵時使用。
- `'text'` - 當元素應被視為靜態不可變文字時使用。
- `'adjustable'` - 當元素可被「調整」時使用（例如滑桿）。
- `'imagebutton'` - 當元素應被視為按鈕且同時是圖片時使用。
- `'header'` - 當元素作為內容區段的標題時使用（例如導覽列標題）。
- `'summary'` - 當元素用於在應用程式首次啟動時提供當前條件的快速摘要時使用。
- `'alert'` - 當元素包含需向用戶展示的重要文字時使用。
- `'checkbox'` - 當元素代表可勾選、取消勾選或混合狀態的核取方塊時使用。
- `'combobox'` - 當元素代表可讓用戶從多個選項中選擇的下拉式方塊時使用。
- `'menu'` - 當元件是選單時使用。
- `'menubar'` - 當元件是多個選單的容器時使用。
- `'menuitem'` - 用於表示選單內的項目。
- `'progressbar'` - 用於表示顯示任務進度的元件。
- `'radio'` - 用於表示單選按鈕。
- `'radiogroup'` - 用於表示一組單選按鈕。
- `'scrollbar'` - 用於表示捲軸。
- `'spinbutton'` - 用於表示開啟選項清單的按鈕。
- `'switch'` - 用於表示可開關的切換開關。
- `'tab'` - 用於表示分頁標籤。
- `'tablist'` - 用於表示分頁標籤清單。
- `'timer'` - 用於表示計時器。
- `'toolbar'` - 用於表示工具列（動作按鈕或元件的容器）。

| Type   |
| ------ |
| string |

---

### `accessibilityState`

向輔助技術用戶描述元件的當前狀態。

詳見[無障礙指南](accessibility.md#accessibilitystate-ios-android)以獲取更多資訊。

| Type                                                                                             |
| ------------------------------------------------------------------------------------------------ |
| object: `{disabled: bool, selected: bool, checked: bool or 'mixed', busy: bool, expanded: bool}` |

---

### `accessibilityActions`

無障礙操作允許輔助技術以程式化方式觸發元件的操作。`accessibilityActions`屬性應包含操作物件清單，每個操作物件需包含名稱和標籤欄位。

詳見[無障礙指南](accessibility.md#accessibility-actions)以獲取更多資訊。

| Type  |
| ----- |
| array |

---

### `aria-busy`

表示元素正在修改中，輔助技術可能需要等待變更完成後再通知用戶。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-checked`

表示可勾選元素的狀態。此欄位可接受布林值或「mixed」字串來表示混合狀態的核取方塊。

| Type             | Default |
| ---------------- | ------- |
| boolean, 'mixed' | false   |

---

### `aria-disabled`

表示元素可感知但被禁用，因此不可編輯或操作。

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

表示此無障礙元素內包含的子元素是否隱藏。

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

### `aria-live` <div class="label android">Android</div>

表示元素將會更新，並描述用戶代理、輔助技術和用戶可以預期的即時區域更新類型。

- **off** 輔助服務不應宣佈此視圖的變更。
- **polite** 輔助服務應宣佈此視圖的變更。
- **assertive** 輔助服務應中斷正在進行的語音，立即宣佈此視圖的變更。

| Type                                     | Default |
| ---------------------------------------- | ------- |
| enum(`'assertive'`, `'off'`, `'polite'`) | `'off'` |

---

### `aria-modal` <div class="label ios">iOS</div>

布林值，表示 VoiceOver 是否應忽略接收器兄弟視圖中的元素。優先於 [`accessibilityViewIsModal`](#accessibilityviewismodal-ios) 屬性。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-selected`

表示可選元素當前是否被選中。

| Type    |
| ------- |
| boolean |

### `onAccessibilityAction`

當用戶執行無障礙操作時調用。此函數的唯一參數是一個包含要執行操作名稱的事件。

詳見[無障礙指南](accessibility.md#accessibility-actions)以獲取更多資訊。

| Type     |
| -------- |
| function |

---

### `accessibilityValue`

表示組件的當前值。可以是組件值的文字描述，或是基於範圍的組件（如滑塊和進度條）的範圍資訊（最小值、當前值和最大值）。

詳見[無障礙指南](accessibility.md#accessibilityvalue-ios-android)以獲取更多資訊。

| Type                                                            |
| --------------------------------------------------------------- |
| object: `{min: number, max: number, now: number, text: string}` |

---

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

### `delayLongPress`

從 `onPressIn` 開始到呼叫 `onLongPress` 的持續時間（以毫秒為單位）。

| Type   |
| ------ |
| number |

---

### `delayPressIn`

從觸碰開始到呼叫 `onPressIn` 的持續時間（以毫秒為單位）。

| Type   |
| ------ |
| number |

---

### `delayPressOut`

從觸碰釋放到呼叫 `onPressOut` 的持續時間（以毫秒為單位）。

| Type   |
| ------ |
| number |

---

### `disabled`

若為 true，則禁用此元件的所有互動。

| Type |
| ---- |
| bool |

---

### `hitSlop`

定義觸碰可以從按鈕多遠的距離開始。當移出按鈕時，此值會加到 `pressRetentionOffset` 上。

> 觸碰區域永遠不會超出父視圖的邊界，且如果觸碰同時命中兩個重疊的視圖，兄弟視圖的 Z-index 永遠具有優先權。

| Type                   |
| ---------------------- |
| [Rect](rect) or number |

### `id`

用於從原生程式碼定位此視圖。優先於 `nativeID` 屬性。

| Type   |
| ------ |
| string |

---

### `onBlur`

當項目失去焦點時呼叫。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({nativeEvent: [TargetEvent](targetevent)}) => void` |

---

### `onFocus`

當項目獲得焦點時呼叫。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({nativeEvent: [TargetEvent](targetevent)}) => void` |

---

### `onLayout`

在掛載時和佈局變更時呼叫。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)}) => void` |

---

### `onLongPress`

若 `onPressIn` 後的持續時間超過 370 毫秒則呼叫。此時間段可透過 [`delayLongPress`](#delaylongpress) 自訂。

| Type     |
| -------- |
| function |

---

### `onPress`

當觸碰釋放時呼叫，但若被取消（例如因滾動搶佔了回應鎖）則不呼叫。第一個函數參數是 [PressEvent](pressevent) 形式的事件。

| Type     |
| -------- |
| function |

---

### `onPressIn`

當可觸碰元件被按下時立即呼叫，甚至在 onPress 之前呼叫。這在發起網路請求時很有用。第一個函數參數是 [PressEvent](pressevent) 形式的事件。

| Type     |
| -------- |
| function |

---

### `onPressOut`

當觸碰釋放時立即呼叫，甚至在 onPress 之前呼叫。第一個函數參數是 [PressEvent](pressevent) 形式的事件。

| Type     |
| -------- |
| function |

---

### `pressRetentionOffset`

當滾動視圖被禁用時，此值定義觸碰可以從按鈕移出多遠的距離才會停用按鈕。一旦停用，嘗試移回按鈕會發現按鈕再次被啟用！在滾動視圖禁用時來回移動多次。請確保傳入常數以減少記憶體分配。

| Type                   |
| ---------------------- |
| [Rect](rect) or number |

---

### `nativeID`

| Type   |
| ------ |
| string |

---

### `testID`

用於在端對端測試中定位此視圖。

| Type   |
| ------ |
| string |

---

### `touchSoundDisabled` <div class="label android">Android</div>

若為 true，觸碰時不會播放系統音效。

| Type    |
| ------- |
| Boolean |