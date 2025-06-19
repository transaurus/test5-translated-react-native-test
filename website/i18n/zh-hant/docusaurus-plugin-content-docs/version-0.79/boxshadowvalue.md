---
id: boxshadowvalue
title: BoxShadowValue Object Type
---

The `BoxShadowValue` object is taken by the [`boxShadow`](./view-style-props.md#boxshadow) style prop. It is comprised of 2-4 lengths, an optional color, and an optional `inset` boolean. These values collectively define the box shadow's color, position, size, and blurriness.

## 範例

```js
{
  offsetX: 10,
  offsetY: -3,
  blurRadius: '15px',
  spreadDistance: '10px',
  color: 'red',
  inset: true,
}
```

## 鍵值與說明

### `offsetX`

x 軸偏移量。可為正或負值，正值表示向右偏移，負值表示向左偏移。

| Type             | Optional |
| ---------------- | -------- |
| number \| string | No       |

### `offsetY`

y 軸偏移量。可為正或負值，正值表示向上偏移，負值表示向下偏移。

| Type             | Optional |
| ---------------- | -------- |
| number \| string | No       |

### `blurRadius`

代表[高斯模糊](https://en.wikipedia.org/wiki/Gaussian_blur)演算法中使用的半徑。數值越大陰影越模糊，僅允許非負值，預設為 0。

| Type            | Optional |
| --------------- | -------- |
| numer \| string | Yes      |

### `spreadDistance`

控制陰影擴張或收縮的程度。正值會使陰影擴大，負值會使陰影縮小。

| Type            | Optional |
| --------------- | -------- |
| numer \| string | Yes      |

### `color`

陰影顏色，預設為 `black`（黑色）。

| Type                 | Optional |
| -------------------- | -------- |
| [color](./colors.md) | Yes      |

### `inset`

設定陰影是否為內陰影。內陰影會出現在元素邊框盒的內側，而非外側。

| Type    | Optional |
| ------- | -------- |
| boolean | Yes      |

## 使用場景

- [`boxShadow`](./view-style-props.md#boxshadow)