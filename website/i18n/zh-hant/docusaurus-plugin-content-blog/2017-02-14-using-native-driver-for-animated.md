---
title: Using Native Driver for Animated
author: Janic Duplessis
authorTitle: Software Engineer at App & Flow
authorURL: 'https://twitter.com/janicduplessis'
authorImageURL: 'https://secure.gravatar.com/avatar/8d6b6c0f5b228b0a8566a69de448b9dd?s=128'
authorTwitter: janicduplessis
tags: [engineering]
---

過去一年來，我們持續改進使用 Animated 函式庫的動畫效能。動畫對於打造流暢的使用者體驗至關重要，但實作難度也較高。我們希望讓開發者能輕鬆創建高效能動畫，無需擔心程式碼導致的卡頓問題。

## 這是什麼？

Animated API 的設計核心在於「可序列化」的關鍵約束。這意味著我們能在動畫開始前就將所有參數傳送至原生端，讓原生程式碼能在 UI 執行緒直接處理動畫，無需每幀都透過橋接層溝通。此機制特別實用，因為即使動畫開始後 JS 執行緒被阻塞，動畫仍能流暢運行。實務上這種情況很常見，因為使用者程式碼運行於 JS 執行緒，且 React 渲染也可能長時間佔用 JS 資源。

## 歷史背景...

此專案始於一年前，當時 Expo 為 Android 版 li.st 應用進行開發。[Krzysztof Magiera](https://twitter.com/kzzzf) 受託建置 Android 端的初始實現，最終成果優異，使 li.st 成為首個採用 Animated 原生驅動動畫的應用。數月後，[Brandon Withrow](https://github.com/buba447) 完成了 iOS 端的初始實現。此後 [Ryan Gomba](https://twitter.com/ryangomba) 與我共同補強功能缺口，例如支援 `Animated.event` 及修復生產環境中的錯誤。這真正是社群協力的成果，我要感謝所有參與者以及贊助大部分開發工作的 Expo。該技術現已用於 React Native 的 `Touchable` 元件，以及新發布的 [React Navigation](https://github.com/react-community/react-navigation) 函式庫的轉場動畫。

## 運作原理？

First, let's check out how animations currently work using Animated with the JS driver. When using Animated, you declare a graph of nodes that represent the animations that you want to perform, and then use a driver to update an Animated value using a predefined curve. You may also update an Animated value by connecting it to an event of a `View` using `Animated.event`.

![](/blog/assets/animated-diagram.png)

以下是動畫運作的步驟分解與執行位置：

- JS: The animation driver uses `requestAnimationFrame` to execute on every frame and update the value it drives using the new value it calculates based on the animation curve.
- JS: Intermediate values are calculated and passed to a props node that is attached to a `View`.
- JS: The `View` is updated using `setNativeProps`.
- JS to Native bridge.
- Native: The `UIView` or `android.View` is updated.

如您所見，多數運算發生在 JS 執行緒。若該執行緒阻塞，動畫將掉幀。且每幀都需透過 JS 至原生橋接層來更新原生視圖。

原生驅動程式的改良在於將所有步驟移至原生端。由於 Animated 會產生動畫節點圖，我們可在動畫開始時就序列化並一次性傳送至原生端，消除回調 JS 執行緒的需求；原生程式碼能直接於 UI 執行緒每幀更新視圖。

以下示範如何序列化動畫值與插值節點（非實際實作，僅為範例）。

建立原生值節點（將被動畫化的數值）：

```
NativeAnimatedModule.createNode({
  id: 1,
  type: 'value',
  initialValue: 0,
});
```

建立原生插值節點（告知原生驅動程式如何插值）：

```
NativeAnimatedModule.createNode({
  id: 2,
  type: 'interpolation',
  inputRange: [0, 10],
  outputRange: [10, 0],
  extrapolate: 'clamp',
});
```

建立原生屬性節點（標明其附加於視圖的哪個屬性）：

```
NativeAnimatedModule.createNode({
  id: 3,
  type: 'props',
  properties: ['style.opacity'],
});
```

連接節點：

```
NativeAnimatedModule.connectNodes(1, 2);
NativeAnimatedModule.connectNodes(2, 3);
```

將屬性節點連接至視圖：

```
NativeAnimatedModule.connectToView(3, ReactNative.findNodeHandle(viewRef));
```

如此一來，原生動畫模組便擁有所有必要資訊，能直接更新原生視圖，無需透過 JS 計算任何數值。

剩下的工作就是實際啟動動畫，指定我們想要的動畫曲線類型及要更新的動畫數值。時間動畫亦可透過預先在 JS 中計算每一幀來簡化，使原生實作更精簡。

```
NativeAnimatedModule.startAnimation({
  type: 'timing',
  frames: [0, 0.1, 0.2, 0.4, 0.65, ...],
  animatedValueId: 1,
});
```

以下是動畫運行時的實際流程解析：

- 原生端：原生動畫驅動器使用 `CADisplayLink` 或 `android.view.Choreographer` 在每一幀執行，並根據動畫曲線計算的新數值更新驅動值。
- 原生端：計算中間值並傳遞至附加於原生視圖的屬性節點。
- 原生端：更新 `UIView` 或 `android.View`。

如你所見，不再需要 JS 執行緒與橋接層，這意味著更流暢的動畫！🎉🎉

## 如何在應用程式中使用此功能？

對於一般動畫，方法很簡單：在啟動動畫時於設定中加入 `useNativeDriver: true` 即可。

修改前：

```
Animated.timing(this.state.animatedValue, {
  toValue: 1,
  duration: 500,
}).start();
```

修改後：

```
Animated.timing(this.state.animatedValue, {
  toValue: 1,
  duration: 500,
  useNativeDriver: true, // <-- Add this
}).start();
```

動畫數值僅相容單一驅動器，因此若你在某數值上啟動動畫時使用原生驅動器，請確保該數值的所有動畫皆採用原生驅動器。

此功能亦適用於 `Animated.event`，對於需跟隨滾動位置的動畫特別實用，因為在非原生驅動模式下，由於 React Native 的非同步特性，動畫總是會比手勢慢一幀。

修改前：

```
<ScrollView
  scrollEventThrottle={16}
  onScroll={Animated.event(
    [{ nativeEvent: { contentOffset: { y: this.state.animatedValue } } }]
  )}
>
  {content}
</ScrollView>
```

修改後：

```
<Animated.ScrollView // <-- Use the Animated ScrollView wrapper
  scrollEventThrottle={1} // <-- Use 1 here to make sure no events are ever missed
  onScroll={Animated.event(
    [{ nativeEvent: { contentOffset: { y: this.state.animatedValue } } }],
    { useNativeDriver: true } // <-- Add this
  )}
>
  {content}
</Animated.ScrollView>
```

## 注意事項

並非所有 Animated 功能皆支援原生動畫。主要限制在於僅能動畫化非佈局屬性，例如 `transform` 和 `opacity` 可正常運作，但 Flexbox 與定位屬性則否。另一限制是 `Animated.event` 僅適用於直接事件，不支援冒泡事件。這意味著它無法與 `PanResponder` 協作，但可搭配 `ScrollView#onScroll` 等事件使用。

原生動畫雖已內建於 React Native 一段時間，但因其被視為實驗性功能而未被列入文件。因此若欲使用此功能，請確保你的 React Native 版本為近期發佈（0.40+）。

## 參考資源

For more information about animated I recommend watching [this talk](https://www.youtube.com/watch?v=xtqUJVqpKNo) by [Christopher Chedeau](https://twitter.com/Vjeux).

If you want a deep dive into animations and how offloading them to native can improve user experience there is also [this talk](https://www.youtube.com/watch?v=qgSMjYWqBk4) by [Krzysztof Magiera](https://twitter.com/kzzzf).