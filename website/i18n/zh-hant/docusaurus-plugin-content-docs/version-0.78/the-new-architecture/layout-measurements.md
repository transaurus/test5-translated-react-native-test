# 測量佈局

有時，您需要測量當前的佈局，以便對整體佈局進行調整，或根據測量結果做出決策並調用特定邏輯。

React Native 提供了一些原生方法來獲取視圖的測量值。

調用這些方法的最佳時機是在 `useLayoutEffect` 鉤子中：這將為您提供最新的測量值，並允許您在同一幀中應用變更，而測量值正是在該幀中計算的。

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

確定給定視圖在視口中的位置（`x` 和 `y`）、`width` 和 `height`。通過異步回調返回這些值。如果成功，回調將被調用並傳入以下參數：

- `x`：被測量視圖的原點（左上角）在視口中的 `x` 坐標。
- `y`：被測量視圖的原點（左上角）在視口中的 `y` 坐標。
- `width`：視圖的 `width`。
- `height`：視圖的 `height`。
- `pageX`：視圖在視口（通常是整個屏幕）中的 `x` 坐標。
- `pageY`：視圖在視口（通常是整個屏幕）中的 `y` 坐標。

Also the `width` and `height` returned by `measure()` are the `width` and `height` of the component in the viewport.

## measureInWindow(callback)

確定給定視圖在當前窗口中的位置（`x` 和 `y`），並通過異步回調返回這些值。如果 React 根視圖嵌入在另一個原生視圖中，這將為您提供絕對坐標。如果成功，回調將被調用並傳入以下參數：

- `x`：視圖在當前窗口中的 `x` 坐標。
- `y`：視圖在當前窗口中的 `y` 坐標。
- `width`：視圖的 `width`。
- `height`：視圖的 `height`。