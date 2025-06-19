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

## 鍵與值

### `offsetX`

x 軸上的偏移量。可以是正數或負數。正值表示向右偏移，負值表示向左偏移。

| Type             | Optional |
| ---------------- | -------- |
| number \| string | No       |

### `offsetY`

y 軸上的偏移量。可以是正數或負數。正值表示向上偏移，負值表示向下偏移。

| Type             | Optional |
| ---------------- | -------- |
| number \| string | No       |

### `blurRadius`

表示用於[高斯模糊](https://en.wikipedia.org/wiki/Gaussian_blur)演算法的半徑。值越大，陰影越模糊。僅允許非負值。預設值為 0。

| Type            | Optional |
| --------------- | -------- |
| numer \| string | Yes      |

### `spreadDistance`

陰影放大或縮小的程度。正值會放大陰影，負值會縮小陰影。

| Type            | Optional |
| --------------- | -------- |
| numer \| string | Yes      |

### `color`

陰影的顏色。預設為 `black`（黑色）。

| Type                 | Optional |
| -------------------- | -------- |
| [color](./colors.md) | Yes      |

### `inset`

陰影是否為內嵌。內嵌陰影會出現在元素邊框盒的內部，而非外部。

| Type    | Optional |
| ------- | -------- |
| boolean | Yes      |

## 使用於

- [`boxShadow`](./view-style-props.md#boxshadow)