---
id: targetevent
title: TargetEvent Object Type
---

`TargetEvent` object is returned in the callback as a result of focus change, for example `onFocus` or `onBlur` in the [TextInput](textinput) component.

## 範例

```
{
    target: 1127
}
```

## 鍵值與數值

### `target`

接收 TargetEvent 的元素的節點 ID。

| Type                        | Optional |
| --------------------------- | -------- |
| number, `null`, `undefined` | No       |

## 使用此物件的元件

- [`TextInput`](textinput)
- [`TouchableWithoutFeedback`](touchablewithoutfeedback)