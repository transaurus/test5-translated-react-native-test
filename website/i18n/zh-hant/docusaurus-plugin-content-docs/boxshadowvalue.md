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

X 軸偏移量。可為正或負值，正值表示向右偏移，負值表示向左偏移。

| Type             | Optional |
| ---------------- | -------- |
| number \| string | No       |

### `offsetY`

Y 軸偏移量。可為正或負值，正值表示向上偏移，負值表示向下偏移。

| Type             | Optional |
| ---------------- | -------- |
| number \| string | No       |

### `blurRadius`

代表 [高斯模糊](https://en.wikipedia.org/wiki/Gaussian_blur) 演算法使用的半徑。數值越大陰影越模糊，僅允許非負值，預設為 0。

| Type            | Optional |
| --------------- | -------- |
| numer \| string | Yes      |

### `spreadDistance`

控制陰影擴張或收縮的程度。正值會擴張陰影，負值會收縮陰影。

| Type            | Optional |
| --------------- | -------- |
| numer \| string | Yes      |

### `color`

陰影顏色，預設為 `black`（黑色）。

| Type                 | Optional |
| -------------------- | -------- |
| [color](./colors.md) | Yes      |

### `inset`

是否為內陰影。內陰影會出現在元素邊框盒的內側，而非外側。

| Type    | Optional |
| ------- | -------- |
| boolean | Yes      |

## 使用場景

- [`boxShadow`](./view-style-props.md#boxshadow)