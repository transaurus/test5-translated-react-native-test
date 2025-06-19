# 測量佈局

有時，您需要測量當前的佈局，以便對整體佈局進行調整或根據測量結果執行特定邏輯。

React Native 提供了一些原生方法來獲取視圖的測量數據。

調用這些方法的最佳時機是在 `useLayoutEffect` 鉤子中：這樣可以獲取最新的測量值，並在同一幀中應用變更。

典型的代碼如下所示：

```tsx
function AComponent(children) {
  const targetRef = React.useRef(null)

  useLayoutEffect(() => {
    targetRef.current?.measure((x, y, width, height, pageX, pageY) => {
      //do something with the measurements
    });
  }, [ /* add dependencies here */]);

  return (
    <View ref={targetRef}>
     {children}
    <View />
  );
}
```

:::note
The methods described here are available on most of the default components provided by React Native. However, they are _not_ available on composite components that aren't directly backed by a native view. This will generally include most components that you define in your own app.
:::

## measure(callback)

Determines the location on screen (`x` and `y`), `width`, and `height` in the viewport of the given view. Returns the values via an async callback. If successful, the callback will be called with the following arguments:

- `x`: the `x` coordinate of the origin (top-left corner) of the measured view in the viewport.
- `y`: the `y` coordinate of the origin (top-left corner) of the measured view in the viewport.
- `width`: the `width` of the view.
- `height`: the `height` of the view.
- `pageX`: the `x` coordinate of the view in the viewport (typically the whole screen).
- `pageY`: the `y` coordinate of the view in the viewport (typically the whole screen).

Also the `width` and `height` returned by `measure()` are the `width` and `height` of the component in the viewport.

## measureInWindow(callback)

確定給定視圖在當前窗口中的位置（`x` 和 `y`），並通過異步回調返回這些值。如果 React 根視圖嵌入到另一個原生視圖中，此方法將返回絕對坐標。如果成功，回調函數將接收以下參數：

- `x`: the `x` coordinate of the view in the current window.
- `y`: the `y` coordinate of the view in the current window.
- `width`: the `width` of the view.
- `height`: the `height` of the view.