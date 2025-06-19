---
id: layout-props
title: Layout Props
---

> 關於這些屬性的更詳細範例，請參閱 [使用 Flexbox 佈局](flexbox) 頁面。

### 範例

以下範例展示不同屬性如何影響或塑造 React Native 的佈局。您可以嘗試在變更 `flexWrap` 屬性值的同時，從 UI 中新增或移除方塊。

```SnackPlayer name=LayoutProps%20Example
import React, { useState } from 'react';
import { Button, ScrollView, StatusBar, StyleSheet, Text, View } from 'react-native';

const App = () => {
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
  const [flexDirection, setFlexDirection] = useState(0);
  const [justifyContent, setJustifyContent] = useState(0);
  const [alignItems, setAlignItems] = useState(0);
  const [direction, setDirection] = useState(0);
  const [wrap, setWrap] = useState(0);

  const hookedStyles = {
    flexDirection: flexDirections[flexDirection],
    justifyContent: justifyContents[justifyContent],
    alignItems: alignItemsArr[alignItems],
    direction: directions[direction],
    flexWrap: wraps[wrap],
  };

  const changeSetting = (value, options, setterFunction) => {
    if (value == options.length - 1) {
      setterFunction(0);
      return;
    }
    setterFunction(value + 1);
  };
  const [squares, setSquares] = useState([<Square />, <Square />, <Square />]);
  return (
    <>
      <View style={{ paddingTop: StatusBar.currentHeight }} />
      <View style={[styles.container, styles.playingSpace, hookedStyles]}>
        {squares.map(elem => elem)}
      </View>
      <ScrollView style={[styles.container]}>
        <View style={[styles.controlSpace]}>
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
                  setJustifyContent
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
              onPress={() => changeSetting(direction, directions, setDirection)}
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
              onPress={() => setSquares([...squares, <Square/>])}
            />
          </View>
          <View style={styles.buttonView}>
            <Button
              title="Delete Square"
              onPress={() =>
                setSquares(squares.filter((v, i) => i != squares.length - 1))
              }
            />
          </View>
        </View>
      </ScrollView>
    </>
  );
};

const styles = StyleSheet.create({
  container: {
    height: '50%',
  },
  playingSpace: {
    backgroundColor: 'white',
    borderColor: 'blue',
    borderWidth: 3,
  },
  controlSpace: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    backgroundColor: '#F5F5F5',
  },
  buttonView: {
    width: '50%',
    padding: 10,
  },
  text: { textAlign: 'center' },
});

const Square = () => (
  <View style={{
    width: 50,
    height: 50,
    backgroundColor: randomHexColor(),
  }} />
);

const randomHexColor = () => {
  return '#000000'.replace(/0/g, () => {
    return (~~(Math.random() * 16)).toString(16);
  });
};

export default App;
```

---

# 參考

## 屬性

### `alignContent`

`alignContent` controls how rows align in the cross direction, overriding the `alignContent` of the parent. See https://developer.mozilla.org/en-US/docs/Web/CSS/align-content for more details.

| Type                                                                                 | Required |
| ------------------------------------------------------------------------------------ | -------- |
| enum('flex-start', 'flex-end', 'center', 'stretch', 'space-between', 'space-around') | No       |

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

- 在已設定寬度/高度的節點上，長寬比控制未設定維度的大小
- 在已設定 flex basis 的節點上，長寬比控制未設定交叉軸方向的大小
- 在具有測量函式的節點上，長寬比的作用如同測量函式測量 flex basis
- 在具有 flex grow/shrink 的節點上，長寬比控制未設定交叉軸方向的大小
- 長寬比會考慮最小/最大尺寸限制

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `borderBottomWidth`

`borderBottomWidth` works like `border-bottom-width` in CSS. See https://developer.mozilla.org/en-US/docs/Web/CSS/border-bottom-width for more details.

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `borderEndWidth`

When direction is `ltr`, `borderEndWidth` is equivalent to `borderRightWidth`. When direction is `rtl`, `borderEndWidth` is equivalent to `borderLeftWidth`.

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

When direction is `ltr`, `borderStartWidth` is equivalent to `borderLeftWidth`. When direction is `rtl`, `borderStartWidth` is equivalent to `borderRightWidth`.

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

`bottom` 表示此元件底部邊緣的邏輯像素偏移量。

其功能類似 CSS 中的 `bottom`，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/bottom 了解 `bottom` 如何影響佈局。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `direction`

`direction` specifies the directional flow of the user interface. The default is `inherit`, except for root node which will have value based on the current locale. See https://www.yogalayout.dev/docs/styling/layout-direction for more details.

| Type                          | Required | Platform |
| ----------------------------- | -------- | -------- |
| enum('inherit', 'ltr', 'rtl') | No       | iOS      |

---

### `display`

`display` 設定此元件的顯示類型。

其功能類似 CSS 中的 `display`，但僅支援 'flex' 和 'none'，預設值為 'flex'。

| Type                 | Required |
| -------------------- | -------- |
| enum('none', 'flex') | No       |

---

### `end`

當方向為 `ltr` 時，`end` 等同於 `right`；當方向為 `rtl` 時，`end` 等同於 `left`。

此樣式優先於 `left` 和 `right` 樣式。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `flex`

在 React Native 中，`flex` 的行為與 CSS 不同。`flex` 是一個數字而非字串，其運作遵循 [Yoga](https://github.com/facebook/yoga) 佈局引擎。

When `flex` is a positive number, it makes the component flexible, and it will be sized proportional to its flex value. So a component with `flex` set to 2 will take twice the space as a component with `flex` set to 1. `flex: <positive number>` equates to `flexGrow: <positive number>, flexShrink: 1, flexBasis: 0`.

當 `flex` 為 0 時，元件大小由 `width` 和 `height` 決定，且不具彈性。

當 `flex` 為 -1 時，元件通常按 `width` 和 `height` 設定大小，但若空間不足，會縮小至 `minWidth` 和 `minHeight`。

`flexGrow`、`flexShrink` 和 `flexBasis` 的行為與 CSS 相同。

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

`flexGrow` describes how any space within a container should be distributed among its children along the main axis. After laying out its children, a container will distribute any remaining space according to the flex grow values specified by its children.

`flexGrow` 接受任何大於或等於 0 的浮點數，預設值為 0。容器會根據子項目的 `flexGrow` 值加權分配剩餘空間。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `flexShrink`

[`flexShrink`](layout-props#flexshrink) 描述當子項目在主軸上的總大小超出容器大小時，應如何沿主軸縮小子項目。`flexShrink` 與 `flexGrow` 非常相似，可以將溢出的空間視為負的剩餘空間來理解。這兩個屬性可以很好地配合使用，讓子項目根據需要進行擴展或縮小。

`flexShrink` 接受任何大於或等於 0 的浮點數，預設值為 0。容器會根據子項目的 `flexShrink` 值加權縮小子項目。

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

其功能類似 CSS 中的 `left` 屬性，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

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

When direction is `ltr`, `marginEnd` is equivalent to `marginRight`. When direction is `rtl`, `marginEnd` is equivalent to `marginLeft`.

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

When direction is `ltr`, `marginStart` is equivalent to `marginLeft`. When direction is `rtl`, `marginStart` is equivalent to `marginRight`.

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

其功能類似 CSS 中的 `max-height` 屬性，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

See https://developer.mozilla.org/en-US/docs/Web/CSS/max-height for more details.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `maxWidth`

`maxWidth` 是此元件的最大寬度，以邏輯像素為單位。

其功能類似 CSS 中的 `max-width` 屬性，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

See https://developer.mozilla.org/en-US/docs/Web/CSS/max-width for more details.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `minHeight`

`minHeight` 是此元件的最小高度，以邏輯像素為單位。

其功能類似於 CSS 中的 `min-height`，但在 React Native 中必須使用點數或百分比，不支援 ems 等其他單位。

詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/min-height 以獲取更多資訊。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `minWidth`

`minWidth` 是此元件的最小寬度，以邏輯像素為單位。

其功能類似於 CSS 中的 `min-width`，但在 React Native 中必須使用點數或百分比，不支援 ems 等其他單位。

詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/min-width 以獲取更多資訊。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `overflow`

`overflow` 控制子元件的測量與顯示方式。`overflow: hidden` 會裁剪視圖，而 `overflow: scroll` 會使視圖獨立於其父元件的主軸進行測量。其功能類似於 CSS 中的 `overflow`（預設值為 visible）。詳見 https://developer.mozilla.org/en/docs/Web/CSS/overflow 以獲取更多資訊。

| Type                                | Required |
| ----------------------------------- | -------- |
| enum('visible', 'hidden', 'scroll') | No       |

---

### `padding`

設定 `padding` 的效果等同於分別設定 `paddingTop`、`paddingBottom`、`paddingLeft` 和 `paddingRight`。詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/padding 以獲取更多資訊。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `paddingBottom`

`paddingBottom` 的功能類似於 CSS 中的 `padding-bottom`。詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/padding-bottom 以獲取更多資訊。

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

`paddingLeft` 的功能類似於 CSS 中的 `padding-left`。詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/padding-left 以獲取更多資訊。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `paddingRight`

`paddingRight` 的功能類似於 CSS 中的 `padding-right`。詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/padding-right 以獲取更多資訊。

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

`paddingTop` 的功能類似於 CSS 中的 `padding-top`。詳見 https://developer.mozilla.org/en-US/docs/Web/CSS/padding-top 以獲取更多資訊。

| Type            | Required |
| --------------- | -------- |
| number, ,string | No       |

---

### `paddingVertical`

設定 `paddingVertical` 等同於同時設定 `paddingTop` 和 `paddingBottom`。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `position`

`position` in React Native is similar to regular CSS, but everything is set to `relative` by default, so `absolute` positioning is always relative to the parent.

若需以邏輯像素數值相對於父元素定位子元素，請將子元素設為 `absolute` 定位。

若需相對於非父元素的物件定位子元素，請勿使用樣式實現，應透過元件樹結構處理。

詳見 https://github.com/facebook/yoga 瞭解 React Native 與 CSS 在 `position` 屬性上的差異。

| Type                         | Required |
| ---------------------------- | -------- |
| enum('absolute', 'relative') | No       |

---

### `right`

`right` 表示以邏輯像素為單位偏移元件右邊緣的距離。

功能類似 CSS 的 `right` 屬性，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

參閱 https://developer.mozilla.org/en-US/docs/Web/CSS/right 瞭解 `right` 如何影響佈局。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `start`

當方向為 `ltr` 時，`start` 等同於 `left`；當方向為 `rtl` 時，`start` 等同於 `right`。

此樣式優先級高於 `left`、`right` 和 `end` 樣式。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `top`

`top` 表示以邏輯像素為單位偏移元件頂邊緣的距離。

功能類似 CSS 的 `top` 屬性，但在 React Native 中必須使用點數或百分比單位，不支援 em 等其他單位。

參閱 https://developer.mozilla.org/en-US/docs/Web/CSS/top 瞭解 `top` 如何影響佈局。

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `width`

`width` 設定元件的寬度。

It works similarly to `width` in CSS, but in React Native you must use points or percentages. Ems and other units are not supported. See https://developer.mozilla.org/en-US/docs/Web/CSS/width for more details.

| Type           | Required |
| -------------- | -------- |
| number, string | No       |

---

### `zIndex`

`zIndex` controls which components display on top of others. Normally, you don't use `zIndex`. Components render according to their order in the document tree, so later components draw over earlier ones. `zIndex` may be useful if you have animations or custom modal interfaces where you don't want this behavior.

It works like the CSS `z-index` property - components with a larger `zIndex` will render on top. Think of the z-direction like it's pointing from the phone into your eyeball. See https://developer.mozilla.org/en-US/docs/Web/CSS/z-index for more details.

在 iOS 上，`zIndex` 可能需要多個 `View` 元件互為兄弟節點才能正常運作。

| Type   | Required |
| ------ | -------- |
| number | No       |

---