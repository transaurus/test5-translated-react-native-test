---
id: animations
title: Animations
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

動畫對於創造出色的用戶體驗至關重要。靜止的物體開始移動時必須克服慣性，而運動中的物體具有動量，很少會立即停止。動畫能讓您在介面中傳達符合物理規律的運動效果。

React Native 提供兩套互補的動畫系統：[`Animated`](animations#animated-api) 用於對特定值進行細粒度互動控制，[`LayoutAnimation`](animations#layoutanimation-api) 則用於全局佈局變換的動畫處理。

## `Animated` API

The [`Animated`](animated) API is designed to concisely express a wide variety of interesting animation and interaction patterns in a very performant way. `Animated` focuses on declarative relationships between inputs and outputs, with configurable transforms in between, and `start`/`stop` methods to control time-based animation execution.

`Animated` 導出六種可動畫化組件類型：`View`、`Text`、`Image`、`ScrollView`、`FlatList` 和 `SectionList`，您也可以使用 `Animated.createAnimatedComponent()` 創建自定義組件。

例如，一個在掛載時淡入的容器視圖可以這樣實現：

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer ext=js&supportedPlatforms=ios,android
import React, {useEffect} from 'react';
import {Animated, Text, View, useAnimatedValue} from 'react-native';

const FadeInView = props => {
  const fadeAnim = useAnimatedValue(0); // Initial value for opacity: 0

  useEffect(() => {
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 10000,
      useNativeDriver: true,
    }).start();
  }, [fadeAnim]);

  return (
    <Animated.View // Special animatable View
      style={{
        ...props.style,
        opacity: fadeAnim, // Bind opacity to animated value
      }}>
      {props.children}
    </Animated.View>
  );
};

// You can then use your `FadeInView` in place of a `View` in your components:
export default () => {
  return (
    <View
      style={{
        flex: 1,
        alignItems: 'center',
        justifyContent: 'center',
      }}>
      <FadeInView
        style={{
          width: 250,
          height: 50,
          backgroundColor: 'powderblue',
        }}>
        <Text style={{fontSize: 28, textAlign: 'center', margin: 10}}>
          Fading in
        </Text>
      </FadeInView>
    </View>
  );
};
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer ext=tsx
import React, {useEffect} from 'react';
import {Animated, Text, View, useAnimatedValue} from 'react-native';
import type {PropsWithChildren} from 'react';
import type {ViewStyle} from 'react-native';

type FadeInViewProps = PropsWithChildren<{style: ViewStyle}>;

const FadeInView: React.FC<FadeInViewProps> = props => {
  const fadeAnim = useAnimatedValue(0); // Initial value for opacity: 0

  useEffect(() => {
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 10000,
      useNativeDriver: true,
    }).start();
  }, [fadeAnim]);

  return (
    <Animated.View // Special animatable View
      style={{
        ...props.style,
        opacity: fadeAnim, // Bind opacity to animated value
      }}>
      {props.children}
    </Animated.View>
  );
};

// You can then use your `FadeInView` in place of a `View` in your components:
export default () => {
  return (
    <View
      style={{
        flex: 1,
        alignItems: 'center',
        justifyContent: 'center',
      }}>
      <FadeInView
        style={{
          width: 250,
          height: 50,
          backgroundColor: 'powderblue',
        }}>
        <Text style={{fontSize: 28, textAlign: 'center', margin: 10}}>
          Fading in
        </Text>
      </FadeInView>
    </View>
  );
};
```

</TabItem>
</Tabs>

Let's break down what's happening here. In the `FadeInView` constructor, a new `Animated.Value` called `fadeAnim` is initialized as part of `state`. The opacity property on the `View` is mapped to this animated value. Behind the scenes, the numeric value is extracted and used to set opacity.

組件掛載時，透明度設為 0。接著對 `fadeAnim` 啟動緩動動畫，該值會逐幀過渡到最終值 1，同時更新所有關聯屬性（本例中僅透明度）。

這種優化實現比調用 `setState` 重新渲染更高效。由於整個配置是聲明式的，未來還能實現更高階的優化，例如序列化配置並在高優先級線程執行動畫。

### 配置動畫

動畫具有高度可配置性。自定義或預設的緩動函數、延遲、持續時間、衰減因子、彈簧係數等參數，均可根據動畫類型進行調整。

`Animated` 提供多種動畫類型，最常用的是 [`Animated.timing()`](animated#timing)。它支持使用預設緩動函數或自定義函數隨時間變化數值。緩動函數通常用於表現物體逐漸加速或減速的效果。

默認情況下，`timing` 會採用 easeInOut 曲線，表現為先漸加速至全速，結束時漸減速至停止。您可通過 `easing` 參數指定其他緩動函數，也支持設置自定義 `duration` 或動畫開始前的 `delay`。

例如要創建一個 2 秒長的動畫，讓物體在到達最終位置前先輕微後退：

```tsx
Animated.timing(this.state.xPosition, {
  toValue: 100,
  easing: Easing.back(),
  duration: 2000,
  useNativeDriver: true,
}).start();
```

Take a look at the [Configuring animations](animated#configuring-animations) section of the `Animated` API reference to learn more about all the config parameters supported by the built-in animations.

### 組合動畫

動畫可以組合並按順序或並行播放。順序動畫可在前一個動畫結束後立即播放，或延遲指定時間啟動。`Animated` API 提供 `sequence()` 和 `delay()` 等方法，這些方法接收動畫數組並自動按需調用 `start()`/`stop()`。

例如下面這個動畫會先滑行停止，然後同時彈回並旋轉：

```tsx
Animated.sequence([
  // decay, then spring to start and twirl
  Animated.decay(position, {
    // coast to a stop
    velocity: {x: gestureState.vx, y: gestureState.vy}, // velocity from gesture release
    deceleration: 0.997,
    useNativeDriver: true,
  }),
  Animated.parallel([
    // after decay, in parallel:
    Animated.spring(position, {
      toValue: {x: 0, y: 0}, // return to start
      useNativeDriver: true,
    }),
    Animated.timing(twirl, {
      // and twirl
      toValue: 360,
      useNativeDriver: true,
    }),
  ]),
]).start(); // start the sequence group
```

如果一個動畫被停止或中斷，那麼群組中的所有其他動畫也會被停止。`Animated.parallel` 提供了一個 `stopTogether` 選項，可設為 `false` 來禁用此行為。

You can find a full list of composition methods in the [Composing animations](animated#composing-animations) section of the `Animated` API reference.

### 合併動畫數值

你可以透過加法、乘法、除法或模運算來[合併兩個動畫數值](animated#combining-animated-values)，從而創建新的動畫數值。

在某些情況下，一個動畫數值需要反轉另一個動畫數值以進行計算。例如反轉縮放比例（2倍 --> 0.5倍）：

```tsx
const a = new Animated.Value(1);
const b = Animated.divide(1, a);

Animated.spring(a, {
  toValue: 2,
  useNativeDriver: true,
}).start();
```

### 插值

每個屬性都可以先經過插值處理。插值將輸入範圍映射到輸出範圍，通常使用線性插值，但也支援緩動函數。預設情況下，它會外推超出給定範圍的曲線，但你也可以讓它箝制輸出值。

將 0-1 範圍轉換為 0-100 範圍的基本映射如下：

```tsx
value.interpolate({
  inputRange: [0, 1],
  outputRange: [0, 100],
});
```

舉例來說，你可能希望將 `Animated.Value` 視為從 0 到 1，但將位置從 150px 動畫到 0px，同時將透明度從 0 動畫到 1。可以透過修改上述範例中的 `style` 來實現：

```tsx
  style={{
    opacity: this.state.fadeAnim, // Binds directly
    transform: [{
      translateY: this.state.fadeAnim.interpolate({
        inputRange: [0, 1],
        outputRange: [150, 0]  // 0 : 150, 0.5 : 75, 1 : 0
      }),
    }],
  }}
```

[`interpolate()`](animated#interpolate) 還支援多個範圍區段，這對於定義死區和其他實用技巧非常方便。例如，要在 -300 處建立否定關係，在 -100 處降至 0，在 0 處回升至 1，在 100 處再次降至 0，之後的所有值保持為 0 的死區，可以這樣做：

```tsx
value.interpolate({
  inputRange: [-300, -100, 0, 100, 101],
  outputRange: [300, 0, 1, 0, 0],
});
```

其映射關係如下：

```
Input | Output
------|-------
  -400|    450
  -300|    300
  -200|    150
  -100|      0
   -50|    0.5
     0|      1
    50|    0.5
   100|      0
   101|      0
   200|      0
```

`interpolate()` 還支援映射到字串，允許你動畫顏色以及帶單位的數值。例如，如果要動畫旋轉，可以這樣做：

```tsx
value.interpolate({
  inputRange: [0, 360],
  outputRange: ['0deg', '360deg'],
});
```

`interpolate()` 還支援任意緩動函數，其中許多已在 [`Easing`](easing) 模組中實現。`interpolate()` 還可配置外推 `outputRange` 的行為。你可以通過設置 `extrapolate`、`extrapolateLeft` 或 `extrapolateRight` 選項來配置外推。預設值為 `extend`，但你可以使用 `clamp` 來防止輸出值超出 `outputRange`。

### 追蹤動態數值

Animated values can also track other values by setting the `toValue` of an animation to another animated value instead of a plain number. For example, a "Chat Heads" animation like the one used by Messenger on Android could be implemented with a `spring()` pinned on another animated value, or with `timing()` and a `duration` of 0 for rigid tracking. They can also be composed with interpolations:

```tsx
Animated.spring(follower, {toValue: leader}).start();
Animated.timing(opacity, {
  toValue: pan.x.interpolate({
    inputRange: [0, 300],
    outputRange: [1, 0],
    useNativeDriver: true,
  }),
}).start();
```

The `leader` and `follower` animated values would be implemented using `Animated.ValueXY()`. `ValueXY` is a handy way to deal with 2D interactions, such as panning or dragging. It is a basic wrapper that contains two `Animated.Value` instances and some helper functions that call through to them, making `ValueXY` a drop-in replacement for `Value` in many cases. It allows us to track both x and y values in the example above.

### 追蹤手勢

手勢（如平移或滾動）和其他事件可以使用 [`Animated.event`](animated#event) 直接映射到動畫數值。這是通過結構化映射語法實現的，以便可以從複雜的事件對象中提取數值。第一層是一個數組，允許跨多個參數進行映射，該數組包含嵌套對象。

舉例來說，若要將水平滾動手勢的 `event.nativeEvent.contentOffset.x` 映射到 `scrollX`（一個 `Animated.Value`），您可以這樣操作：

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

The following example implements a horizontal scrolling carousel where the scroll position indicators are animated using the `Animated.event` used in the `ScrollView`

#### 使用 Animated Event 的 ScrollView 範例

```SnackPlayer name=Animated&supportedPlatforms=ios,android
import React from 'react';
import {
  SafeAreaView,
  ScrollView,
  Text,
  StyleSheet,
  View,
  ImageBackground,
  Animated,
  useWindowDimensions,
  useAnimatedValue,
} from 'react-native';

const images = new Array(6).fill(
  'https://images.unsplash.com/photo-1556740749-887f6717d7e4',
);

const App = () => {
  const scrollX = useAnimatedValue(0);

  const {width: windowWidth} = useWindowDimensions();

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.scrollContainer}>
        <ScrollView
          horizontal={true}
          pagingEnabled
          showsHorizontalScrollIndicator={false}
          onScroll={Animated.event([
            {
              nativeEvent: {
                contentOffset: {
                  x: scrollX,
                },
              },
            },
          ])}
          scrollEventThrottle={1}>
          {images.map((image, imageIndex) => {
            return (
              <View style={{width: windowWidth, height: 250}} key={imageIndex}>
                <ImageBackground source={{uri: image}} style={styles.card}>
                  <View style={styles.textContainer}>
                    <Text style={styles.infoText}>
                      {'Image - ' + imageIndex}
                    </Text>
                  </View>
                </ImageBackground>
              </View>
            );
          })}
        </ScrollView>
        <View style={styles.indicatorContainer}>
          {images.map((image, imageIndex) => {
            const width = scrollX.interpolate({
              inputRange: [
                windowWidth * (imageIndex - 1),
                windowWidth * imageIndex,
                windowWidth * (imageIndex + 1),
              ],
              outputRange: [8, 16, 8],
              extrapolate: 'clamp',
            });
            return (
              <Animated.View
                key={imageIndex}
                style={[styles.normalDot, {width}]}
              />
            );
          })}
        </View>
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
  scrollContainer: {
    height: 300,
    alignItems: 'center',
    justifyContent: 'center',
  },
  card: {
    flex: 1,
    marginVertical: 4,
    marginHorizontal: 16,
    borderRadius: 5,
    overflow: 'hidden',
    alignItems: 'center',
    justifyContent: 'center',
  },
  textContainer: {
    backgroundColor: 'rgba(0,0,0, 0.7)',
    paddingHorizontal: 24,
    paddingVertical: 8,
    borderRadius: 5,
  },
  infoText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  normalDot: {
    height: 8,
    width: 8,
    borderRadius: 4,
    backgroundColor: 'silver',
    marginHorizontal: 4,
  },
  indicatorContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
  },
});

export default App;
```

When using `PanResponder`, you could use the following code to extract the x and y positions from `gestureState.dx` and `gestureState.dy`. We use a `null` in the first position of the array, as we are only interested in the second argument passed to the `PanResponder` handler, which is the `gestureState`.

```tsx
onPanResponderMove={Animated.event(
  [null, // ignore the native event
  // extract dx and dy from gestureState
  // like 'pan.x = gestureState.dx, pan.y = gestureState.dy'
  {dx: pan.x, dy: pan.y}
])}
```

#### 使用 Animated Event 的 PanResponder 範例

```SnackPlayer name=Animated
import React, {useRef} from 'react';
import {Animated, View, StyleSheet, PanResponder, Text} from 'react-native';

const App = () => {
  const pan = useRef(new Animated.ValueXY()).current;
  const panResponder = useRef(
    PanResponder.create({
      onMoveShouldSetPanResponder: () => true,
      onPanResponderMove: Animated.event([null, {dx: pan.x, dy: pan.y}]),
      onPanResponderRelease: () => {
        Animated.spring(pan, {
          toValue: {x: 0, y: 0},
          useNativeDriver: true,
        }).start();
      },
    }),
  ).current;

  return (
    <View style={styles.container}>
      <Text style={styles.titleText}>Drag & Release this box!</Text>
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

### 響應當前的動畫值

您可能會注意到，在動畫進行時沒有直接讀取當前值的方法。這是因為出於優化考量，數值可能僅在原生運行時中可知。若需根據當前值執行 JavaScript，有兩種解決方案：

- `spring.stopAnimation(callback)` 會停止動畫並以最終值觸發 `callback`，適用於手勢過渡場景。
- `spring.addListener(callback)` 會在動畫運行時異步觸發 `callback`，提供最新值。此方法適合觸發狀態變化（例如當用戶拖曳時將浮動控件吸附至新選項），因這類大型狀態變化對少數幀延遲的敏感度低於需保持 60 fps 的連續手勢（如平移操作）。

`Animated` 的設計目標是完全可序列化，使動畫能高性能運行且獨立於常規 JavaScript 事件循環。這會影響 API 設計，因此當某些操作比全同步系統更複雜時請理解其背景。可適度使用 `Animated.Value.addListener` 來突破部分限制，但需謹慎使用以避免潛在的性能影響。

### 使用原生驅動

The `Animated` API is designed to be serializable. By using the [native driver](/blog/2017/02/14/using-native-driver-for-animated), we send everything about the animation to native before starting the animation, allowing native code to perform the animation on the UI thread without having to go through the bridge on every frame. Once the animation has started, the JS thread can be blocked without affecting the animation.

Using the native driver for normal animations can be accomplished by setting `useNativeDriver: true` in animation config when starting it. Animations without a `useNativeDriver` property will default to false for legacy reasons, but emit a warning (and typechecking error in TypeScript).

```tsx
Animated.timing(this.state.animatedValue, {
  toValue: 1,
  duration: 500,
  useNativeDriver: true, // <-- Set this to true
}).start();
```

動畫值僅相容單一驅動模式，若對某值啟用原生驅動，請確保該值的所有動畫皆使用原生驅動。

原生驅動也可搭配 `Animated.event` 使用，這對於跟隨滾動位置的動畫特別重要——若未啟用原生驅動，由於 React Native 的異步特性，動畫會始終比手勢延遲一幀。

```tsx
<Animated.ScrollView // <-- Use the Animated ScrollView wrapper
  scrollEventThrottle={1} // <-- Use 1 here to make sure no events are ever missed
  onScroll={Animated.event(
    [
      {
        nativeEvent: {
          contentOffset: {y: this.state.animatedValue},
        },
      },
    ],
    {useNativeDriver: true}, // <-- Set this to true
  )}>
  {content}
</Animated.ScrollView>
```

您可透過運行 [RNTester 應用](https://github.com/facebook/react-native/blob/main/packages/rn-tester/)並查看「原生動畫範例」來觀察原生驅動的實際效果，或參考[源代碼](https://github.com/facebook/react-native/blob/master/packages/rn-tester/js/examples/NativeAnimation/NativeAnimationsExample.js)了解實現方式。

#### 注意事項

並非所有使用 `Animated` 的功能目前都受到原生驅動程式的支援。主要的限制在於，你只能動畫化非佈局屬性：例如 `transform` 和 `opacity` 會有效，但 Flexbox 和位置屬性則不會。當使用 `Animated.event` 時，它僅適用於直接事件，而不適用於冒泡事件。這意味著它不適用於 `PanResponder`，但適用於像 `ScrollView#onScroll` 這樣的功能。

當動畫運行時，它可能會阻止 `VirtualizedList` 元件渲染更多的行。如果你需要在用戶滾動列表時運行長時間或循環的動畫，你可以在動畫配置中使用 `isInteraction: false` 來避免這個問題。

### 注意事項

在使用如 `rotateY`、`rotateX` 等變形樣式時，確保變形樣式 `perspective` 已設置。目前某些動畫在沒有它的情況下可能無法在 Android 上渲染。以下是一個例子。

```tsx
<Animated.View
  style={{
    transform: [
      {scale: this.state.scale},
      {rotateY: this.state.rotateY},
      {perspective: 1000}, // without this line this Animation will not render on Android while working fine on iOS
    ],
  }}
/>
```

### 其他範例

RNTester 應用中有多個使用 `Animated` 的範例：

- [AnimatedGratuitousApp](https://github.com/facebook/react-native/tree/main/packages/rn-tester/js/examples/AnimatedGratuitousApp)
- [NativeAnimationsExample](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/examples/NativeAnimation/NativeAnimationsExample.js)

## `LayoutAnimation` API

`LayoutAnimation` 允許你全局配置 `create` 和 `update` 動畫，這些動畫將在下一個渲染/佈局週期中用於所有視圖。這對於進行 Flexbox 佈局更新非常有用，無需測量或計算特定屬性來直接動畫化它們，並且當佈局更改可能影響祖先時特別有用，例如一個「查看更多」的擴展，同時增加父級的大小並向下推動下面的行，否則需要元件之間的明確協調才能同步動畫化它們。

請注意，儘管 `LayoutAnimation` 非常強大且相當有用，但它提供的控制比 `Animated` 和其他動畫庫少得多，因此如果你無法讓 `LayoutAnimation` 實現你想要的效果，可能需要使用其他方法。

請注意，為了在 **Android** 上使其工作，你需要通過 `UIManager` 設置以下標誌：

```tsx
UIManager.setLayoutAnimationEnabledExperimental(true);
```

```SnackPlayer name=LayoutAnimations&supportedPlatforms=ios,android
import React from 'react';
import {
  NativeModules,
  LayoutAnimation,
  Text,
  TouchableOpacity,
  StyleSheet,
  View,
} from 'react-native';

const {UIManager} = NativeModules;

UIManager.setLayoutAnimationEnabledExperimental &&
  UIManager.setLayoutAnimationEnabledExperimental(true);

export default class App extends React.Component {
  state = {
    w: 100,
    h: 100,
  };

  _onPress = () => {
    // Animate the update
    LayoutAnimation.spring();
    this.setState({w: this.state.w + 15, h: this.state.h + 15});
  };

  render() {
    return (
      <View style={styles.container}>
        <View
          style={[styles.box, {width: this.state.w, height: this.state.h}]}
        />
        <TouchableOpacity onPress={this._onPress}>
          <View style={styles.button}>
            <Text style={styles.buttonText}>Press me!</Text>
          </View>
        </TouchableOpacity>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  box: {
    width: 200,
    height: 200,
    backgroundColor: 'red',
  },
  button: {
    backgroundColor: 'black',
    paddingHorizontal: 20,
    paddingVertical: 15,
    marginTop: 15,
  },
  buttonText: {
    color: '#fff',
    fontWeight: 'bold',
  },
});
```

這個例子使用了一個預設值，你可以根據需要自定義動畫，更多信息請參閱 [LayoutAnimation.js](https://github.com/facebook/react-native/blob/main/packages/react-native/Libraries/LayoutAnimation/LayoutAnimation.js)。

## 其他注意事項

### `requestAnimationFrame`

`requestAnimationFrame` 是一個你可能熟悉的瀏覽器 polyfill。它接受一個函數作為其唯一參數，並在下一次重繪之前調用該函數。它是所有基於 JavaScript 的動畫 API 的基本構建塊。通常，你不需要自己調用它——動畫 API 會為你管理幀更新。

### `setNativeProps`

如[在直接操作部分](direct-manipulation)中提到的，`setNativeProps` 允許我們直接修改原生支持的元件（實際由原生視圖支持的元件，與複合元件不同）的屬性，而無需 `setState` 和重新渲染元件層次結構。

我們可以在 Rebound 範例中使用它來更新縮放比例——如果我們正在更新的元件深度嵌套且未使用 `shouldComponentUpdate` 進行優化，這可能會有所幫助。

如果您發現動畫出現掉幀（低於每秒60幀）的情況，可以考慮使用 `setNativeProps` 或 `shouldComponentUpdate` 來優化。或者，您也可以選擇在UI線程而非JavaScript線程上執行動畫 [透過useNativeDriver選項](/blog/2017/02/14/using-native-driver-for-animated)。此外，建議將任何計算密集型的工作推遲到動畫完成後再進行，可使用 [InteractionManager](interactionmanager)。您可以使用應用內開發選單中的「FPS監測」工具來監控幀率。