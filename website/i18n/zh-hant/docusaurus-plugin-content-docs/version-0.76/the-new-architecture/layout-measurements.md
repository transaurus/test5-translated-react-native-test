# 測量佈局

有時，您需要測量當前佈局以對整體佈局進行調整，或根據測量結果做出決策並調用特定邏輯。

React Native 提供了一些原生方法來獲取視圖的尺寸資訊。

調用這些方法的最佳時機是在 `useLayoutEffect` 鉤子中：這能確保您獲得最新的測量值，並允許您在同一幀中應用基於這些測量的變更。

典型程式碼範例如下：

```tsx
function AComponent(children) {
  const targetRef = React.useRef(null)

  useLayoutEffect(() => {
    targetRef.current?.measure(({measurements}) => {
      //do something with the `measurements`
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

測量視圖在當前視窗中的絕對位置（`x` 和 `y`），通過異步回調返回結果。當 React 根視圖嵌入其他原生視圖時，此方法可獲取絕對座標。成功時回調參數包括：

- `x`: the `x` coordinate of the view in the current window.
- `y`: the `y` coordinate of the view in the current window.
- `width`: the `width` of the view.
- `height`: the `height` of the view.