---
id: layoutevent
title: LayoutEvent Object Type
---

`LayoutEvent` object is returned in the callback as a result of component layout change, for example `onLayout` in [View](view) component.

## 範例

```js
{
    layout: {
        width: 520,
        height: 70.5,
        x: 0,
        y: 42.5
    },
    target: 1127
}
```

## 鍵值與數值

### `height`

元件佈局變化後的高度。

| Type   | Optional |
| ------ | -------- |
| number | No       |

### `width`

元件佈局變化後的寬度。

| Type   | Optional |
| ------ | -------- |
| number | No       |

### `x`

元件在父元件內的 X 座標。

| Type   | Optional |
| ------ | -------- |
| number | No       |

### `y`

元件在父元件內的 Y 座標。

| Type   | Optional |
| ------ | -------- |
| number | No       |

### `target`

接收 PressEvent 的元素的節點 ID。

| Type                        | Optional |
| --------------------------- | -------- |
| number, `null`, `undefined` | No       |

## 使用此物件的元件

- [`Image`](image)
- [`Pressable`](pressable)
- [`ScrollView`](scrollview)
- [`Text`](text)
- [`TextInput`](textinput)
- [`TouchableWithoutFeedback`](touchablewithoutfeedback)
- [`View`](view)