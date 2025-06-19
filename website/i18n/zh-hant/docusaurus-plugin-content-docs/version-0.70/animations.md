---
id: animations
title: Animations
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

動畫對於創造出色的用戶體驗至關重要。靜止的物體開始移動時必須克服慣性，而運動中的物體具有動量，很少會立即停止。動畫能讓您在介面中傳達符合物理規律的運動效果。

React Native 提供兩套互補的動畫系統：[`Animated`](animations#animated-api) 用於對特定值進行細粒度互動控制，[`LayoutAnimation`](animations#layoutanimation-api) 則用於全局佈局變換的動畫處理。

## `Animated` API

The [`Animated`](animated) API is designed to concisely express a wide variety of interesting animation and interaction patterns in a very performant way. `Animated` focuses on declarative relationships between inputs and outputs, with configurable transforms in between, and `start`/`stop` methods to control time-based animation execution.

`Animated` 導出六種可動畫化的組件類型：`View`、`Text`、`Image`、`ScrollView`、`FlatList` 和 `SectionList`，您也可以使用 `Animated.createAnimatedComponent()` 創建自定義組件。

例如，一個在掛載時淡入的容器視圖可能如下所示：

```SnackPlayer
import React, { useRef, useEffect } from 'react';
import { Animated, Text, View } from 'react-native';

const FadeInView = (props) => {
  const fadeAnim = useRef(new Animated.Value(0)).current  // Initial value for opacity: 0

  useEffect(() => {
    Animated.timing(
      fadeAnim,
      {
        toValue: 1,
        duration: 10000,
      }
    ).start();
  }, [fadeAnim])

  return (
    <Animated.View                 // Special animatable View
      style={{
        ...props.style,
        opacity: fadeAnim,         // Bind opacity to animated value
      }}
    >
      {props.children}
    </Animated.View>
  );
}

// You can then use your `FadeInView` in place of a `View` in your components:
export default () => {
  return (
    <View style={{flex: 1, alignItems: 'center', justifyContent: 'center'}}>
      <FadeInView style={{width: 250, height: 50, backgroundColor: 'powderblue'}}>
        <Text style={{fontSize: 28, textAlign: 'center', margin: 10}}>Fading in</Text>
      </FadeInView>
    </View>
  )
}
```

Let's break down what's happening here. In the `FadeInView` constructor, a new `Animated.Value` called `fadeAnim` is initialized as part of `state`. The opacity property on the `View` is mapped to this animated value. Behind the scenes, the numeric value is extracted and used to set opacity.

組件掛載時，透明度設為 0。接著對 `fadeAnim` 啟動緩動動畫，該值會逐幀過渡到最終值 1，同時更新所有關聯屬性（本例中僅透明度）。

此過程經過優化，比調用 `setState` 重新渲染更高效。由於整個配置是聲明式的，未來還能實現將配置序列化並在高優先級線程運行動畫的進一步優化。

### 配置動畫

動畫具有高度可配置性。自定義或預設的緩動函數、延遲、持續時間、衰減係數、彈簧常數等參數，均可根據動畫類型調整。

`Animated` 提供多種動畫類型，最常用的是 [`Animated.timing()`](animated#timing)。它支持使用預設緩動函數（或自定義函數）隨時間變化數值。緩動函數通常用於表現物體的漸進加速與減速效果。

默認情況下，`timing` 採用 easeInOut 曲線，表現為先漸進加速至全速，結束時漸進減速至停止。您可通過 `easing` 參數指定其他緩動函數，也支持設置自定義 `duration` 或動畫開始前的 `delay`。

例如，若要創建一個時長 2 秒的動畫，讓物體在到達最終位置前先輕微後退：

```jsx
Animated.timing(this.state.xPosition, {
  toValue: 100,
  easing: Easing.back(),
  duration: 2000,
}).start();
```

Take a look at the [Configuring animations](animated#configuring-animations) section of the `Animated` API reference to learn more about all the config parameters supported by the built-in animations.

### 組合動畫

動畫可組合並以序列或並行方式播放。序列動畫可在前一動畫結束後立即播放，或延遲指定時間啟動。`Animated` API 提供 `sequence()` 和 `delay()` 等方法，這些方法接收動畫數組並自動按需調用 `start()`/`stop()`。

例如，以下動畫先滑行至停止，然後同時彈回並旋轉：

```jsx
Animated.sequence([
  // decay, then spring to start and twirl
  Animated.decay(position, {
    // coast to a stop
    velocity: {x: gestureState.vx, y: gestureState.vy}, // velocity from gesture release
    deceleration: 0.997,
  }),
  Animated.parallel([
    // after decay, in parallel:
    Animated.spring(position, {
      toValue: {x: 0, y: 0}, // return to start
    }),
    Animated.timing(twirl, {
      // and twirl
      toValue: 360,
    }),
  ]),
]).start(); // start the sequence group
```

若某個動畫被停止或中斷，組合中的所有其他動畫也會停止。`Animated.parallel` 提供 `stopTogether` 選項，設置為 `false` 可禁用此行為。

You can find a full list of composition methods in the [Composing animations](animated#composing-animations) section of the `Animated` API reference.

### 合併動畫數值

您可以透過加法、乘法、除法或模數運算來[合併兩個動畫數值](animated#combining-animated-values)，以產生新的動畫數值。

某些情況下，動畫數值需要反轉另一個動畫數值進行計算。例如反轉縮放比例（2倍 → 0.5倍）：

```jsx
const a = new Animated.Value(1);
const b = Animated.divide(1, a);

Animated.spring(a, {
  toValue: 2,
}).start();
```

### 插值運算

每個屬性都可以先經過插值處理。插值運算會將輸入範圍映射到輸出範圍，通常使用線性插值，但也支援緩動函數。預設情況下，它會將曲線外推至給定範圍之外，但您也可以設定讓輸出值固定於範圍內。

將 0-1 範圍轉換為 0-100 範圍的基本映射如下：

```jsx
value.interpolate({
  inputRange: [0, 1],
  outputRange: [0, 100],
});
```

舉例來說，您可能希望將 `Animated.Value` 視為從 0 到 1 變化，但實際動畫效果是將位置從 150px 移動到 0px，同時透明度從 0 變為 1。可以透過修改上述範例中的 `style` 來實現：

```jsx
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

[`interpolate()`](animated#interpolate) 還支援多段範圍映射，這對於定義死區和其他實用技巧非常方便。例如，若要實現一個在 -300 處反向變化、在 -100 處歸零、在 0 處回升至 1、在 100 處再次歸零，之後保持為 0 的死區效果，可以這樣設定：

```jsx
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

`interpolate()` 也支援映射至字串，讓您能動畫化顏色和帶單位的數值。例如，若要實現旋轉動畫，可以這樣做：

```jsx
value.interpolate({
  inputRange: [0, 360],
  outputRange: ['0deg', '360deg'],
});
```

`interpolate()` also supports arbitrary easing functions, many of which are already implemented in the [`Easing`](easing) module. `interpolate()` also has configurable behavior for extrapolating the `outputRange`. You can set the extrapolation by setting the `extrapolate`, `extrapolateLeft`, or `extrapolateRight` options. The default value is `extend` but you can use `clamp` to prevent the output value from exceeding `outputRange`.

### 追蹤動態數值

Animated values can also track other values by setting the `toValue` of an animation to another animated value instead of a plain number. For example, a "Chat Heads" animation like the one used by Messenger on Android could be implemented with a `spring()` pinned on another animated value, or with `timing()` and a `duration` of 0 for rigid tracking. They can also be composed with interpolations:

```jsx
Animated.spring(follower, {toValue: leader}).start();
Animated.timing(opacity, {
  toValue: pan.x.interpolate({
    inputRange: [0, 300],
    outputRange: [1, 0],
  }),
}).start();
```

The `leader` and `follower` animated values would be implemented using `Animated.ValueXY()`. `ValueXY` is a handy way to deal with 2D interactions, such as panning or dragging. It is a basic wrapper that contains two `Animated.Value` instances and some helper functions that call through to them, making `ValueXY` a drop-in replacement for `Value` in many cases. It allows us to track both x and y values in the example above.

### 追蹤手勢

透過 [`Animated.event`](animated#event)，手勢（如平移或滾動）和其他事件可直接映射至動畫數值。此功能採用結構化映射語法，能從複雜事件物件中提取數值。第一層是陣列，用於跨多個參數映射，該陣列包含嵌套物件。

例如，處理水平滾動手勢時，可透過以下方式將 `event.nativeEvent.contentOffset.x` 映射至 `scrollX`（一個 `Animated.Value`）：

```jsx
 onScroll={Animated.event(
   // scrollX = e.nativeEvent.contentOffset.x
   [{ nativeEvent: {
        contentOffset: {
          x: scrollX
        }
      }
    }]
 )}
```

The following example implements a horizontal scrolling carousel where the scroll position indicators are animated using the `Animated.event` used in the `ScrollView`

#### 使用 Animated Event 的 ScrollView 範例

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Animated&supportedPlatforms=ios,android
import React, { useRef } from "react";
import {
  SafeAreaView,
  ScrollView,
  Text,
  StyleSheet,
  View,
  ImageBackground,
  Animated,
  useWindowDimensions
} from "react-native";

const images = new Array(6).fill('https://images.unsplash.com/photo-1556740749-887f6717d7e4');

const App = () => {
  const scrollX = useRef(new Animated.Value(0)).current;

  const { width: windowWidth } = useWindowDimensions();

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
                  x: scrollX
                }
              }
            }
          ])}
          scrollEventThrottle={1}
        >
          {images.map((image, imageIndex) => {
            return (
              <View
                style={{ width: windowWidth, height: 250 }}
                key={imageIndex}
              >
                <ImageBackground source={{ uri: image }} style={styles.card}>
                  <View style={styles.textContainer}>
                    <Text style={styles.infoText}>
                      {"Image - " + imageIndex}
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
                windowWidth * (imageIndex + 1)
              ],
              outputRange: [8, 16, 8],
              extrapolate: "clamp"
            });
            return (
              <Animated.View
                key={imageIndex}
                style={[styles.normalDot, { width }]}
              />
            );
          })}
        </View>
      </View>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center"
  },
  scrollContainer: {
    height: 300,
    alignItems: "center",
    justifyContent: "center"
  },
  card: {
    flex: 1,
    marginVertical: 4,
    marginHorizontal: 16,
    borderRadius: 5,
    overflow: "hidden",
    alignItems: "center",
    justifyContent: "center"
  },
  textContainer: {
    backgroundColor: "rgba(0,0,0, 0.7)",
    paddingHorizontal: 24,
    paddingVertical: 8,
    borderRadius: 5
  },
  infoText: {
    color: "white",
    fontSize: 16,
    fontWeight: "bold"
  },
  normalDot: {
    height: 8,
    width: 8,
    borderRadius: 4,
    backgroundColor: "silver",
    marginHorizontal: 4
  },
  indicatorContainer: {
    flexDirection: "row",
    alignItems: "center",
    justifyContent: "center"
  }
});

export default App;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=Animated&supportedPlatforms=ios,android
import React, { Component } from "react";
import {
  SafeAreaView,
  ScrollView,
  Text,
  StyleSheet,
  View,
  ImageBackground,
  Animated,
  Dimensions
} from "react-native";

const images = new Array(6).fill('https://images.unsplash.com/photo-1556740749-887f6717d7e4');

const window = Dimensions.get("window");

export default class App extends Component {
  scrollX = new Animated.Value(0);

  state = {
    dimensions: {
      window
    }
  };

  onDimensionsChange = ({ window }) => {
    this.setState({ dimensions: { window } });
  };

  componentDidMount() {
    Dimensions.addEventListener("change", this.onDimensionsChange);
  }

  componentWillUnmount() {
    Dimensions.removeEventListener("change", this.onDimensionsChange);
  }

  render() {
    const windowWidth = this.state.dimensions.window.width;

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
                    x: this.scrollX
                  }
                }
              }
            ])}
            scrollEventThrottle={1}
          >
            {images.map((image, imageIndex) => {
              return (
                <View
                  style={{
                    width: windowWidth,
                    height: 250
                  }}
                  key={imageIndex}
                >
                  <ImageBackground source={{ uri: image }} style={styles.card}>
                    <View style={styles.textContainer}>
                      <Text style={styles.infoText}>
                        {"Image - " + imageIndex}
                      </Text>
                    </View>
                  </ImageBackground>
                </View>
              );
            })}
          </ScrollView>
          <View style={styles.indicatorContainer}>
            {images.map((image, imageIndex) => {
              const width = this.scrollX.interpolate({
                inputRange: [
                  windowWidth * (imageIndex - 1),
                  windowWidth * imageIndex,
                  windowWidth * (imageIndex + 1)
                ],
                outputRange: [8, 16, 8],
                extrapolate: "clamp"
              });
              return (
                <Animated.View
                  key={imageIndex}
                  style={[styles.normalDot, { width }]}
                />
              );
            })}
          </View>
        </View>
      </SafeAreaView>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center"
  },
  scrollContainer: {
    height: 300,
    alignItems: "center",
    justifyContent: "center"
  },
  card: {
    flex: 1,
    marginVertical: 4,
    marginHorizontal: 16,
    borderRadius: 5,
    overflow: "hidden",
    alignItems: "center",
    justifyContent: "center"
  },
  textContainer: {
    backgroundColor: "rgba(0,0,0, 0.7)",
    paddingHorizontal: 24,
    paddingVertical: 8,
    borderRadius: 5
  },
  infoText: {
    color: "white",
    fontSize: 16,
    fontWeight: "bold"
  },
  normalDot: {
    height: 8,
    width: 8,
    borderRadius: 4,
    backgroundColor: "silver",
    marginHorizontal: 4
  },
  indicatorContainer: {
    flexDirection: "row",
    alignItems: "center",
    justifyContent: "center"
  }
});
```

</TabItem>
</Tabs>

使用 `PanResponder` 時，可以透過以下程式碼從 `gestureState.dx` 和 `gestureState.dy` 提取 x 和 y 座標。我們在陣列的第一個位置使用 `null`，因為我們只對傳遞給 `PanResponder` 處理器的第二個參數 `gestureState` 感興趣。

```jsx
onPanResponderMove={Animated.event(
  [null, // ignore the native event
  // extract dx and dy from gestureState
  // like 'pan.x = gestureState.dx, pan.y = gestureState.dy'
  {dx: pan.x, dy: pan.y}
])}
```

#### 使用 Animated Event 的 PanResponder 範例

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Animated
import React, { useRef } from "react";
import { Animated, View, StyleSheet, PanResponder, Text } from "react-native";

const App = () => {
  const pan = useRef(new Animated.ValueXY()).current;
  const panResponder = useRef(
    PanResponder.create({
      onMoveShouldSetPanResponder: () => true,
      onPanResponderMove: Animated.event([
        null,
        { dx: pan.x, dy: pan.y }
      ]),
      onPanResponderRelease: () => {
        Animated.spring(pan, { toValue: { x: 0, y: 0 } }).start();
      }
    })
  ).current;

  return (
    <View style={styles.container}>
      <Text style={styles.titleText}>Drag & Release this box!</Text>
      <Animated.View
        style={{
          transform: [{ translateX: pan.x }, { translateY: pan.y }]
        }}
        {...panResponder.panHandlers}
      >
        <View style={styles.box} />
      </Animated.View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center"
  },
  titleText: {
    fontSize: 14,
    lineHeight: 24,
    fontWeight: "bold"
  },
  box: {
    height: 150,
    width: 150,
    backgroundColor: "blue",
    borderRadius: 5
  }
});

export default App;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=Animated
import React, { Component } from "react";
import { Animated, View, StyleSheet, PanResponder, Text } from "react-native";

export default class App extends Component {
  pan = new Animated.ValueXY();
  panResponder = PanResponder.create({
    onMoveShouldSetPanResponder: () => true,
    onPanResponderMove: Animated.event([
      null,
      { dx: this.pan.x, dy: this.pan.y }
    ]),
    onPanResponderRelease: () => {
      Animated.spring(this.pan, { toValue: { x: 0, y: 0 } }).start();
    }
  });

  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.titleText}>Drag & Release this box!</Text>
        <Animated.View
          style={{
            transform: [{ translateX: this.pan.x }, { translateY: this.pan.y }]
          }}
          {...this.panResponder.panHandlers}
        >
          <View style={styles.box} />
        </Animated.View>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center"
  },
  titleText: {
    fontSize: 14,
    lineHeight: 24,
    fontWeight: "bold"
  },
  box: {
    height: 150,
    width: 150,
    backgroundColor: "blue",
    borderRadius: 5
  }
});
```

</TabItem>
</Tabs>

### 響應當前的動畫值

你可能會注意到，在動畫進行時沒有明確的方法可以讀取當前值。這是因為由於優化，該值可能只在原生運行時中才能得知。如果你需要根據當前值執行 JavaScript，有兩種方法：

- `spring.stopAnimation(callback)` 會停止動畫並以最終值調用 `callback`。這在製作手勢過渡時非常有用。
- `spring.addListener(callback)` 會在動畫運行時異步調用 `callback`，提供最近的值。這對於觸發狀態變化很有用，例如當用戶將一個浮動元素拖近時將其吸附到新選項，因為這些較大的狀態變化對幾幀的延遲不太敏感，而像平移這樣的連續手勢則需要以 60 fps 運行。

`Animated` 的設計是完全可序列化的，以便動畫可以以高性能的方式運行，獨立於正常的 JavaScript 事件循環。這確實會影響 API，所以當你發現某些操作比完全同步的系統更棘手時，請記住這一點。可以查閱 `Animated.Value.addListener` 作為解決這些限制的一種方法，但請謹慎使用，因為它可能會在未來影響性能。

### 使用原生驅動

The `Animated` API is designed to be serializable. By using the [native driver](/blog/2017/02/14/using-native-driver-for-animated), we send everything about the animation to native before starting the animation, allowing native code to perform the animation on the UI thread without having to go through the bridge on every frame. Once the animation has started, the JS thread can be blocked without affecting the animation.

為普通動畫使用原生驅動很簡單。你可以在啟動動畫時在動畫配置中添加 `useNativeDriver: true`。

```jsx
Animated.timing(this.state.animatedValue, {
  toValue: 1,
  duration: 500,
  useNativeDriver: true, // <-- Add this
}).start();
```

動畫值僅與一個驅動程序兼容，因此如果你在啟動某個值的動畫時使用了原生驅動，請確保該值的所有動畫也都使用原生驅動。

原生驅動也可以與 `Animated.event` 一起使用。這對於跟隨滾動位置的動畫特別有用，因為如果沒有原生驅動，由於 React Native 的異步特性，動畫總是會比手勢慢一幀。

```jsx
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
    {useNativeDriver: true}, // <-- Add this
  )}>
  {content}
</Animated.ScrollView>
```

你可以通過運行 [RNTester 應用](https://github.com/facebook/react-native/blob/0.70-stable/packages/rn-tester/)，然後加載原生動畫範例來查看原生驅動的實際效果。你也可以查看[源代碼](https://github.com/facebook/react-native/blob/0.70-stable/packages/rn-tester/js/examples/NativeAnimation/NativeAnimationsExample.js)來了解這些範例是如何製作的。

#### 注意事項

並非所有你可以用 `Animated` 做的事情目前都受到原生驅動的支持。主要的限制是你只能動畫非佈局屬性：像 `transform` 和 `opacity` 這樣的屬性可以工作，但 Flexbox 和位置屬性則不行。當使用 `Animated.event` 時，它僅適用於直接事件而不適用於冒泡事件。這意味著它不適用於 `PanResponder`，但適用於像 `ScrollView#onScroll` 這樣的東西。

當動畫正在執行時，可能會阻止 `VirtualizedList` 元件渲染更多列。如果需要在用戶滾動列表時執行長時間或循環動畫，可以在動畫配置中使用 `isInteraction: false` 來避免此問題。

### 注意事項

使用如 `rotateY`、`rotateX` 等變形樣式時，請確保同時設置 `perspective` 變形樣式。目前某些動畫若缺少此設定，在 Android 上可能無法正常渲染。範例如下。

```jsx
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

RNTester 應用程式包含多種使用 `Animated` 的範例：

- [AnimatedGratuitousApp](https://github.com/facebook/react-native/tree/0.70-stable/packages/rn-tester/js/examples/AnimatedGratuitousApp)
- [NativeAnimationsExample](https://github.com/facebook/react-native/blob/0.70-stable/packages/rn-tester/js/examples/NativeAnimation/NativeAnimationsExample.js)

## `LayoutAnimation` API

`LayoutAnimation` allows you to globally configure `create` and `update` animations that will be used for all views in the next render/layout cycle. This is useful for doing Flexbox layout updates without bothering to measure or calculate specific properties in order to animate them directly, and is especially useful when layout changes may affect ancestors, for example a "see more" expansion that also increases the size of the parent and pushes down the row below which would otherwise require explicit coordination between the components in order to animate them all in sync.

Note that although `LayoutAnimation` is very powerful and can be quite useful, it provides much less control than `Animated` and other animation libraries, so you may need to use another approach if you can't get `LayoutAnimation` to do what you want.

注意：在 **Android** 上需透過 `UIManager` 設定以下標記才能生效：

```jsx
UIManager.setLayoutAnimationEnabledExperimental &&
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

const { UIManager } = NativeModules;

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
    this.setState({w: this.state.w + 15, h: this.state.h + 15})
  }

  render() {
    return (
      <View style={styles.container}>
        <View style={[styles.box, {width: this.state.w, height: this.state.h}]} />
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

此範例使用預設值，您可依需求自訂動畫，詳見 [LayoutAnimation.js](https://github.com/facebook/react-native/blob/0.70-stable/Libraries/LayoutAnimation/LayoutAnimation.js)。

## 補充說明

### `requestAnimationFrame`

`requestAnimationFrame` 是來自瀏覽器的 polyfill，接受函數作為唯一參數，並在下一次重繪前呼叫該函數。它是所有基於 JavaScript 動畫 API 的基礎建構塊。通常不需直接呼叫此方法，動畫 API 會自動管理影格更新。

### `setNativeProps`

如[直接操作章節](direct-manipulation)所述，`setNativeProps` 允許直接修改原生元件（實際由原生視圖支持的元件，非複合元件）的屬性，而無需透過 `setState` 重新渲染元件樹。

我們可在 Rebound 範例中使用此方法更新縮放比例——這對於需更新的深層嵌套元件或未經 `shouldComponentUpdate` 優化的情況特別有用。

若發現動畫出現掉幀（低於每秒 60 幀），可嘗試使用 `setNativeProps` 或 `shouldComponentUpdate` 進行優化，或改用 [useNativeDriver 選項](/blog/2017/02/14/using-native-driver-for-animated) 在 UI 執行緒而非 JavaScript 執行緒執行動畫。也可透過 [InteractionManager](interactionmanager) 延後計算密集型任務至動畫完成後執行。使用開發者選單中的「FPS 監測」工具可監控幀率。