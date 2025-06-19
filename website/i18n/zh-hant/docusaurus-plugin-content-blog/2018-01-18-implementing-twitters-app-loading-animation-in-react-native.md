---
title: 'Implementing Twitter’s App Loading Animation in React Native'
authors: [Eli White]
tags: [engineering]
---

Twitter 的 iOS 應用有一個我非常喜歡的載入動畫。

<img src="/blog/assets/loading-screen-01.gif" style={{float: 'left', paddingRight: 80, paddingBottom: 20}} />

當應用準備就緒時，Twitter 標誌會愉悅地展開，顯示出應用內容。

我想弄清楚如何用 React Native 重新創建這個載入動畫。

<hr style={{clear: 'both', marginBottom: 40, width: 80}} />

To understand _how_ to build it, I first had to understand the difference pieces of the loading animation. The easiest way to see the subtlety is to slow it down.

<img src="/blog/assets/loading-screen-02.gif" style={{marginTop: 20, float: 'left', paddingRight: 80, paddingBottom: 20}} />

其中有幾個主要部分需要我們弄清楚如何構建。

1. 縮放小鳥圖標。
1. 隨著小鳥圖標變大，顯示下方的應用內容
1. 最後稍微縮小應用內容

我花了相當長的時間才弄清楚如何製作這個動畫。

I started with an _incorrect_ assumption that the blue background and Twitter bird were a layer on _top_ of the app and that as the bird grew, it became transparent which revealed the app underneath. This approach doesn’t work because the Twitter bird becoming transparent would show the blue layer, not the app underneath!

親愛的讀者，幸運的是，你不必經歷我同樣的挫折。你可以直接跳過這些，閱讀這篇精彩的教程！

<hr style={{clear: 'both', marginBottom: 40, width: 80}} />

## 正確的方法

在我們開始編寫代碼之前，理解如何分解這個效果非常重要。為了幫助可視化這個效果，我在 [CodePen](https://codepen.io/TheSavior/pen/NXNoJM) 上重新創建了它（嵌入在接下來的幾段中），這樣你可以互動式地看到不同的圖層。

<img src="/blog/assets/loading-screen-03.png" style={{float: 'left', paddingRight: 80, paddingBottom: 20}} />

這個效果有三個主要圖層。第一個是藍色背景圖層。儘管它看起來像是在應用內容的上方，但它實際上是在後方。

然後我們有一個純白色的圖層。最後，在最前面的是我們的應用內容。

<hr style={{clear: 'both', marginBottom: 40, width: 80}} />

<img src="/blog/assets/loading-screen-04.png" style={{float: 'left', paddingRight: 80, paddingBottom: 20}} />

The main trick to this animation is using the Twitter logo as a `mask` and masking both the app, and the white layer. I won’t go too deep on the details of masking, there are [plenty](https://www.html5rocks.com/en/tutorials/masking/adobe/) of [resources](https://designshack.net/articles/graphics/a-complete-beginners-guide-to-masking-in-photoshop/) [online](https://www.sketchapp.com/docs/shapes/masking/) for that.

在這個上下文中，遮罩的基本原理是：遮罩的不透明像素會顯示它們遮罩的內容，而透明像素則會隱藏它們遮罩的內容。

我們使用 Twitter 標誌作為遮罩，並讓它遮罩兩個圖層：純白色圖層和應用內容圖層。

為了顯示應用內容，我們將遮罩放大，直到它比整個屏幕還要大。

在遮罩放大的同時，我們逐漸增加應用內容圖層的不透明度，顯示應用內容並隱藏其後的純白色圖層。為了完成效果，我們將應用內容圖層的初始縮放比例設為大於 1，並在動畫結束時將其縮小到 1。然後我們隱藏非應用內容的圖層，因為它們之後不會再被看到。

俗話說，一張圖片勝過千言萬語。那麼一個互動式可視化又值多少字呢？點擊「下一步」按鈕來播放動畫。顯示圖層可以讓你從側面視角觀察。網格是為了幫助可視化透明圖層。

<iframe
  height="750"
  scrolling="no"
  title="Loading Screen Animation Steps"
  src="//codepen.io/TheSavior/embed/NXNoJM/?height=265&theme-id=0&default-tab=result&embed-version=2"
  frameborder="no"
  allowFullScreen={true}
  className="codepen">
  See the Pen{' '}
  <a href="https://codepen.io/TheSavior/pen/NXNoJM/">
    Loading Screen Animation Steps
  </a>
  {' '}by Eli White (
  <a href="https://codepen.io/TheSavior">@TheSavior</a>) on{' '}
  <a href="https://codepen.io">CodePen</a>.
</iframe>

## 現在，進入 React Native 部分

好了。現在我們知道了我們要構建什麼以及動畫的工作原理，我們可以開始編寫代碼了——這才是你真正來這裡的目的。

這個效果的關鍵在於 [MaskedViewIOS](https://reactnative.dev/docs/0.63/maskedviewios)，這是 React Native 的核心組件之一。

```jsx
import {MaskedViewIOS} from 'react-native';

<MaskedViewIOS maskElement={<Text>Basic Mask</Text>}>
  <View style={{backgroundColor: 'blue'}} />
</MaskedViewIOS>;
```

`MaskedViewIOS` takes props `maskElement` and `children`. The children are masked by the `maskElement`. Note that the mask doesn’t need to be an image, it can be any arbitrary view. The behavior of the above example would be to render the blue view, but for it to be visible only where the words “Basic Mask” are from the `maskElement`. We just made complicated blue text.

我們要做的是渲染藍色層，然後在上面渲染被 Twitter 標誌遮罩的應用程式和白色層。

```jsx
{
  fullScreenBlueLayer;
}
<MaskedViewIOS
  style={{flex: 1}}
  maskElement={
    <View style={styles.centeredFullScreen}>
      <Image source={twitterLogo} />
    </View>
  }>
  {fullScreenWhiteLayer}
  <View style={{flex: 1}}>
    <MyApp />
  </View>
</MaskedViewIOS>;
```

這樣我們就能得到下面看到的圖層結構。

<img src="/blog/assets/loading-screen-04.png" style={{marginLeft: 'auto', marginRight: 'auto', display: 'block'}} />

## 現在進入動畫部分

我們已經具備所有必要的組件，下一步是讓它們動起來。為了讓動畫效果流暢，我們將使用 React Native 的 [Animated](/docs/animated) API。

Animated 讓我們能夠以宣告式的方式在 JavaScript 中定義動畫。默認情況下，這些動畫在 JavaScript 中運行，並在每一幀告訴原生層需要做哪些更改。儘管 JavaScript 會嘗試在每一幀更新動畫，但它可能無法足夠快地完成，從而導致掉幀（卡頓）。這不是我們想要的！

Animated has special behavior to allow you to get animations without this jank. Animated has a flag called `useNativeDriver` which sends your animation definition from JavaScript to native at the beginning of your animation, allowing the native side to process the updates to your animation without having to go back and forth to JavaScript every frame. The downside of `useNativeDriver` is you can only update a specific set of properties, mostly `transform` and `opacity`. You can’t animate things like background color with `useNativeDriver`, at least not yet — we will add more over time, and of course you can always submit a PR for properties you need for your project, benefitting the whole community 😀.

由於我們希望這個動畫流暢，我們將在這些限制下工作。要更深入了解 `useNativeDriver` 的內部工作原理，請查看我們的 [博客文章](/blog/2017/02/14/using-native-driver-for-animated)。

## 分解我們的動畫

我們的動畫有 4 個組成部分：

1. 放大鳥標誌，顯示應用程式和純白色層
1. 淡入應用程式
1. 縮小應用程式
1. 完成後隱藏白色層和藍色層

使用 Animated 時，有兩種主要方式來定義動畫。第一種是使用 `Animated.timing`，它讓你可以精確指定動畫運行的時間長度，並使用緩動曲線來平滑運動。另一種方法是使用基於物理的 API，例如 `Animated.spring`。使用 `Animated.spring` 時，你指定彈簧的摩擦力和張力等參數，讓物理來運行你的動畫。

我們有多個動畫需要同時運行，這些動畫彼此密切相關。例如，我們希望在遮罩部分顯示時開始淡入應用程式。由於這些動畫密切相關，我們將使用 `Animated.timing` 和單一的 `Animated.Value`。

`Animated.Value` 是一個原生值的包裝器，Animated 用它來了解動畫的狀態。通常情況下，一個完整的動畫你只需要一個這樣的變量。大多數使用 Animated 的組件會將這個值存儲在狀態中。

由於我將這個動畫視為在完整動畫的不同時間點發生的步驟，我們將從 0 開始我們的 `Animated.Value`，表示 0% 完成，並在 100 結束，表示 100% 完成。

我們的初始組件狀態如下。

```jsx
state = {
  loadingProgress: new Animated.Value(0),
};
```

當我們準備開始動畫時，我們會告訴 Animated 將這個值動畫化到 100。

```jsx
Animated.timing(this.state.loadingProgress, {
  toValue: 100,
  duration: 1000,
  useNativeDriver: true, // This is important!
}).start();
```

接著，我嘗試估算動畫的各個部分，以及它們在整體動畫不同階段應該具有的值。以下是一個表格，顯示了動畫的各個部分，以及我認為它們在時間進展過程中不同點應該具有的值。

![](/blog/assets/loading-screen-05.png)

Twitter 鳥形遮罩應該從縮放比例 1 開始，在放大之前會先變小。因此，在動畫進行到 10% 時，它的縮放值應該是 0.8，然後在結束時放大到 70。選擇 70 其實相當隨意，它需要足夠大以確保鳥形完全揭示屏幕，而 60 還不夠大 😀。有趣的是，數字越大，它看起來增長得越快，因為它必須在相同的時間內達到目標。這個數字經過了一些試驗和錯誤，才能讓這個標誌看起來好看。不同大小的標誌或設備需要不同的結束縮放比例，以確保整個屏幕被完全揭示。

應用程式應該保持不透明一段時間，至少在 Twitter 標誌變小的過程中。根據官方動畫，我希望在鳥形放大到一半時開始顯示它，並在整體動畫進行到 30% 時完全顯示它。因此，在 15% 時開始顯示，到 30% 時完全可見。

應用程式的縮放從 1.1 開始，並在動畫結束時縮小到正常比例。

## 現在，用代碼實現。

我們上面所做的本質上是將動畫進度百分比的值映射到各個部分的值。我們使用 Animated 的 `.interpolate` 方法來實現這一點。我們創建了三個不同的樣式對象，每個對象對應動畫的一個部分，這些樣式對象的值是基於 `this.state.loadingProgress` 的插值。

```jsx
const loadingProgress = this.state.loadingProgress;

const opacityClearToVisible = {
  opacity: loadingProgress.interpolate({
    inputRange: [0, 15, 30],
    outputRange: [0, 0, 1],
    extrapolate: 'clamp',
    // clamp means when the input is 30-100, output should stay at 1
  }),
};

const imageScale = {
  transform: [
    {
      scale: loadingProgress.interpolate({
        inputRange: [0, 10, 100],
        outputRange: [1, 0.8, 70],
      }),
    },
  ],
};

const appScale = {
  transform: [
    {
      scale: loadingProgress.interpolate({
        inputRange: [0, 100],
        outputRange: [1.1, 1],
      }),
    },
  ],
};
```

現在我們有了這些樣式對象，可以在渲染文章前面提到的視圖片段時使用它們。注意，只有 `Animated.View`、`Animated.Text` 和 `Animated.Image` 能夠使用包含 `Animated.Value` 的樣式對象。

```jsx
const fullScreenBlueLayer = (
  <View style={styles.fullScreenBlueLayer} />
);
const fullScreenWhiteLayer = (
  <View style={styles.fullScreenWhiteLayer} />
);

return (
  <View style={styles.fullScreen}>
    {fullScreenBlueLayer}
    <MaskedViewIOS
      style={{flex: 1}}
      maskElement={
        <View style={styles.centeredFullScreen}>
          <Animated.Image
            style={[styles.maskImageStyle, imageScale]}
            source={twitterLogo}
          />
        </View>
      }>
      {fullScreenWhiteLayer}
      <Animated.View
        style={[opacityClearToVisible, appScale, {flex: 1}]}>
        {this.props.children}
      </Animated.View>
    </MaskedViewIOS>
  </View>
);
```

<img src="/blog/assets/loading-screen-06.gif" style={{float: 'left', paddingRight: 80, paddingBottom: 20}} />

太好了！我們現在讓動畫的各個部分看起來如我們所願。現在我們只需要清理那些永遠不會再看到的藍色和白色圖層。

要知道何時可以清理它們，我們需要知道動畫何時完成。幸運的是，在我們調用 `Animated.timing` 的地方，`.start` 方法接受一個可選的回調函數，該函數會在動畫完成時運行。

```jsx
Animated.timing(this.state.loadingProgress, {
  toValue: 100,
  duration: 1000,
  useNativeDriver: true,
}).start(() => {
  this.setState({
    animationDone: true,
  });
});
```

現在我們有了一個 `state` 中的值來知道動畫是否完成，我們可以修改我們的藍色和白色圖層來使用它。

```jsx
const fullScreenBlueLayer = this.state.animationDone ? null : (
  <View style={[styles.fullScreenBlueLayer]} />
);
const fullScreenWhiteLayer = this.state.animationDone ? null : (
  <View style={[styles.fullScreenWhiteLayer]} />
);
```

瞧！我們的動畫現在可以工作了，並且在動畫完成後清理了未使用的圖層。我們已經構建了 Twitter 應用程式的加載動畫！

## 但是等等，我的動畫不工作！

別擔心，親愛的讀者。我也討厭那些只給你代碼片段而不提供完整源碼的指南。

這個組件已經發布到 npm，並在 GitHub 上作為 [react-native-mask-loader](https://github.com/TheSavior/react-native-mask-loader)。如果你想在手機上試試，可以在 [Expo](https://expo.io/@eliwhite/react-native-mask-loader-example) 上找到它：

<img src="/blog/assets/loading-screen-07.png" style={{marginLeft: 'auto', marginRight: 'auto', display: 'block'}} />

## 更多閱讀 / 額外練習

1. [這本 gitbook](https://browniefed.com/react-native-animation-book/) 是閱讀完 React Native 文件後，深入學習 Animated 的絕佳資源。
1. 實際的 Twitter 動畫似乎會在接近尾聲時加速遮罩的顯現。試著修改載入器，使用不同的緩動函數（或彈簧效果！）來更貼近該行為。
1. 目前遮罩的結束縮放比例是硬編碼的，在平板上可能無法完整顯現整個應用程式。根據螢幕尺寸和圖片大小計算結束縮放比例，會是一個很棒的 Pull Request。