---
id: landing-page
title: About the New Architecture
---

:::info
若您正在尋找新架構指南，相關內容已遷移至[工作小組](https://github.com/reactwg/react-native-new-architecture#guides)。
:::

自2018年起，React Native團隊便開始重新設計React Native的核心架構，旨在讓開發者能打造更高品質的使用體驗。截至2024年，此版本React Native已通過大規模驗證，並支撐著Meta旗下多款生產級應用程式。

The term _New Architecture_ refers to both the new framework architecture and the work to bring it to open source.

新架構自[React Native 0.68](/blog/2022/03/30/version-068#opting-in-to-the-new-architecture)起已開放實驗性啟用選項，並在後續每個版本持續改進。團隊現正致力使其成為React Native開源生態的預設體驗。

## 為何需要新架構？

經過多年React Native開發實踐，團隊識別出一系列阻礙開發者打造高品質體驗的根本性限制。這些限制源自框架現有設計，因此新架構成為對React Native未來的戰略投資。

新架構實現了舊架構無法達成的功能與改進：

### 同步佈局與效果

建構自適應UI時，經常需要測量視圖尺寸位置並調整佈局。

Today, you would use the [`onLayout`](/docs/view#onlayout) event to get the layout information of a view and make any adjustments. However, state updates within the `onLayout` callback may apply after painting the previous render. This means that users may see intermediate states or visual jumps between rendering the initial layout and responding to layout measurements.

新架構透過同步存取佈局資訊與恰當調度更新，能徹底避免此問題，確保使用者不會看到中間狀態。

<details>
<summary>Example: Rendering a Tooltip</summary>

Measuring and placing a tooltip above a view allows us to showcase what synchronous rendering unlocks. The tooltip needs to know the position of its target view to determine where it should render.

In the current architecture, we use `onLayout` to get the measurements of the view and then update the positioning of the tooltip based on where the view is.

```jsx
function ViewWithTooltip() {
  // ...

  // We get the layout information and pass to ToolTip to position itself
  const onLayout = React.useCallback(event => {
    targetRef.current?.measureInWindow((x, y, width, height) => {
      // This state update is not guaranteed to run in the same commit
      // This results in a visual "jump" as the ToolTip repositions itself
      setTargetRect({x, y, width, height});
    });
  }, []);

  return (
    <>
      <View ref={targetRef} onLayout={onLayout}>
        <Text>Some content that renders a tooltip above</Text>
      </View>
      <Tooltip targetRect={targetRect} />
    </>
  );
}
```

With the New Architecture, we can use [`useLayoutEffect`](https://react.dev/reference/react/useLayoutEffect) to synchronously measure and apply layout updates in a single commit, avoiding the visual "jump".

```jsx
function ViewWithTooltip() {
  // ...

  useLayoutEffect(() => {
    // The measurement and state update for `targetRect` happens in a single commit
    // allowing ToolTip to position itself without intermediate paints
    targetRef.current?.measureInWindow((x, y, width, height) => {
      setTargetRect({x, y, width, height});
    });
  }, [setTargetRect]);

  return (
    <>
      <View ref={targetRef}>
        <Text>Some content that renders a tooltip above</Text>
      </View>
      <Tooltip targetRect={targetRect} />
    </>
  );
}
```

<div className="TwoColumns TwoFigures">
 <figure>
  <img src="/img/new-architecture/async-on-layout.gif" alt="A view that is moving to the corners of the viewport and center with a tooltip rendered either above or below it. The tooltip is rendered after a short delay after the view moves" />
  <figcaption>Asynchronous measurement and render of the ToolTip. [See code](https://gist.github.com/lunaleaps/eabd653d9864082ac1d3772dac217ab9).</figcaption>
</figure>
<figure>
  <img src="/img/new-architecture/sync-use-layout-effect.gif" alt="A view that is moving to the corners of the viewport and center with a tooltip rendered either above or below it. The view and tooltip move in unison." />
  <figcaption>Synchronous measurement and render of the ToolTip. [See code](https://gist.github.com/lunaleaps/148756563999c83220887757f2e549a3).</figcaption>
</figure>
</div>

</details>

### 支援並行渲染器與特性

新架構支援[React 18](https://react.dev/blog/2022/03/29/react-v18)引入的並行渲染與特性。您現在可在React Native程式碼中使用Suspense資料獲取、Transitions等新API，進一步統一網頁與原生React開發的概念體系。

並行渲染器還帶來自動批次處理等開箱即用的改進，能減少React不必要的重新渲染。

<details>
<summary>Example: Automatic Batching</summary>

With the New Architecture, you'll get automatic batching with the React 18 renderer.

In this example, a slider specifies how many tiles to render. Dragging the slider from 0 to 1000 will fire off a quick succession of state updates and re-renders.

In comparing the renderers for the [same code](https://gist.github.com/lunaleaps/79bb6f263404b12ba57db78e5f6f28b2), you can visually notice the renderer provides a smoother UI, with less intermediate UI updates. State updates from native event handlers, like this native Slider component, are now batched.

<div className="TwoColumns TwoFigures">
 <figure>
  <img src="/img/new-architecture/legacy-renderer.gif" alt="A video demonstrating an app rendering many views according to a slider input. The slider value is adjusted from 0 to 1000 and the UI slowly catches up to rendering 1000 views." />
  <figcaption>Rendering frequent state updates with legacy renderer.</figcaption>
</figure>
<figure>
  <img src="/img/new-architecture/react18-renderer.gif" alt="A video demonstrating an app rendering many views according to a slider input. The slider value is adjusted from 0 to 1000 and the UI resolves to 1000 views faster than the previous example, without as many intermediate states." />
  <figcaption>Rendering frequent state updates with React 18 renderer.</figcaption>
</figure>
</div>
</details>

New concurrent features, like [Transitions](https://react.dev/reference/react/useTransition), give you the power to express the priority of UI updates. Marking an update as lower priority tells React it can "interrupt" rendering the update to handle higher priority updates to ensure a responsive user experience where it matters.

<details>
<summary>Example: Using `startTransition`</summary>

We can build on the previous example to showcase how transitions can interrupt in-progress rendering to handle a newer state update.

We wrap the tile number state update with `startTransition` to indicate that rendering the tiles can be interrupted. `startTransition` also provides a `isPending` flag to tell us when the transition is complete.

```jsx
function TileSlider({value, onValueChange}) {
  const [isPending, startTransition] = useTransition();

  return (
    <>
      <View>
        <Text>
          Render {value} Tiles
        </Text>
        <ActivityIndicator animating={isPending} />
      </View>
      <Slider
        value={1}
        minimumValue={1}
        maximumValue={1000}
        step={1}
        onValueChange={newValue => {
          startTransition(() => {
            onValueChange(newValue);
          });
        }}
      />
    </>
  );
}

function ManyTiles() {
  const [value, setValue] = useState(1);
  const tiles = generateTileViews(value);
  return (
      <TileSlider onValueChange={setValue} value={value} />
      <View>
        {tiles}
      </View>
  )
}
```

You'll notice that with the frequent updates in a transition, React renders fewer intermediate states because it bails out of rendering the state as soon as it becomes stale. In comparison, without transitions, more intermediate states are rendered. Both examples still use automatic batching. Still, transitions give even more power to developers to batch in-progress renders.

<div className="TwoColumns TwoFigures">
<figure>
  <img src="/img/new-architecture/with-transitions.gif" alt="A video demonstrating an app rendering many views (tiles) according to a slider input. The views are rendered in batches as the slider is quickly adjusted from 0 to 1000. There are less batch renders in comparison to the next video." />
  <figcaption>Rendering tiles with transitions to interrupt in-progress renders of stale state. [See code](https://gist.github.com/lunaleaps/eac391bf3fe4c85953cefeb74031bab0/revisions).</figcaption>
</figure>
<figure>
  <img src="/img/new-architecture/without-transitions.gif" alt="A video demonstrating an app rendering many views (tiles) according to a slider input. The views are rendered in batches as the slider is quickly adjusted from 0 to 1000." />
  <figcaption>Rendering tiles without marking it as a transition. [See code](https://gist.github.com/lunaleaps/eac391bf3fe4c85953cefeb74031bab0/revisions).</figcaption>
</figure>
</div>
</details>

### 高效的JavaScript/原生交互

新架構以JavaScript Interface (JSI)取代原先JavaScript與原生端之間的[非同步橋接](https://reactnative.dev/blog/2018/06/14/state-of-react-native-2018#architecture)。JSI允許JavaScript持有C++物件的引用（反之亦然），透過記憶體引用可直接呼叫方法，消除序列化開銷。

JSI使熱門相機函式庫[VisionCamera](https://github.com/mrousavy/react-native-vision-camera)能即時處理影格。典型影格緩衝區約10MB，相當於每秒處理1GB資料量（依影格率而定）。相較橋接序列化的高昂成本，JSI能輕鬆處理此量級的交互數據，並可暴露資料庫、影像、音訊樣本等複雜實例型別。

新架構中採用的 JSI 技術移除了所有原生與 JavaScript 互操作中的序列化工作負載。這包括初始化與重新渲染如 `View` 和 `Text` 等原生核心元件。您可以在新架構的[渲染效能研究討論](https://github.com/reactwg/react-native-new-architecture/discussions/123)中閱讀更多關於我們測得的改進基準。

### 深入瞭解

為實現這些改進，新架構必須重構 React Native 基礎設施的多個部分。要瞭解更多關於重構及其帶來的其他優勢，請查閱新架構工作組的[文件](https://github.com/reactwg/react-native-new-architecture)。

## 啟用新架構能帶來什麼？

雖然新架構實現了這些功能與改進，但為您的應用程式或函式庫啟用新架構可能不會立即提升效能或使用者體驗。

例如，您的程式碼可能需要重構才能充分利用同步佈局效果或並行功能等新特性。儘管 JSI 能最小化 JavaScript 與原生記憶體之間的開銷，但資料序列化可能並非您應用效能的瓶頸。

為您的應用程式或函式庫啟用新架構，即是選擇邁向 React Native 的未來。

團隊正在積極研究與開發新架構所解鎖的潛能。例如，與網頁技術的對齊是 Meta 正在探索的領域，未來將發布至 React Native 開源生態系統。

- [事件循環模型的更新](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0744-well-defined-event-loop.md)
- [節點與佈局 API](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0607-dom-traversal-and-layout-apis.md)
- [樣式與佈局一致性](https://github.com/facebook/yoga/releases/tag/v2.0.0)

您可以在我們專屬的[討論與提案](https://github.com/react-native-community/discussions-and-proposals/discussions/651)儲存庫中追蹤進度並參與貢獻。

## 我現在應該使用新架構嗎？

目前新架構仍被視為實驗性功能，我們持續完善向後兼容性以提供更好的採用體驗。

團隊計劃在 2024 年底前發布的 React Native 版本中預設啟用新架構。

我們的建議如下：

- For most production apps, we do _not_ recommend enabling the New Architecture today. Waiting for the official release will offer the best experience.
- If you maintain a React Native library, we recommend enabling it and verifying your use cases are covered. You can find the [instructions here](https://github.com/reactwg/react-native-new-architecture#guides).

### 啟用新架構

若您有興趣親身體驗新架構，可在我們專屬的工作組中找到[啟用指引](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/enable-apps.md)。[新架構工作組](https://github.com/reactwg/react-native-new-architecture)是專門提供新架構採用支援與協調的空間，團隊會在此定期發布更新。