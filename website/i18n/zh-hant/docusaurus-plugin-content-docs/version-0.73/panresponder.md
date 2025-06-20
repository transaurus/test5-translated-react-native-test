---
id: panresponder
title: PanResponder
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

`PanResponder` 會將多個觸控點協調為單一手勢。它使單點觸控手勢對額外觸點具有韌性，並可用於識別基本的多點觸控手勢。

預設情況下，`PanResponder` 會持有 `InteractionManager` 的控制權，以阻止長時間運行的 JavaScript 事件中斷活躍的手勢。

它為[手勢響應系統](gesture-responder-system.md)提供的響應處理器提供了一個可預測的封裝。對於每個處理器，它會在原生事件物件旁提供一個新的 `gestureState` 物件：

```
onPanResponderMove: (event, gestureState) => {}
```

原生事件是一個合成觸控事件，形式為 [PressEvent](pressevent)。

A `gestureState` object has the following:

- `stateID` - gestureState 的 ID，只要螢幕上至少有一個觸點就會持續存在
- `moveX` - 最近移動觸點的最新螢幕座標
- `moveY` - 最近移動觸點的最新螢幕座標
- `x0` - 響應授予時的螢幕座標
- `y0` - 響應授予時的螢幕座標
- `dx` - 自觸控開始以來手勢的累積距離
- `dy` - 自觸控開始以來手勢的累積距離
- `vx` - 手勢的當前速度
- `vy` - 手勢的當前速度
- `numberActiveTouches` - 當前螢幕上的觸點數量

## 使用模式

```tsx
const ExampleComponent = () => {
  const panResponder = React.useRef(
    PanResponder.create({
      // Ask to be the responder:
      onStartShouldSetPanResponder: (evt, gestureState) => true,
      onStartShouldSetPanResponderCapture: (evt, gestureState) =>
        true,
      onMoveShouldSetPanResponder: (evt, gestureState) => true,
      onMoveShouldSetPanResponderCapture: (evt, gestureState) =>
        true,

      onPanResponderGrant: (evt, gestureState) => {
        // The gesture has started. Show visual feedback so the user knows
        // what is happening!
        // gestureState.d{x,y} will be set to zero now
      },
      onPanResponderMove: (evt, gestureState) => {
        // The most recent move distance is gestureState.move{X,Y}
        // The accumulated gesture distance since becoming responder is
        // gestureState.d{x,y}
      },
      onPanResponderTerminationRequest: (evt, gestureState) =>
        true,
      onPanResponderRelease: (evt, gestureState) => {
        // The user has released all touches while this view is the
        // responder. This typically means a gesture has succeeded
      },
      onPanResponderTerminate: (evt, gestureState) => {
        // Another component has become the responder, so this gesture
        // should be cancelled
      },
      onShouldBlockNativeResponder: (evt, gestureState) => {
        // Returns whether this component should block native components from becoming the JS
        // responder. Returns true by default. Is currently only supported on android.
        return true;
      },
    }),
  ).current;

  return <View {...panResponder.panHandlers} />;
};
```

## 範例

`PanResponder` 與 `Animated` API 協同工作，幫助在 UI 中構建複雜的手勢。以下範例包含一個動畫化的 `View` 組件，可以在螢幕上自由拖動

```SnackPlayer name=PanResponder
import React, {useRef} from 'react';
import {Animated, View, StyleSheet, PanResponder, Text} from 'react-native';

const App = () => {
  const pan = useRef(new Animated.ValueXY()).current;

  const panResponder = useRef(
    PanResponder.create({
      onMoveShouldSetPanResponder: () => true,
      onPanResponderMove: Animated.event([null, {dx: pan.x, dy: pan.y}]),
      onPanResponderRelease: () => {
        pan.extractOffset();
      },
    }),
  ).current;

  return (
    <View style={styles.container}>
      <Text style={styles.titleText}>Drag this box!</Text>
      <Animated.View
        style={{
          transform: [{translateX: pan.x}, {translateY: pan.y}],
        }}
        {...panResponder.panHandlers}>
        <View style={styles.box} />
      </Animated.View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  titleText: {
    fontSize: 14,
    lineHeight: 24,
    fontWeight: 'bold',
  },
  box: {
    height: 150,
    width: 150,
    backgroundColor: 'blue',
    borderRadius: 5,
  },
});

export default App;
```

試試 [RNTester 中的 PanResponder 範例](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/examples/PanResponder/PanResponderExample.js)。

---

# 參考

## 方法

### `create()`

```tsx
static create(config: PanResponderCallbacks): PanResponderInstance;
```

**參數：**

| Name                                                        | Type   | Description |
| ----------------------------------------------------------- | ------ | ----------- |
| config <div className="label basic required">Required</div> | object | Refer below |

The `config` object provides enhanced versions of all of the responder callbacks that provide not only the [`PressEvent`](pressevent), but also the `PanResponder` gesture state, by replacing the word `Responder` with `PanResponder` in each of the typical `onResponder*` callbacks. For example, the `config` object would look like:

- `onMoveShouldSetPanResponder: (e, gestureState) => {...}`
- `onMoveShouldSetPanResponderCapture: (e, gestureState) => {...}`
- `onStartShouldSetPanResponder: (e, gestureState) => {...}`
- `onStartShouldSetPanResponderCapture: (e, gestureState) => {...}`
- `onPanResponderReject: (e, gestureState) => {...}`
- `onPanResponderGrant: (e, gestureState) => {...}`
- `onPanResponderStart: (e, gestureState) => {...}`
- `onPanResponderEnd: (e, gestureState) => {...}`
- `onPanResponderRelease: (e, gestureState) => {...}`
- `onPanResponderMove: (e, gestureState) => {...}`
- `onPanResponderTerminate: (e, gestureState) => {...}`
- `onPanResponderTerminationRequest: (e, gestureState) => {...}`
- `onShouldBlockNativeResponder: (e, gestureState) => {...}`

一般來說，對於具有捕獲階段等效事件的情況，我們會在捕獲階段更新一次 gestureState，並可以在冒泡階段同樣使用它。

使用 `onStartShould*` 回調時需謹慎。這些回調僅反映冒泡/捕獲到節點的開始/結束事件所更新的 `gestureState`。一旦節點成為響應者，您可以依賴每個開始/結束事件都由手勢處理，並且 `gestureState` 會相應更新。（numberActiveTouches）可能不完全準確，除非您是響應者。