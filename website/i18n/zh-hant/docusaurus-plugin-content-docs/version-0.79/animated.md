---
id: animated
title: Animated
---

The `Animated` library is designed to make animations fluid, powerful, and painless to build and maintain. `Animated` focuses on declarative relationships between inputs and outputs, configurable transforms in between, and `start`/`stop` methods to control time-based animation execution.

建立動畫的核心流程是：創建一個 `Animated.Value`，將其綁定至動畫元件的多個樣式屬性，再透過 `Animated.timing()` 驅動動畫更新。

> 請勿直接修改動畫值。可使用 [`useRef` Hook](https://reactjs.org/docs/hooks-reference.html#useref) 返回可變 ref 物件，該物件的 `current` 屬性會作為初始參數並在元件生命週期內持續存在。

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

參閱 [動畫指南](animations#animated-api) 以查看更多 `Animated` 實際應用範例。

## 概覽

There are two value types you can use with `Animated`:

- [`Animated.Value()`](animated#value) 用於單一數值
- [`Animated.ValueXY()`](animated#valuexy) 用於向量

`Animated.Value` 可綁定至樣式屬性或其他參數，並支援插值計算。單一 `Animated.Value` 可驅動任意數量的屬性。

### 動畫配置

`Animated` 提供三種動畫類型，每種都具備特定的動畫曲線，控制數值如何從初始值變化至最終值：

- [`Animated.decay()`](animated#decay) 以初始速度開始並逐漸減速至停止
- [`Animated.spring()`](animated#spring) 提供基礎彈簧物理模型
- [`Animated.timing()`](animated#timing) 使用[緩動函式](easing)隨時間變化數值

多數情況下會使用 `timing()`。預設採用對稱的 easeInOut 曲線，模擬物體加速至全速後再減速停止的過程。

### 動畫操作

透過呼叫動畫的 `start()` 方法啟動動畫。`start()` 接受一個完成回調函式，當動畫結束時會被呼叫。若動畫正常完成，回調將收到 `{finished: true}`；若因中途呼叫 `stop()` 而中止（例如被手勢或其他動畫打斷），則會收到 `{finished: false}`。

```tsx
Animated.timing({}).start(({finished}) => {
  /* completion callback */
});
```

### 使用原生驅動

啟用原生驅動後，系統會在動畫開始前將所有參數傳送至原生端，讓 UI 執行緒直接處理動畫，無需每幀都透過橋接器溝通。動畫啟動後，即使 JavaScript 執行緒被阻塞也不影響動畫運行。

在動畫配置中指定 `useNativeDriver: true` 即可啟用原生驅動。詳見[動畫指南](animations#using-the-native-driver)。

### 可動畫化元件

僅有特殊處理的可動畫化元件能執行動畫。這些元件會將動畫值綁定至屬性，並針對性地進行原生更新，避免每幀都觸發 React 渲染與協調過程的效能消耗，同時會在卸載時自動清理資源。

- [`createAnimatedComponent()`](animated#createanimatedcomponent) 可將普通元件轉換為可動畫化元件。

`Animated` 預先導出以下透過該封裝處理的動畫元件：

- `Animated.Image`
- `Animated.ScrollView`
- `Animated.Text`
- `Animated.View`
- `Animated.FlatList`
- `Animated.SectionList`

### 動畫組合

動畫也可以透過組合函數以複雜的方式結合：

- [`Animated.delay()`](animated#delay) 在指定延遲後開始動畫。
- [`Animated.parallel()`](animated#parallel) 同時啟動多個動畫。
- [`Animated.sequence()`](animated#sequence) 按順序啟動動畫，等待前一個完成後才開始下一個。
- [`Animated.stagger()`](animated#stagger) 按順序且並行啟動動畫，但會連續延遲。

動畫也可以透過將一個動畫的 `toValue` 設為另一個 `Animated.Value` 來串聯。請參閱動畫指南中的[追蹤動態值](animations#tracking-dynamic-values)。

預設情況下，如果一個動畫被停止或中斷，則群組中的所有其他動畫也會停止。

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

更多關於插值的資訊，請參閱[動畫](animations#interpolation)指南。

### 處理手勢和其他事件

手勢（如平移或滾動）和其他事件可以使用 `Animated.event()` 直接映射到動畫值。這是透過結構化映射語法完成的，以便可以從複雜的事件對象中提取值。第一層是一個陣列，允許跨多個參數進行映射，該陣列包含嵌套對象。

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

當給定的值是 ValueXY 而不是 Value 時，每個配置選項可以是 `{x: ..., y: ...}` 形式的向量，而不是標量。

### `decay()`

```tsx
static decay(value, config): CompositeAnimation;
```

根據衰減係數，將值從初始速度動畫化為零。

配置是一個對象，可能包含以下選項：

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

配置是一個對象，可能包含以下選項：

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

根據[阻尼諧振子](https://en.wikipedia.org/wiki/Harmonic_oscillator#Damped_harmonic_oscillator)的解析彈簧模型動畫化數值。透過追蹤速度狀態，在`toValue`更新時創建流暢運動效果，並可串聯使用。

配置為一個物件，可包含以下選項：

注意：只能選擇定義以下其中一組參數，不可同時定義多組：

The friction/tension or bounciness/speed options match the spring model in [`Facebook Pop`](https://github.com/facebook/pop), [Rebound](https://github.com/facebookarchive/rebound), and [Origami](https://origami.design/).

- `friction`：控制「彈性」/過衝。預設值7。
- `tension`：控制速度。預設值40。
- `speed`：控制動畫速度。預設值12。
- `bounciness`：控制彈性。預設值8。

指定stiffness/damping/mass參數會使`Animated.spring`採用基於[阻尼諧振子運動方程](https://en.wikipedia.org/wiki/Harmonic_oscillator#Damped_harmonic_oscillator)的解析彈簧模型。此行為更精確且符合彈簧動力學的物理原理，並高度還原iOS中CASpringAnimation的實現方式。

- `stiffness`：彈簧剛度係數。預設值100。
- `damping`：定義因摩擦力導致彈簧運動的衰減程度。預設值10。
- `mass`：附加在彈簧末端物體的質量。預設值1。

其他配置選項如下：

- `velocity`：附加在彈簧上物體的初始速度。預設0（物體靜止）。
- `overshootClamping`：布林值，指示彈簧是否應箝位而不反彈。預設false。
- `restDisplacementThreshold`：彈簧被視為靜止的位移閾值。預設0.001。
- `restSpeedThreshold`：彈簧被視為靜止的速度閾值（像素/秒）。預設0.001。
- `delay`：延遲後開始動畫（毫秒）。預設0。
- `isInteraction`：此動畫是否在`InteractionManager`上建立「互動句柄」。預設true。
- `useNativeDriver`：為true時使用原生驅動。必填。

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

創建提供Animated值的（非負）模數新Animated值。

---

### `diffClamp()`

```tsx
static diffClamp(a: Animated, min: number, max: number): AnimatedDiffClamp;
```

建立一個新的動畫值，其值限制在兩個數值之間。它會根據前後值的差異來調整，因此即使當前值遠離邊界，當值開始接近時仍會觸發變化（計算方式為 `value = clamp(value + diff, min, max)`）。

這在處理滾動事件時特別有用，例如向上滾動時顯示導覽列，向下滾動時隱藏。

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

按順序執行一組動畫，等待前一個動畫完成後才開始下一個。若當前運行的動畫被停止，後續動畫將不會啟動。

---

### `parallel()`

```tsx
static parallel(
  animations: CompositeAnimation[],
  config?: ParallelConfig
): CompositeAnimation;
```

同時啟動一組動畫。預設情況下，若其中一個動畫被停止，所有動畫都會停止。可透過 `stopTogether` 標誌覆寫此行為。

---

### `stagger()`

```tsx
static stagger(
  time: number,
  animations: CompositeAnimation[]
): CompositeAnimation;
```

讓一組動畫以序列延遲的方式啟動，但允許並行（重疊）執行。適合用於實現拖尾效果。

---

### `loop()`

```tsx
static loop(
  animation: CompositeAnimation[],
  config?: LoopAnimationConfig
): CompositeAnimation;
```

循環播放指定動畫，每次到達結尾時重置並重新開始。若子動畫設為 `useNativeDriver: true`，循環過程不會阻塞 JS 線程。注意：循環動畫可能導致基於 `VirtualizedList` 的元件無法渲染新行，可透過在子動畫配置中設定 `isInteraction: false` 解決。

配置為物件，可包含以下選項：

- `iterations`: 動畫循環次數。預設為 `-1`（無限循環）。

---

### `event()`

```tsx
static event(
  argMapping: Mapping[],
  config?: EventConfig
): (...args: any[]) => void;
```

接收映射陣列，從每個參數提取對應值後，對映射輸出呼叫 `setValue`。例如：

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

配置為物件，可包含以下選項：

- `listener`: 可選的非同步監聽器。
- `useNativeDriver`: 設為 true 時使用原生驅動。必填。

---

### `forkEvent()`

```jsx
static forkEvent(event: AnimatedEvent, listener: Function): AnimatedEvent;
```

Advanced imperative API for snooping on animated events that are passed in through props. It permits to add a new javascript listener to an existing `AnimatedEvent`. If `animatedEvent` is a javascript listener, it will merge the 2 listeners into a single one, and if `animatedEvent` is null/undefined, it will assign the javascript listener directly. Use values directly where possible.

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

透過呼叫動畫的 start() 方法啟動動畫。start() 接受完成回調函數，當動畫正常結束或因提前呼叫 stop() 而中止時會觸發該回調。

**參數：**

| Name     | Type                                    | Required | Description                                                                                                                                                     |
| -------- | --------------------------------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| callback | `(result: {finished: boolean}) => void` | No       | Function that will be called after the animation finished running normally or when the animation is done because stop() was called on it before it could finish |

含回調的啟動範例：

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

停止所有正在執行的動畫並將數值重置為原始狀態。

## 屬性

### `Value`

驅動動畫的標準數值類別。通常透過 `useAnimatedValue(0);` 或在類別元件中使用 `new Animated.Value(0);` 初始化。

You can read more about `Animated.Value` API on the separate [page](animatedvalue).

---

### `ValueXY`

用於驅動二維動畫（例如平移手勢）的 2D 數值類別。

You can read more about `Animated.ValueXY` API on the separate [page](animatedvaluexy).

---

### `Interpolation`

導出以在流程中使用插值類型。

---

### `Node`

導出以便於類型檢查。所有動畫數值均派生自此類別。

---

### `createAnimatedComponent`

使任何 React 元件可動畫化。用於建立 `Animated.View` 等元件。

---

### `attachNativeEvent`

將動畫數值附加到視圖事件的命令式 API。如果可能，建議優先使用 `Animated.event` 並設定 `useNativeDriver: true`。