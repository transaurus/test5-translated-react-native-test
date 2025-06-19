---
id: layout-props
title: Layout Props
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

> 關於這些屬性的更詳細範例，請參閱 [使用 Flexbox 佈局](flexbox) 頁面。

### 範例

以下範例展示不同屬性如何影響或塑造 React Native 的佈局。您可以嘗試在變更 `flexWrap` 屬性值的同時，從 UI 中新增或移除方塊。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=LayoutProps%20Example&ext=js
import React, {useState} from 'react';
import {Button, ScrollView, StyleSheet, Text, View} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  const [flexDirection, setFlexDirection] = useState(0);
  const [justifyContent, setJustifyContent] = useState(0);
  const [alignItems, setAlignItems] = useState(0);
  const [direction, setDirection] = useState(0);
  const [wrap, setWrap] = useState(0);

  const [squares, setSquares] = useState([<Square />, <Square />, <Square />]);

  const hookedStyles = {
    flexDirection: flexDirections[flexDirection],
    justifyContent: justifyContents[justifyContent],
    alignItems: alignItemsArr[alignItems],
    direction: directions[direction],
    flexWrap: wraps[wrap],
  };

  const changeSetting = (value, options, setterFunction) => {
    if (value === options.length - 1) {
      setterFunction(0);
      return;
    }
    setterFunction(value + 1);
  };

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <View style={[styles.container, styles.playingSpace, hookedStyles]}>
          {squares.map(elem => elem)}
        </View>
        <ScrollView style={styles.layoutContainer}>
          <View style={styles.controlSpace}>
            <View style={styles.buttonView}>
              <Button
                title="Change Flex Direction"
                onPress={() =>
                  changeSetting(flexDirection, flexDirections, setFlexDirection)
                }
              />
              <Text style={styles.text}>{flexDirections[flexDirection]}</Text>
            </View>
            <View style={styles.buttonView}>
              <Button
                title="Change Justify Content"
                onPress={() =>
                  changeSetting(
                    justifyContent,
                    justifyContents,
                    setJustifyContent,
                  )
                }
              />
              <Text style={styles.text}>{justifyContents[justifyContent]}</Text>
            </View>
            <View style={styles.buttonView}>
              <Button
                title="Change Align Items"
                onPress={() =>
                  changeSetting(alignItems, alignItemsArr, setAlignItems)
                }
              />
              <Text style={styles.text}>{alignItemsArr[alignItems]}</Text>
            </View>
            <View style={styles.buttonView}>
              <Button
                title="Change Direction"
                onPress={() =>
                  changeSetting(direction, directions, setDirection)
                }
              />
              <Text style={styles.text}>{directions[direction]}</Text>
            </View>
            <View style={styles.buttonView}>
              <Button
                title="Change Flex Wrap"
                onPress={() => changeSetting(wrap, wraps, setWrap)}
              />
              <Text style={styles.text}>{wraps[wrap]}</Text>
            </View>
            <View style={styles.buttonView}>
              <Button
                title="Add Square"
                onPress={() => setSquares([...squares, <Square />])}
              />
            </View>
            <View style={styles.buttonView}>
              <Button
                title="Delete Square"
                onPress={() =>
                  setSquares(squares.filter((v, i) => i !== squares.length - 1))
                }
              />
            </View>
          </View>
        </ScrollView>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const flexDirections = ['row', 'row-reverse', 'column', 'column-reverse'];
const justifyContents = [
  'flex-start',
  'flex-end',
  'center',
  'space-between',
  'space-around',
  'space-evenly',
];
const alignItemsArr = [
  'flex-start',
  'flex-end',
  'center',
  'stretch',
  'baseline',
];
const wraps = ['nowrap', 'wrap', 'wrap-reverse'];
const directions = ['inherit', 'ltr', 'rtl'];

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  layoutContainer: {
    flex: 0.5,
  },
  playingSpace: {
    backgroundColor: 'white',
    borderColor: 'blue',
    borderWidth: 3,
    overflow: 'hidden',
  },
  controlSpace: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  buttonView: {
    width: '50%',
    padding: 10,
  },
  text: {
    textAlign: 'center',
  },
});

const Square = () => (
  <View
    style={{
      width: 50,
      height: 50,
      backgroundColor: randomHexColor(),
    }}
  />
);

const randomHexColor = () => {
  return '#000000'.replace(/0/g, () => {
    return Math.round(Math.random() * 14).toString(16);
  });
};

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=LayoutProps%20Example&ext=tsx
import React, {useState} from 'react';
import {
  Button,
  ScrollView,
  StyleSheet,
  Text,
  View,
  FlexAlignType,
  FlexStyle,
} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  const [flexDirection, setFlexDirection] = useState(0);
  const [justifyContent, setJustifyContent] = useState(0);
  const [alignItems, setAlignItems] = useState(0);
  const [direction, setDirection] = useState(0);
  const [wrap, setWrap] = useState(0);

  const [squares, setSquares] = useState([<Square />, <Square />, <Square />]);

  const hookedStyles = {
    flexDirection: flexDirections[flexDirection],
    justifyContent: justifyContents[justifyContent],
    alignItems: alignItemsArr[alignItems],
    direction: directions[direction],
    flexWrap: wraps[wrap],
  } as FlexStyle;

  const changeSetting = (
    value: number,
    options: any[],
    setterFunction: (index: number) => void,
  ) => {
    if (value === options.length - 1) {
      setterFunction(0);
      return;
    }
    setterFunction(value + 1);
  };

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <View style={[styles.container, styles.playingSpace, hookedStyles]}>
          {squares.map(elem => elem)}
        </View>
        <ScrollView style={styles.layoutContainer}>
          <View style={styles.controlSpace}>
            <View style={styles.buttonView}>
              <Button
                title="Change Flex Direction"
                onPress={() =>
                  changeSetting(flexDirection, flexDirections, setFlexDirection)
                }
              />
              <Text style={styles.text}>{flexDirections[flexDirection]}</Text>
            </View>
            <View style={styles.buttonView}>
              <Button
                title="Change Justify Content"
                onPress={() =>
                  changeSetting(
                    justifyContent,
                    justifyContents,
                    setJustifyContent,
                  )
                }
              />
              <Text style={styles.text}>{justifyContents[justifyContent]}</Text>
            </View>
            <View style={styles.buttonView}>
              <Button
                title="Change Align Items"
                onPress={() =>
                  changeSetting(alignItems, alignItemsArr, setAlignItems)
                }
              />
              <Text style={styles.text}>{alignItemsArr[alignItems]}</Text>
            </View>
            <View style={styles.buttonView}>
              <Button
                title="Change Direction"
                onPress={() =>
                  changeSetting(direction, directions, setDirection)
                }
              />
              <Text style={styles.text}>{directions[direction]}</Text>
            </View>
            <View style={styles.buttonView}>
              <Button
                title="Change Flex Wrap"
                onPress={() => changeSetting(wrap, wraps, setWrap)}
              />
              <Text style={styles.text}>{wraps[wrap]}</Text>
            </View>
            <View style={styles.buttonView}>
              <Button
                title="Add Square"
                onPress={() => setSquares([...squares, <Square />])}
              />
            </View>
            <View style={styles.buttonView}>
              <Button
                title="Delete Square"
                onPress={() =>
                  setSquares(squares.filter((v, i) => i !== squares.length - 1))
                }
              />
            </View>
          </View>
        </ScrollView>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const flexDirections = [
  'row',
  'row-reverse',
  'column',
  'column-reverse',
] as FlexStyle['flexDirection'][];
const justifyContents = [
  'flex-start',
  'flex-end',
  'center',
  'space-between',
  'space-around',
  'space-evenly',
] as FlexStyle['justifyContent'][];
const alignItemsArr = [
  'flex-start',
  'flex-end',
  'center',
  'stretch',
  'baseline',
] as FlexAlignType[];
const wraps = ['nowrap', 'wrap', 'wrap-reverse'];
const directions = ['inherit', 'ltr', 'rtl'];

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  layoutContainer: {
    flex: 0.5,
  },
  playingSpace: {
    backgroundColor: 'white',
    borderColor: 'blue',
    borderWidth: 3,
    overflow: 'hidden',
  },
  controlSpace: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  buttonView: {
    width: '50%',
    padding: 10,
  },
  text: {
    textAlign: 'center',
  },
});

const Square = () => (
  <View
    style={{
      width: 50,
      height: 50,
      backgroundColor: randomHexColor(),
    }}
  />
);

const randomHexColor = () => {
  return '#000000'.replace(/0/g, () => {
    return Math.round(Math.random() * 14).toString(16);
  });
};

export default App;
```

</TabItem>
</Tabs>

---

# 參考

## 屬性

### `alignContent`

`alignContent` controls how rows align in the cross direction, overriding the `alignContent` of the parent. See https://developer.mozilla.org/en-US/docs/Web/CSS/align-content for more details.

| Type                                                                                                 | Required |
| ---------------------------------------------------------------------------------------------------- | -------- |
| enum('flex-start', 'flex-end', 'center', 'stretch', 'space-between', 'space-around', 'space-evenly') | No       |

---

### `alignItems`

`alignItems` aligns children in the cross direction. For example, if children are flowing vertically, `alignItems` controls how they align horizontally. It works like `align-items` in CSS (default: stretch). See https://developer.mozilla.org/en-US/docs/Web/CSS/align-items for more details.

| Type                                                            | Required |
| --------------------------------------------------------------- | -------- |
| enum('flex-start', 'flex-end', 'center', 'stretch', 'baseline') | No       |

---

### `alignSelf`

`alignSelf` controls how a child aligns in the cross direction, overriding the `alignItems` of the parent. It works like `align-self` in CSS (default: auto). See https://developer.mozilla.org/en-US/docs/Web/CSS/align-self for more details.

| Type                                                                    | Required |
| ----------------------------------------------------------------------- | -------- |
| enum('auto', 'flex-start', 'flex-end', 'center', 'stretch', 'baseline') | No       |

---

### `aspectRatio`

Aspect ratio controls the size of the undefined dimension of a node. See https://developer.mozilla.org/en-US/docs/Web/CSS/aspect-ratio for more details.

- 在已設定寬度/高度的節點上，長寬比控制未設定維度的尺寸
- 在已設定 flex basis 的節點上，長寬比控制未設定時交叉軸上的節點尺寸
- 在具有測量函式的節點上，長寬比的作用如同測量函式測量 flex basis
- 在具有 flex grow/shrink 的節點上，長寬比控制未設定時交叉軸上的節點尺寸
- 長寬比會考慮最小/最大尺寸限制

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `borderBottomWidth`

`borderBottomWidth` works like `border-bottom-width` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/border-bottom-width for more details.

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `borderEndWidth`

當方向為 `ltr` 時，`borderEndWidth` 等同於 `borderRightWidth`。當方向為 `rtl` 時，`borderEndWidth` 等同於 `borderLeftWidth`。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `borderLeftWidth`

`borderLeftWidth` works like `border-left-width` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/border-left-width for more details.

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `borderRightWidth`

`borderRightWidth` works like `border-right-width` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/border-right-width for more details.

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `borderStartWidth`

當方向為 `ltr` 時，`borderStartWidth` 等同於 `borderLeftWidth`。當方向為 `rtl` 時，`borderStartWidth` 等同於 `borderRightWidth`。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `borderTopWidth`

`borderTopWidth` works like `border-top-width` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/border-top-width for more details.

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `borderWidth`

`borderWidth` works like `border-width` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/border-width for more details.

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `bottom`

`bottom` 表示此元件底部邊緣偏移的邏輯像素數。

其功能類似 CSS 中的 `bottom`，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/bottom 了解 `bottom` 如何影響佈局。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `boxSizing`

`boxSizing` defines how the element's various sizing props (`width`, `height`, `minWidth`, `minHeight`, etc.) are computed. If `boxSizing` is `border-box`, these sizes apply to the border box of the element. If it is `content-box`, they apply to the content box of the element. The default value is `border-box`. The [web documentation](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing) is a good source of information if you wish to learn more about how this prop works.

| Type                              | Required |
| --------------------------------- | -------- |
| enum('border-box', 'content-box') | No       |

---

### `columnGap`

`columnGap` works like `column-gap` in CSS. Only pixel units are supported in React Native. See https://developer.mozilla.org/en-US/docs/Web/CSS/column-gap for more details.

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `direction`

`direction` specifies the directional flow of the user interface. The default is `inherit`, except for root node which will have value based on the current locale. See https://www.yogalayout.dev/docs/styling/layout-direction for more details.

| Type                          | Required | Platform |
| ----------------------------- | -------- | -------- |
| enum('inherit', 'ltr', 'rtl') | No       | iOS      |

---

### `display`

`display` 設定此元件的顯示類型。

其功能類似 CSS 中的 `display`，但僅支援 'flex'、'none' 和 'contents' 值。預設值為 `flex`。

| Type                             | Required |
| -------------------------------- | -------- |
| enum('none', 'flex', 'contents') | No       |

---

### `end`

當方向為 `ltr` 時，`end` 等同於 `right`；當方向為 `rtl` 時，`end` 等同於 `left`。

此樣式優先於 `left` 和 `right` 樣式。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `flex`

在 React Native 中，`flex` 的運作方式與 CSS 不同。`flex` 是一個數字而非字串，其行為遵循 [Yoga](https://github.com/facebook/yoga) 佈局引擎。

When `flex` is a positive number, it makes the component flexible, and it will be sized proportional to its flex value. So a component with `flex` set to 2 will take twice the space as a component with `flex` set to 1. `flex: <positive number>` equates to `flexGrow: <positive number>, flexShrink: 1, flexBasis: 0`.

當 `flex` 為 0 時，組件會根據 `width` 和 `height` 調整大小，且不具彈性。

當 `flex` 為 -1 時，組件通常會根據 `width` 和 `height` 調整大小。然而，如果空間不足，組件會縮小至其 `minWidth` 和 `minHeight`。

`flexGrow`、`flexShrink` 和 `flexBasis` 的功能與 CSS 中相同。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `flexBasis`

`flexBasis` is an axis-independent way of providing the default size of an item along the main axis. Setting the `flexBasis` of a child is similar to setting the `width` of that child if its parent is a container with `flexDirection: row` or setting the `height` of a child if its parent is a container with `flexDirection: column`. The `flexBasis` of an item is the default size of that item, the size of the item before any `flexGrow` and `flexShrink` calculations are performed.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `flexDirection`

`flexDirection` controls which directions children of a container go. `row` goes left to right, `column` goes top to bottom, and you may be able to guess what the other two do. It works like `flex-direction` in CSS, except the default is `column`. See https://developer.mozilla.org/en-US/docs/Web/CSS/flex-direction for more details.

| Type                                                   | Required |
| ------------------------------------------------------ | -------- |
| enum('row', 'row-reverse', 'column', 'column-reverse') | No       |

---

### `flexGrow`

`flexGrow` 描述容器內的主軸空間應如何在其子組件之間分配。在佈局其子組件後，容器會根據子組件指定的 flex grow 值分配任何剩餘空間。

`flexGrow` 接受任何大於或等於 0 的浮點數，默認值為 0。容器會根據子組件的 `flexGrow` 值加權分配任何剩餘空間。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `flexShrink`

[`flexShrink`](layout-props#flexshrink) 描述當子組件的總大小在主軸上超出容器大小時，如何沿主軸縮小子組件。`flexShrink` 與 `flexGrow` 非常相似，可以將其視為將任何溢出的空間視為負的剩餘空間。這兩個屬性也能很好地協同工作，允許子組件根據需要增長或縮小。

`flexShrink` 接受任何大於或等於 0 的浮點數，默認值為 0。容器會根據子組件的 `flexShrink` 值加權縮小其子組件。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `flexWrap`

`flexWrap` controls whether children can wrap around after they hit the end of a flex container. It works like `flex-wrap` in CSS (default: nowrap). See https://developer.mozilla.org/en-US/docs/Web/CSS/flex-wrap for more details. Note it does not work anymore with `alignItems: stretch` (the default), so you may want to use `alignItems: flex-start` for example (breaking change details: https://github.com/facebook/react-native/releases/tag/v0.28.0).

| Type                                   | Required |
| -------------------------------------- | -------- |
| enum('wrap', 'nowrap', 'wrap-reverse') | No       |

---

### `gap`

`gap` works like `gap` in CSS. Only pixel units are supported in React Native. See https://developer.mozilla.org/en-US/docs/Web/CSS/gap for more details.

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `height`

`height` 設定此元件的高度。

It works similarly to `height` in CSS, but in React Native you must use points or percentages. Ems and other units are not supported. See https://developer.mozilla.org/en-US/docs/Web/CSS/height for more details.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `justifyContent`

`justifyContent` aligns children in the main direction. For example, if children are flowing vertically, `justifyContent` controls how they align vertically. It works like `justify-content` in CSS (default: flex-start). See https://developer.mozilla.org/en-US/docs/Web/CSS/justify-content for more details.

| Type                                                                                      | Required |
| ----------------------------------------------------------------------------------------- | -------- |
| enum('flex-start', 'flex-end', 'center', 'space-between', 'space-around', 'space-evenly') | No       |

---

### `left`

`left` 是以邏輯像素為單位，偏移此元件左邊緣的數值。

其功能類似 CSS 中的 `left`，但在 React Native 中必須使用點數或百分比。不支援 em 等其他單位。

See https://developer.mozilla.org/en-US/docs/Web/CSS/left for more details of how `left` affects layout.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `margin`

Setting `margin` has the same effect as setting each of `marginTop`, `marginLeft`, `marginBottom`, and `marginRight`. See https://developer.mozilla.org/en-US/docs/Web/CSS/margin for more details.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `marginBottom`

`marginBottom` works like `margin-bottom` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/margin-bottom for more details.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `marginEnd`

當方向為 `ltr` 時，`marginEnd` 等同於 `marginRight`；當方向為 `rtl` 時，`marginEnd` 等同於 `marginLeft`。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `marginHorizontal`

設定 `marginHorizontal` 的效果等同於同時設定 `marginLeft` 和 `marginRight`。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `marginLeft`

`marginLeft` works like `margin-left` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/margin-left for more details.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `marginRight`

`marginRight` works like `margin-right` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/margin-right for more details.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `marginStart`

當方向為 `ltr` 時，`marginStart` 等同於 `marginLeft`；當方向為 `rtl` 時，`marginStart` 等同於 `marginRight`。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `marginTop`

`marginTop` works like `margin-top` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/margin-top for more details.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `marginVertical`

設定 `marginVertical` 的效果等同於同時設定 `marginTop` 和 `marginBottom`。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `maxHeight`

`maxHeight` 是此元件的最大高度，以邏輯像素為單位。

其功能類似 CSS 中的 `max-height`，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/max-height 以獲取更多細節。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `maxWidth`

`maxWidth` 是此元件的最大寬度，以邏輯像素為單位。

其功能類似 CSS 中的 `max-width`，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/max-width 以獲取更多細節。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `minHeight`

`minHeight` 是此元件的最小高度，以邏輯像素為單位。

其功能類似 CSS 中的 `min-height`，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/min-height 以獲取更多細節。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `minWidth`

`minWidth` 是此元件的最小寬度，以邏輯像素為單位。

其功能類似 CSS 中的 `min-width`，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/min-width 以獲取更多細節。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `overflow`

`overflow` 控制子元件的測量與顯示方式。`overflow: hidden` 會裁切視圖，而 `overflow: scroll` 會讓視圖獨立於父元件主軸進行測量。其功能類似 CSS 中的 `overflow`（預設值為 visible）。詳見 https://developer.mozilla.org/en/docs/Web/CSS/overflow 以獲取更多細節。

| Type                                | Required |
| ----------------------------------- | -------- |
| enum('visible', 'hidden', 'scroll') | No       |

---

### `padding`

設定 `padding` 的效果等同於同時設定 `paddingTop`、`paddingBottom`、`paddingLeft` 和 `paddingRight`。詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/padding 以獲取更多細節。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `paddingBottom`

`paddingBottom` 的功能類似 CSS 中的 `padding-bottom`。詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/padding-bottom 以獲取更多細節。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `paddingEnd`

當方向為 `ltr` 時，`paddingEnd` 等同於 `paddingRight`；當方向為 `rtl` 時，`paddingEnd` 等同於 `paddingLeft`。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `paddingHorizontal`

設定 `paddingHorizontal` 的效果等同於同時設定 `paddingLeft` 和 `paddingRight`。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `paddingLeft`

`paddingLeft` works like `padding-left` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/padding-left for more details.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `paddingRight`

`paddingRight` works like `padding-right` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/padding-right for more details.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `paddingStart`

當方向為 `ltr` 時，`paddingStart` 等同於 `paddingLeft`；當方向為 `rtl` 時，`paddingStart` 等同於 `paddingRight`。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `paddingTop`

`paddingTop` works like `padding-top` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/padding-top for more details.

| Type            | Required |
| --------------- | -------- |
| number, ,string | No       |

---

### `paddingVertical`

設定 `paddingVertical` 的效果等同於同時設定 `paddingTop` 和 `paddingBottom`。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `position`

`position` in React Native is similar to [regular CSS](https://developer.mozilla.org/en-US/docs/Web/CSS/position), but everything is set to `relative` by default.

`relative` 會根據佈局的正常流來定位元素。偏移量（`top`、`bottom`、`left`、`right`）會相對於此佈局進行調整。

`absolute` 會將元素從佈局的正常流中移除。偏移量會相對於其 [包含區塊](./flexbox.md#the-containing-block) 進行調整。

`static` 會根據佈局的正常流來定位元素。偏移量將不會產生任何效果。
`static` 元素不會為絕對定位的後代元素形成包含區塊。

更多資訊請參閱 [Flexbox 佈局文件](./flexbox.md#position)。此外，[Yoga 文件](https://www.yogalayout.dev/docs/styling/position) 詳細說明了 `position` 在 React Native 與 CSS 之間的差異。

| Type                                   | Required |
| -------------------------------------- | -------- |
| enum('absolute', 'relative', 'static') | No       |

---

### `right`

`right` 表示以邏輯像素為單位偏移此元件右邊緣的數值。

其功能類似 CSS 中的 `right`，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

詳情請參閱 https://developer.mozilla.org/en-US/docs/Web/CSS/right 了解 `right` 如何影響佈局。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `rowGap`

`rowGap` works like `row-gap` in CSS. Only pixel units are supported in React Native. See https://developer.mozilla.org/en-US/docs/Web/CSS/row-gap for more details.

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `start`

當方向為 `ltr` 時，`start` 等同於 `left`；當方向為 `rtl` 時，`start` 等同於 `right`。

此樣式的優先級高於 `left`、`right` 和 `end` 樣式。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `top`

`top` 表示以邏輯像素為單位偏移此元件頂部邊緣的數值。

其運作方式類似於 CSS 中的 `top`，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

See https://developer.mozilla.org/en-US/docs/Web/CSS/top for more details of how `top` affects layout.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `width`

`width` 用於設定此元件的寬度。

It works similarly to `width` in CSS, but in React Native you must use points or percentages. Ems and other units are not supported. See https://developer.mozilla.org/en-US/docs/Web/CSS/width for more details.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `zIndex`

`zIndex` 控制哪些元件會顯示在其他元件之上。通常情況下不需要使用 `zIndex`，元件會依照文件樹中的順序渲染，因此後面的元件會覆蓋前面的元件。但在動畫或自訂模態介面等不希望遵循此行為的情境下，`zIndex` 會很有用。

It works like the CSS `z-index` property - components with a larger `zIndex` will render on top. Think of the z-direction like it's pointing from the phone into your eyeball. See https://developer.mozilla.org/en-US/docs/Web/CSS/z-index for more details.

在 iOS 上，要讓 `zIndex` 正常運作，可能需要讓 `View` 元件彼此是兄弟節點。

| Type   | Required |
| ------ | -------- |
| number | No       |

---