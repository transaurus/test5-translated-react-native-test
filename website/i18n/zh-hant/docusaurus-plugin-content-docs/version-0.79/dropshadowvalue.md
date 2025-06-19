---
id: dropshadowvalue
title: DropShadowValue Object Type
---

The `DropShadowValue` object is taken by the [`filter`](./view-style-props.md#filter) style prop for the `dropShadow` function. It is comprised of 2 or 3 lengths and an optional color. These values collectively define the drop shadow's color, position, and blurriness.

## 範例

```js
{
  offsetX: 10,
  offsetY: -3,
  standardDeviation: '15px',
  color: 'blue',
}
```

## 鍵與值

### `offsetX`

x 軸上的偏移量。可以是正值或負值。正值表示向右，負值表示向左。

| Type             | Optional |
| ---------------- | -------- |
| number \| string | No       |

### `offsetY`

y 軸上的偏移量。可以是正值或負值。正值表示向上，負值表示向下。

| Type             | Optional |
| ---------------- | -------- |
| number \| string | No       |

### `standardDeviation`

代表用於[高斯模糊](https://en.wikipedia.org/wiki/Gaussian_blur)算法的標準差。值越大，陰影越模糊。僅允許非負值，預設值為 0。

| Type            | Optional |
| --------------- | -------- |
| numer \| string | Yes      |

### `color`

陰影的顏色。預設為 `black`（黑色）。

| Type                 | Optional |
| -------------------- | -------- |
| [color](./colors.md) | Yes      |

## 使用場景

- [`filter`](./view-style-props.md#filter)