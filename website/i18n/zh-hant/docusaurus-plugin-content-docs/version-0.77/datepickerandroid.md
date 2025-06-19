---
id: datepickerandroid
title: '🚧 DatePickerAndroid'
---

> **已移除。** 請改用其中一個[社群套件](https://reactnative.directory/?search=datepicker)。

開啟標準的 Android 日期選擇器對話框。

### 範例

```jsx
try {
  const {action, year, month, day} = await DatePickerAndroid.open(
    {
      // Use `new Date()` for current date.
      // May 25 2020. Month 0 is January.
      date: new Date(2020, 4, 25),
    },
  );
  if (action !== DatePickerAndroid.dismissedAction) {
    // Selected year, month (0-11), day
  }
} catch ({code, message}) {
  console.warn('Cannot open date picker', message);
}
```

---

# 參考

## 方法

### `open()`

```jsx
static open(options)
```

開啟標準的 Android 日期選擇器對話框。

The available keys for the `options` object are:

- `date`（`Date` 物件或毫秒時間戳記）- 預設顯示的日期
- `minDate`（`Date` 或毫秒時間戳記）- 可選擇的最小日期
- `maxDate`（`Date` 物件或毫秒時間戳記）- 可選擇的最大日期
- `mode`（`enum('calendar', 'spinner', 'default')`）- 設定日期選擇器的模式為 calendar/spinner/default
  - 'calendar'：以日曆模式顯示日期選擇器。
  - 'spinner'：以滾輪模式顯示日期選擇器。
  - 'default'：根據 Android 版本顯示預設的原生日期選擇器（滾輪/日曆）。

Returns a Promise which will be invoked an object containing `action`, `year`, `month` (0-11), `day` if the user picked a date. If the user dismissed the dialog, the Promise will still be resolved with action being `DatePickerAndroid.dismissedAction` and all the other keys being undefined. **Always** check whether the `action` is equal to `DatePickerAndroid.dateSetAction` before reading the values.

請注意，在 Android 4 及更低版本上，使用 `minDate` 和 `maxDate` 選項時，原生日期選擇器對話框可能會有一些 UI 問題。

---

### `dateSetAction()`

```jsx
static dateSetAction()
```

已選擇日期。

---

### `dismissedAction()`

```jsx
static dismissedAction()
```

對話框已關閉。