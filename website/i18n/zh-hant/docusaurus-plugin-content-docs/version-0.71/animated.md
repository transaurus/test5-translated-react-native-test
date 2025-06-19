---
id: animated
title: Animated
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

The `Animated` library is designed to make animations fluid, powerful, and painless to build and maintain. `Animated` focuses on declarative relationships between inputs and outputs, configurable transforms in between, and `start`/`stop` methods to control time-based animation execution.

建立動畫的核心流程是：創建一個 `Animated.Value`，將其綁定至動畫元件的一個或多個樣式屬性，再透過 `Animated.timing()` 驅動動畫更新。

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

> Don't modify the animated value directly. You can use the [`useRef` Hook](https://reactjs.org/docs/hooks-reference.html#useref) to return a mutable ref object. This ref object's `current` property is initialized as the given argument and persists throughout the component lifecycle.

</TabItem>
<TabItem value="classical">

> Don't modify the animated value directly. It is usually stored as a [state variable](intro-react#state) in class components.

</TabItem>
</Tabs>

## 範例

The following example contains a `View` which will fade in and fade out based on the animated value `fadeAnim`

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Animate&supportedPlatforms=ios,android
import React from 'react';
import {
  Animated,
  Text,
  View,
  StyleSheet,
  Button,
  SafeAreaView,
  useAnimatedValue,
} from 'react-native';

const App = () => {
  // fadeAnim will be used as the value for opacity. Initial Value: 0
  const fadeAnim = useAnimatedValue(0);

  const fadeIn = () => {
    // Will change fadeAnim value to 1 in 5 seconds
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 5000,
      useNativeDriver: true,
    }).start();
  };

  const fadeOut = () => {
    // Will change fadeAnim value to 0 in 3 seconds
    Animated.timing(fadeAnim, {
      toValue: 0,
      duration: 3000,
      useNativeDriver: true,
    }).start();
  };

  return (
    <SafeAreaView style={styles.container}>
      <Animated.View
        style={[
          styles.fadingContainer,
          {
            // Bind opacity to animated value
            opacity: fadeAnim,
          },
        ]}>
        <Text style={styles.fadingText}>Fading View!</Text>
      </Animated.View>
      <View style={styles.buttonRow}>
        <Button title="Fade In View" onPress={fadeIn} />
        <Button title="Fade Out View" onPress={fadeOut} />
      </View>
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  fadingContainer: {
    padding: 20,
    backgroundColor: 'powderblue',
  },
  fadingText: {
    fontSize: 28,
  },
  buttonRow: {
    flexBasis: 100,
    justifyContent: 'space-evenly',
    marginVertical: 16,
  },
});

export default App;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=Animated
import React, {Component} from 'react';
import {
  Animated,
  Text,
  View,
  StyleSheet,
  Button,
  SafeAreaView,
} from 'react-native';

class App extends Component {
  // fadeAnim will be used as the value for opacity. Initial Value: 0
  state = {
    fadeAnim: new Animated.Value(0),
  };

  fadeIn = () => {
    // Will change fadeAnim value to 1 in 5 seconds
    Animated.timing(this.state.fadeAnim, {
      toValue: 1,
      duration: 5000,
      useNativeDriver: true,
    }).start();
  };

  fadeOut = () => {
    // Will change fadeAnim value to 0 in 3 seconds
    Animated.timing(this.state.fadeAnim, {
      toValue: 0,
      duration: 3000,
      useNativeDriver: true,
    }).start();
  };

  render() {
    return (
      <SafeAreaView style={styles.container}>
        <Animated.View
          style={[
            styles.fadingContainer,
            {
              // Bind opacity to animated value
              opacity: this.state.fadeAnim,
            },
          ]}>
          <Text style={styles.fadingText}>Fading View!</Text>
        </Animated.View>
        <View style={styles.buttonRow}>
          <Button title="Fade In View" onPress={this.fadeIn} />
          <Button title="Fade Out View" onPress={this.fadeOut} />
        </View>
      </SafeAreaView>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  fadingContainer: {
    padding: 20,
    backgroundColor: 'powderblue',
  },
  fadingText: {
    fontSize: 28,
  },
  buttonRow: {
    flexBasis: 100,
    justifyContent: 'space-evenly',
    marginVertical: 16,
  },
});

export default App;
```

</TabItem>
</Tabs>

Refer to the [Animations](animations#animated-api) guide to see additional examples of `Animated` in action.

## 概覽

There are two value types you can use with `Animated`:

- [`Animated.Value()`](animated#value) 用於單一數值
- [`Animated.ValueXY()`](animated#valuexy) 用於向量

`Animated.Value` 可綁定至樣式屬性或其他參數，並支援插值運算。單一 `Animated.Value` 可驅動任意數量的屬性。

### 動畫配置

`Animated` 提供三種動畫類型，每種類型皆提供特定的動畫曲線來控制數值如何從初始值變化至最終值：

- [`Animated.decay()`](animated#decay) 以初始速度開始並逐漸減速至停止
- [`Animated.spring()`](animated#spring) 提供基礎彈簧物理模型
- [`Animated.timing()`](animated#timing) 使用[緩動函式](easing)隨時間變化數值

多數情況下會使用 `timing()`。預設採用對稱的 easeInOut 曲線，模擬物體逐漸加速至全速後再減速停止的過程。

### 動畫操作

透過呼叫動畫實例的 `start()` 啟動動畫。`start()` 接受一個完成回調函式，當動畫結束時會被呼叫。若動畫正常完成，回調函式會收到 `{finished: true}`；若因中途呼叫 `stop()` 而中止（例如被手勢或其他動畫打斷），則會收到 `{finished: false}`。

```tsx
Animated.timing({}).start(({finished}) => {
  /* completion callback */
});
```

### 使用原生驅動

啟用原生驅動後，系統會在動畫開始前將所有參數傳送至原生端，讓原生代碼能在 UI 線程執行動畫，無需每幀都透過橋接器溝通。動畫啟動後，即使 JavaScript 線程阻塞也不會影響動畫表現。

在動畫配置中指定 `useNativeDriver: true` 即可啟用原生驅動。詳見[動畫指南](animations#using-the-native-driver)。

### 可動畫化元件

僅有可動畫化元件能套用動畫效果。這些特殊元件會將動畫值綁定至屬性，並透過針對性的原生更新避免每幀都觸發 React 渲染與協調過程的開銷，同時會在卸載時自動清理資源確保安全性。

- [`createAnimatedComponent()`](animated#createanimatedcomponent) can be used to make a component animatable.

`Animated` 導出以下透過該封裝器處理的可動畫化元件：

- `Animated.Image`
- `Animated.ScrollView`
- `Animated.Text`
- `Animated.View`
- `Animated.FlatList`
- `Animated.SectionList`

### 動畫組合

還可透過組合函式實現複雜的動畫效果組合：

- [`Animated.delay()`](animated#delay) 在給定延遲後開始動畫。
- [`Animated.parallel()`](animated#parallel) 同時啟動多個動畫。
- [`Animated.sequence()`](animated#sequence) 按順序啟動動畫，等待前一個完成後才開始下一個。
- [`Animated.stagger()`](animated#stagger) 按順序且並行啟動動畫，但具有連續的延遲。

動畫也可以通過將一個動畫的 `toValue` 設置為另一個 `Animated.Value` 來鏈接在一起。詳見動畫指南中的[追蹤動態值](animations#tracking-dynamic-values)。

默認情況下，如果一個動畫被停止或中斷，則組中的所有其他動畫也會停止。

### 組合動畫值

您可以通過加法、減法、乘法、除法或取模來組合兩個動畫值，以創建一個新的動畫值：

- [`Animated.add()`](animated#add)
- [`Animated.subtract()`](animated#subtract)
- [`Animated.divide()`](animated#divide)
- [`Animated.modulo()`](animated#modulo)
- [`Animated.multiply()`](animated#multiply)

### 插值

The `interpolate()` function allows input ranges to map to different output ranges. By default, it will extrapolate the curve beyond the ranges given, but you can also have it clamp the output value. It uses linear interpolation by default but also supports easing functions.

- [`interpolate()`](animated#interpolate)

詳見[動畫](animations#interpolation)指南中有關插值的更多內容。

### 處理手勢和其他事件

手勢（如平移或滾動）和其他事件可以使用 `Animated.event()` 直接映射到動畫值。這是通過結構化映射語法完成的，以便可以從複雜的事件對象中提取值。第一層是一個數組，允許跨多個參數進行映射，該數組包含嵌套對象。

- [`Animated.event()`](animated#event)

例如，在處理水平滾動手勢時，您可以執行以下操作，將 `event.nativeEvent.contentOffset.x` 映射到 `scrollX`（一個 `Animated.Value`）：

```tsx
 onScroll={Animated.event(
   // scrollX = e.nativeEvent.contentOffset.x
   [{nativeEvent: {
        contentOffset: {
          x: scrollX
        }
      }
    }]
 )}
```

---

# 參考

## 方法

當給定的值是 ValueXY 而不是 Value 時，每個配置選項可以是向量形式 `{x: ..., y: ...}` 而不是標量。

### `decay()`

```tsx
static decay(value, config): CompositeAnimation;
```

根據衰減係數，從初始速度到零動畫化一個值。

配置是一個對象，可能具有以下選項：

- `velocity`: 初始速度。必需。
- `deceleration`: 衰減率。默認 0.997。
- `isInteraction`: 此動畫是否在 `InteractionManager` 上創建「交互句柄」。默認為 true。
- `useNativeDriver`: 為 true 時使用原生驅動。必需。

---

### `timing()`

```tsx
static timing(value, config): CompositeAnimation;
```

沿著定時緩動曲線動畫化一個值。[`Easing`](easing) 模塊有大量預定義曲線，或者您可以使用自己的函數。

配置是一個對象，可能具有以下選項：

- `duration`: 動畫長度（毫秒）。默認 500。
- `easing`: 定義曲線的緩動函數。默認為 `Easing.inOut(Easing.ease)`。
- `delay`: 延遲後開始動畫（毫秒）。默認 0。
- `isInteraction`: 此動畫是否在 `InteractionManager` 上創建「交互句柄」。默認為 true。
- `useNativeDriver`: 為 true 時使用原生驅動。必需。

---

### `spring()`

```tsx
static spring(value, config): CompositeAnimation;
```

根據[阻尼諧振子](https://en.wikipedia.org/wiki/Harmonic_oscillator#Damped_harmonic_oscillator)的彈簧物理模型驅動數值動畫。通過追蹤速度狀態來在`toValue`更新時產生流暢運動效果，並可串聯使用。

配置參數為一個物件，可包含以下選項：

注意：只能選擇定義以下任一組參數，不可混用：

The friction/tension or bounciness/speed options match the spring model in [`Facebook Pop`](https://github.com/facebook/pop), [Rebound](https://github.com/facebookarchive/rebound), and [Origami](http://origami.design/).

- `friction`：控制「彈性」/過衝。預設值7。
- `tension`：控制動畫速度。預設值40。
- `speed`：控制動畫速度。預設值12。
- `bounciness`：控制彈性。預設值8。

使用剛度/阻尼/質量參數時，`Animated.spring`將採用基於[阻尼諧振子運動方程](https://en.wikipedia.org/wiki/Harmonic_oscillator#Damped_harmonic_oscillator)的解析彈簧模型。此行為更精確地模擬了彈簧動力學的物理特性，並高度還原iOS中CASpringAnimation的實現。

- `stiffness`：彈簧剛度係數。預設值100。
- `damping`：定義因摩擦力導致的運動阻尼程度。預設值10。
- `mass`：連接在彈簧末端的物體質量。預設值1。

其他配置選項如下：

- `velocity`：連接彈簧物體的初始速度。預設0（物體靜止）。
- `overshootClamping`：布林值，決定彈簧是否應箝制而不反彈。預設false。
- `restDisplacementThreshold`：判定彈簧靜止的位移閾值。預設0.001。
- `restSpeedThreshold`：判定彈簧靜止的速度閾值（像素/秒）。預設0.001。
- `delay`：延遲開始動畫的時間（毫秒）。預設0。
- `isInteraction`：是否在`InteractionManager`上建立「互動句柄」。預設true。
- `useNativeDriver`：為true時使用原生驅動。必需參數。

---

### `add()`

```tsx
static add(a: Animated, b: Animated): AnimatedAddition;
```

創建由兩個動畫值相加組成的新動畫值。

---

### `subtract()`

```tsx
static subtract(a: Animated, b: Animated): AnimatedSubtraction;
```

創建由第一個動畫值減去第二個動畫值組成的新動畫值。

---

### `divide()`

```tsx
static divide(a: Animated, b: Animated): AnimatedDivision;
```

創建由第一個動畫值除以第二個動畫值組成的新動畫值。

---

### `multiply()`

```tsx
static multiply(a: Animated, b: Animated): AnimatedMultiplication;
```

創建由兩個動畫值相乘組成的新動畫值。

---

### `modulo()`

```tsx
static modulo(a: Animated, modulus: number): AnimatedModulo;
```

創建提供動畫值的非負模數新動畫值。

---

### `diffClamp()`

```tsx
static diffClamp(a: Animated, min: number, max: number): AnimatedDiffClamp;
```

創建限制在兩個值之間的新動畫值。採用差值計算方式，即使當前值遠離邊界，當值再次接近時仍會開始變化（計算公式：`value = clamp(value + diff, min, max)`）。

這在處理滾動事件時非常有用，例如在向上滾動時顯示導航欄，向下滾動時隱藏它。

---

### `delay()`

```tsx
static delay(time: number): CompositeAnimation;
```

在給定的延遲後開始動畫。

---

### `sequence()`

```tsx
static sequence(animations: CompositeAnimation[]): CompositeAnimation;
```

按順序啟動一系列動畫，等待每個動畫完成後再開始下一個。如果當前運行的動畫被停止，後續動畫將不會啟動。

---

### `parallel()`

```tsx
static parallel(
  animations: CompositeAnimation[],
  config?: ParallelConfig
): CompositeAnimation;
```

同時啟動一系列動畫。默認情況下，如果其中一個動畫被停止，所有動畫都會停止。可以通過設置 `stopTogether` 標誌來覆蓋此行為。

---

### `stagger()`

```tsx
static stagger(
  time: number,
  animations: CompositeAnimation[]
): CompositeAnimation;
```

動畫陣列可以並行運行（重疊），但會以連續的延遲依次啟動。適合實現拖尾效果。

---

### `loop()`

```tsx
static loop(
  animation: CompositeAnimation[],
  config?: LoopAnimationConfig
): CompositeAnimation;
```

循環播放指定的動畫，每次到達結尾時重置並從頭開始。如果子動畫設置為 `useNativeDriver: true`，則不會阻塞 JS 線程。此外，循環可能會阻止基於 `VirtualizedList` 的組件在動畫運行時渲染更多行。可以在子動畫配置中傳遞 `isInteraction: false` 來解決此問題。

配置對象可能包含以下選項：

- `iterations`: 動畫應循環的次數。默認為 `-1`（無限）。

---

### `event()`

```tsx
static event(
  argMapping: Mapping[],
  config?: EventConfig
): (...args: any[]) => void;
```

接收一個映射陣列，並從每個參數中提取相應的值，然後在映射的輸出上調用 `setValue`。例如：

```tsx
onScroll={Animated.event(
  [{nativeEvent: {contentOffset: {x: this._scrollX}}}],
  {listener: (event: ScrollEvent) => console.log(event)}, // Optional async listener
)}
 ...
onPanResponderMove: Animated.event(
  [
    null, // raw event arg ignored
    {dx: this._panX},
  ], // gestureState arg
  {
    listener: (
      event: GestureResponderEvent,
      gestureState: PanResponderGestureState
    ) => console.log(event, gestureState),
  } // Optional async listener
);
```

配置對象可能包含以下選項：

- `listener`: 可選的異步監聽器。
- `useNativeDriver`: 為 true 時使用原生驅動。必需。

---

### `forkEvent()`

```jsx
static forkEvent(event: AnimatedEvent, listener: Function): AnimatedEvent;
```

這是一個高階的命令式 API，用於監聽通過 props 傳遞的動畫事件。它允許向現有的 `AnimatedEvent` 添加新的 JavaScript 監聽器。如果 `animatedEvent` 是一個 JavaScript 監聽器，它會將兩個監聽器合併為一個；如果 `animatedEvent` 為 null/undefined，則會直接分配 JavaScript 監聽器。盡可能直接使用值。

---

### `unforkEvent()`

```jsx
static unforkEvent(event: AnimatedEvent, listener: Function);
```

---

### `start()`

```tsx
static start(callback?: (result: {finished: boolean}) => void);
```

通過在動畫上調用 start() 來啟動動畫。start() 接收一個完成回調，該回調會在動畫完成時或在動畫完成前因調用 stop() 而被停止時調用。

**參數：**

| Name     | Type                                    | Required | Description                                                                                                                                                     |
| -------- | --------------------------------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| callback | `(result: {finished: boolean}) => void` | No       | Function that will be called after the animation finished running normally or when the animation is done because stop() was called on it before it could finish |

帶回調的啟動示例：

```tsx
Animated.timing({}).start(({finished}) => {
  /* completion callback */
});
```

---

### `stop()`

```tsx
static stop();
```

停止任何正在運行的動畫。

---

### `reset()`

```tsx
static reset();
```

停止任何正在運行的動畫並將值重置為原始值。

## 屬性

### `Value`

驅動動畫的標準值類別。通常在類別元件中初始化為 `useAnimatedValue(0)` 或 `new Animated.Value(0);`。

You can read more about `Animated.Value` API on the separate [page](animatedvalue).

---

### `ValueXY`

用於驅動 2D 動畫（例如平移手勢）的 2D 值類別。

You can read more about `Animated.ValueXY` API on the separate [page](animatedvaluexy).

---

### `Interpolation`

導出以在流程中使用 Interpolation 類型。

---

### `Node`

導出以便於類型檢查。所有動畫值均派生自此類別。

---

### `createAnimatedComponent`

使任何 React 元件可動畫化。用於創建 `Animated.View` 等。

---

### `attachNativeEvent`

Imperative API to attach an animated value to an event on a view. Prefer using `Animated.event` with `useNativeDrive: true` if possible.