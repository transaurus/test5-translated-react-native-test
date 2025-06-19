---
id: slider
title: '🚧 Slider'
---

> **已棄用。** 請改用[社群套件](https://reactnative.directory/?search=slider)中的其中一個。

用於從數值範圍中選擇單一值的元件。

---

# 參考

## 屬性

繼承自[View 屬性](view.md#props)。

### `style`

用於設定 `Slider` 的樣式與佈局。詳見 `StyleSheet.js` 與 `ViewStylePropTypes.js`。

| Type       | Required |
| ---------- | -------- |
| View.style | No       |

---

### `disabled`

若為 true，使用者將無法移動滑桿。預設值為 false。

| Type | Required |
| ---- | -------- |
| bool | No       |

---

### `maximumValue`

滑桿的初始最大值。預設值為 1。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `minimumTrackTintColor`

用於按鈕左側軌道的顏色。在 iOS 上會覆蓋預設的藍色漸層圖片。

| Type               | Required |
| ------------------ | -------- |
| [color](colors.md) | No       |

---

### `minimumValue`

滑桿的初始最小值。預設值為 0。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `onSlidingComplete`

當使用者放開滑桿時觸發的回呼函式，無論數值是否改變。當前值會作為參數傳遞給回呼處理函式。

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `onValueChange`

在使用者拖曳滑桿時持續觸發的回呼函式。

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `step`

滑桿的步進值。該值應介於 0 到 (maximumValue - minimumValue) 之間。預設值為 0。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `maximumTrackTintColor`

用於按鈕右側軌道的顏色。在 iOS 上會覆蓋預設的灰色漸層圖片。

| Type               | Required |
| ------------------ | -------- |
| [color](colors.md) | No       |

---

### `testID`

用於在 UI 自動化測試中定位此視圖。

| Type   | Required |
| ------ | -------- |
| string | No       |

---

### `value`

滑桿的初始值。該值應介於 minimumValue 與 maximumValue 之間，預設分別為 0 和 1。預設值為 0。

_此元件並非受控元件_，您無需在拖曳過程中更新數值。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `thumbTintColor`

在 iOS 上用於著色預設拇指圖片的顏色，或在 Android 上用於前景開關握柄的顏色。

| Type               | Required |
| ------------------ | -------- |
| [color](colors.md) | No       |

---

### `maximumTrackImage`

指定最大軌道圖片。僅支援靜態圖片。圖片的最左側像素將被拉伸以填滿軌道。

| Type                   | Required | Platform |
| ---------------------- | -------- | -------- |
| Image.propTypes.source | No       | iOS      |

---

### `minimumTrackImage`

指定滑桿的最小軌道圖像。僅支援靜態圖像。圖像最右側的像素將被拉伸以填滿軌道。

| Type                   | Required | Platform |
| ---------------------- | -------- | -------- |
| Image.propTypes.source | No       | iOS      |

---

### `thumbImage`

設定滑桿拇指的圖像。僅支援靜態圖像。

| Type                   | Required | Platform |
| ---------------------- | -------- | -------- |
| Image.propTypes.source | No       | iOS      |

---

### `trackImage`

為滑桿軌道指定單一圖像。僅支援靜態圖像。圖像的中心像素將被拉伸以填滿軌道。

| Type                   | Required | Platform |
| ---------------------- | -------- | -------- |
| Image.propTypes.source | No       | iOS      |