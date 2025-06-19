---
id: animated
title: Animated
---

The `Animated` library is designed to make animations fluid, powerful, and painless to build and maintain. `Animated` focuses on declarative relationships between inputs and outputs, configurable transforms in between, and `start`/`stop` methods to control time-based animation execution.

創建動畫的核心流程是創建一個 `Animated.Value`，將其掛接到動畫元件的一個或多個樣式屬性，然後使用 `Animated.timing()` 驅動動畫更新。

> 請勿直接修改動畫值。您可以使用 [`useRef` Hook](https://reactjs.org/docs/hooks-reference.html#useref) 返回一個可變的 ref 物件。此 ref 物件的 `current` 屬性初始化為給定的參數，並在整個元件生命週期中持續存在。

## 範例

The following example contains a `View` which will fade in and fade out based on the animated value `fadeAnim`

```SnackPlayer name=Animated%20Example&supportedPlatforms=ios,android
import React from 'react';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';
import {
  Animated,
  Text,
  View,
  StyleSheet,
  Button,
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
    <SafeAreaProvider>
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
    </SafeAreaProvider>
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

請參閱 [動畫指南](animations#animated-api) 以查看 `Animated` 的更多實際範例。

## 概述

您可以與 `Animated` 一起使用的值類型有兩種：

- [`Animated.Value()`](animated#value) 用於單一值
- [`Animated.ValueXY()`](animated#valuexy) 用於向量

`Animated.Value` 可以綁定到樣式屬性或其它屬性，也可以進行插值。單個 `Animated.Value` 可以驅動任意數量的屬性。

### 配置動畫

`Animated` 提供三種動畫類型。每種動畫類型提供特定的動畫曲線，控制您的值如何從初始值動畫到最終值：

- [`Animated.decay()`](animated#decay) 以初始速度開始並逐漸減速至停止。
- [`Animated.spring()`](animated#spring) 提供基本的彈簧物理模型。
- [`Animated.timing()`](animated#timing) 使用 [緩動函數](easing) 隨時間動畫化值。

在大多數情況下，您會使用 `timing()`。默認情況下，它使用對稱的 easeInOut 曲線，傳達物體逐漸加速至全速並通過逐漸減速至停止來結束。

### 處理動畫

動畫通過在您的動畫上調用 `start()` 來啟動。`start()` 接受一個完成回調，該回調將在動畫完成時調用。如果動畫正常完成運行，完成回調將以 `{finished: true}` 調用。如果動畫因為在完成之前調用了 `stop()`（例如，因為它被手勢或另一個動畫中斷），則它將接收 `{finished: false}`。

```tsx
Animated.timing({}).start(({finished}) => {
  /* completion callback */
});
```

### 使用原生驅動

通過使用原生驅動，我們在啟動動畫之前將有關動畫的所有內容發送到原生端，允許原生代碼在 UI 線程上執行動畫，而無需在每一幀上都通過橋接。一旦動畫開始，JS 線程可以被阻塞而不影響動畫。

您可以通過在動畫配置中指定 `useNativeDriver: true` 來使用原生驅動。詳情請參閱 [動畫指南](animations#using-the-native-driver)。

### 可動畫化元件

只有可動畫化元件才能被動畫化。這些獨特的元件將動畫值綁定到屬性的魔法，並進行有針對性的原生更新，以避免每一幀上的 React 渲染和協調過程的成本。它們還在卸載時處理清理，因此默認情況下是安全的。

- [`createAnimatedComponent()`](animated#createanimatedcomponent) 可用於使元件可動畫化。

`Animated` 使用上述包裝器導出以下可動畫化元件：

- `Animated.Image`
- `Animated.ScrollView`
- `Animated.Text`
- `Animated.View`
- `Animated.FlatList`
- `Animated.SectionList`

### 組合動畫

動畫也可以透過組合函數以複雜的方式結合：

- [`Animated.delay()`](animated#delay) 在指定延遲後開始動畫。
- [`Animated.parallel()`](animated#parallel) 同時啟動多個動畫。
- [`Animated.sequence()`](animated#sequence) 按順序啟動動畫，等待前一個完成後才開始下一個。
- [`Animated.stagger()`](animated#stagger) 按順序且並行啟動動畫，但會連續延遲。

動畫也可以透過將一個動畫的 `toValue` 設為另一個 `Animated.Value` 來串聯。請參閱動畫指南中的[追蹤動態值](animations#tracking-dynamic-values)。

預設情況下，如果一個動畫被停止或中斷，群組中的所有其他動畫也會停止。

### 結合動畫值

您可以透過加法、減法、乘法、除法或取模來結合兩個動畫值，以創建新的動畫值：

- [`Animated.add()`](animated#add)
- [`Animated.subtract()`](animated#subtract)
- [`Animated.divide()`](animated#divide)
- [`Animated.modulo()`](animated#modulo)
- [`Animated.multiply()`](animated#multiply)

### 插值

The `interpolate()` function allows input ranges to map to different output ranges. By default, it will extrapolate the curve beyond the ranges given, but you can also have it clamp the output value. It uses linear interpolation by default but also supports easing functions.

- [`interpolate()`](animatedvalue#interpolate)

請參閱[動畫](animations#interpolation)指南以了解更多關於插值的資訊。

### 處理手勢和其他事件

手勢（如平移或滾動）和其他事件可以使用 `Animated.event()` 直接映射到動畫值。這是透過結構化映射語法完成的，以便可以從複雜的事件物件中提取值。第一層是一個陣列，允許跨多個參數進行映射，該陣列包含嵌套物件。

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

當給定的值是 ValueXY 而非 Value 時，每個配置選項可以是 `{x: ..., y: ...}` 形式的向量，而非純量。

### `decay()`

```tsx
static decay(value, config): CompositeAnimation;
```

根據衰減係數，從初始速度動畫化一個值到零。

配置是一個物件，可能包含以下選項：

- `velocity`: 初始速度。必填。
- `deceleration`: 衰減率。預設 0.997。
- `isInteraction`: 此動畫是否在 `InteractionManager` 上創建「互動句柄」。預設 true。
- `useNativeDriver`: 為 true 時使用原生驅動。必填。

---

### `timing()`

```tsx
static timing(value, config): CompositeAnimation;
```

沿著定時緩動曲線動畫化一個值。[`Easing`](easing) 模組有大量預定義曲線，或者您可以使用自己的函數。

配置是一個物件，可能包含以下選項：

- `duration`: 動畫長度（毫秒）。預設 500。
- `easing`: 定義曲線的緩動函數。預設為 `Easing.inOut(Easing.ease)`。
- `delay`: 在延遲後開始動畫（毫秒）。預設 0。
- `isInteraction`: 此動畫是否在 `InteractionManager` 上創建「互動句柄」。預設 true。
- `useNativeDriver`: 為 true 時使用原生驅動。必填。

---

### `spring()`

```tsx
static spring(value, config): CompositeAnimation;
```

根據[阻尼諧振子模型](https://en.wikipedia.org/wiki/Harmonic_oscillator#Damped_harmonic_oscillator)的解析彈簧系統動畫化數值。該方法會追蹤速度狀態以在`toValue`更新時產生流暢運動效果，並可進行鏈式調用。

配置參數為物件，可包含以下選項：

注意：彈性/速度、張力/摩擦力或剛度/阻尼/質量這三組參數中只能定義其中一組，不可同時定義多組：

The friction/tension or bounciness/speed options match the spring model in [`Facebook Pop`](https://github.com/facebook/pop), [Rebound](https://github.com/facebookarchive/rebound), and [Origami](https://origami.design/).

- `friction`：控制「彈性」/過衝現象。預設值7。
- `tension`：控制動畫速度。預設值40。
- `speed`：控制動畫速度。預設值12。
- `bounciness`：控制彈性程度。預設值8。

指定剛度/阻尼/質量參數時，`Animated.spring`將採用基於[阻尼諧振子運動方程](https://en.wikipedia.org/wiki/Harmonic_oscillator#Damped_harmonic_oscillator)的解析彈簧模型。此行為更精確地模擬了彈簧動力學的物理特性，並高度還原iOS中CASpringAnimation的實現方式。

- `stiffness`：彈簧剛度係數。預設值100。
- `damping`：定義因摩擦力導致彈簧運動的衰減程度。預設值10。
- `mass`：附著在彈簧末端的物體質量。預設值1。

其他配置選項如下：

- `velocity`：附著在彈簧上物體的初始速度。預設0（物體處於靜止狀態）。
- `overshootClamping`：布林值，指示彈簧是否應鉗制而不反彈。預設false。
- `restDisplacementThreshold`：彈簧被視為靜止的位移閾值。預設0.001。
- `restSpeedThreshold`：彈簧被視為靜止的速度閾值（像素/秒）。預設0.001。
- `delay`：延遲啟動動畫的時間（毫秒）。預設0。
- `isInteraction`：此動畫是否在`InteractionManager`上創建「交互句柄」。預設true。
- `useNativeDriver`：為true時使用原生驅動。必須指定。

---

### `add()`

```tsx
static add(a: Animated, b: Animated): AnimatedAddition;
```

創建由兩個Animated值相加組成的新Animated值。

---

### `subtract()`

```tsx
static subtract(a: Animated, b: Animated): AnimatedSubtraction;
```

創建由第一個Animated值減去第二個Animated值組成的新Animated值。

---

### `divide()`

```tsx
static divide(a: Animated, b: Animated): AnimatedDivision;
```

創建由第一個Animated值除以第二個Animated值組成的新Animated值。

---

### `multiply()`

```tsx
static multiply(a: Animated, b: Animated): AnimatedMultiplication;
```

創建由兩個Animated值相乘組成的新Animated值。

---

### `modulo()`

```tsx
static modulo(a: Animated, modulus: number): AnimatedModulo;
```

創建提供Animated值的非負模數的新Animated值

---

### `diffClamp()`

```tsx
static diffClamp(a: Animated, min: number, max: number): AnimatedDiffClamp;
```

建立一個新的動畫值，其值被限制在兩個值之間。它會根據前後值的差異來調整，因此即使當前值遠離邊界，只要值開始接近邊界，就會開始變化（計算方式為 `value = clamp(value + diff, min, max)`）。

這在處理滾動事件時非常有用，例如在向上滾動時顯示導覽列，向下滾動時隱藏它。

---

### `delay()`

```tsx
static delay(time: number): CompositeAnimation;
```

在指定的延遲時間後開始動畫。

---

### `sequence()`

```tsx
static sequence(animations: CompositeAnimation[]): CompositeAnimation;
```

按順序啟動一組動畫，等待每個動畫完成後再開始下一個。如果當前運行的動畫被停止，後續動畫將不會啟動。

---

### `parallel()`

```tsx
static parallel(
  animations: CompositeAnimation[],
  config?: ParallelConfig
): CompositeAnimation;
```

同時啟動一組動畫。預設情況下，如果其中一個動畫被停止，所有動畫都會被停止。可以通過 `stopTogether` 標誌來覆蓋此行為。

---

### `stagger()`

```tsx
static stagger(
  time: number,
  animations: CompositeAnimation[]
): CompositeAnimation;
```

一組動畫可以並行運行（重疊），但會按順序以連續的延遲啟動。適合用於實現拖尾效果。

---

### `loop()`

```tsx
static loop(
  animation: CompositeAnimation[],
  config?: LoopAnimationConfig
): CompositeAnimation;
```

循環播放指定的動畫，每次到達結尾時會重置並從頭開始。如果子動畫設置為 `useNativeDriver: true`，則循環不會阻塞 JS 線程。此外，循環可能會阻止基於 `VirtualizedList` 的組件在動畫運行時渲染更多行。可以在子動畫配置中傳遞 `isInteraction: false` 來解決此問題。

配置是一個對象，可能包含以下選項：

- `iterations`: 動畫循環的次數。默認為 `-1`（無限循環）。

---

### `event()`

```tsx
static event(
  argMapping: Mapping[],
  config?: EventConfig
): (...args: any[]) => void;
```

接收一個映射數組，並從每個參數中提取相應的值，然後在映射的輸出上調用 `setValue`。例如：

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

配置是一個對象，可能包含以下選項：

- `listener`: 可選的異步監聽器。
- `useNativeDriver`: 為 true 時使用原生驅動。必需。

---

### `forkEvent()`

```jsx
static forkEvent(event: AnimatedEvent, listener: Function): AnimatedEvent;
```

這是一個高階的命令式 API，用於監聽通過 props 傳遞的動畫事件。它允許為現有的 `AnimatedEvent` 添加一個新的 JavaScript 監聽器。如果 `animatedEvent` 是一個 JavaScript 監聽器，它會將兩個監聽器合併為一個；如果 `animatedEvent` 為 null/undefined，則會直接分配 JavaScript 監聽器。盡可能直接使用值。

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

Animations are started by calling start() on your animation. start() takes a completion callback that will be called when the animation is done or when the animation is done because stop() was called on it before it could finish.

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

停止所有正在執行的動畫。

---

### `reset()`

```tsx
static reset();
```

停止所有正在執行的動畫並將數值重置為原始值。

## 屬性

### `Value`

驅動動畫的標準數值類別。通常在類別元件中初始化為 `useAnimatedValue(0);` 或 `new Animated.Value(0);`。

You can read more about `Animated.Value` API on the separate [page](animatedvalue).

---

### `ValueXY`

用於驅動 2D 動畫（例如平移手勢）的 2D 數值類別。

You can read more about `Animated.ValueXY` API on the separate [page](animatedvaluexy).

---

### `Interpolation`

導出以在流程中使用 Interpolation 類型。

---

### `Node`

導出以便於類型檢查。所有動畫數值均派生自此類別。

---

### `createAnimatedComponent`

使任何 React 元件可動畫化。用於創建 `Animated.View` 等元件。

---

### `attachNativeEvent`

將動畫數值附加到視圖事件的命令式 API。如果可能，建議使用 `Animated.event` 並設置 `useNativeDriver: true`。