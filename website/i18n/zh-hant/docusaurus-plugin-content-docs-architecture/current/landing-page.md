---
id: landing-page
title: About the New Architecture
---

自2018年起，React Native團隊開始重新設計框架核心架構，旨在讓開發者能打造更高品質的使用體驗。截至2024年，此版本React Native已通過大規模驗證，並成為Meta旗下多款生產級應用的技術基礎。

The term _New Architecture_ refers to both the new framework architecture and the work to bring it to open source.

新架構自[React Native 0.68](/blog/2022/03/30/version-068#opting-in-to-the-new-architecture)起開放實驗性啟用選項，並在後續每個版本持續改進。團隊現正致力使其成為React Native開源生態的預設體驗。

## 為何需要新架構？

經過多年React Native開發實踐，團隊識別出原有設計中存在根本性限制，這些限制阻礙開發者實現高度精緻的特定體驗。新架構正是為突破這些限制而啟動的框架未來投資。

新架構實現了舊架構無法達成的關鍵能力與改進。

### 同步佈局與效果

建構自適應UI時，常需測量視圖尺寸位置並動態調整佈局。

Today, you would use the [`onLayout`](/docs/view#onlayout) event to get the layout information of a view and make any adjustments. However, state updates within the `onLayout` callback may apply after painting the previous render. This means that users may see intermediate states or visual jumps between rendering the initial layout and responding to layout measurements.

新架構透過同步存取佈局資訊與恰當調度更新，能完全避免此問題，確保用戶不會看到中間狀態。

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

### 並發渲染器與功能支援

新架構支援[React 18](https://react.dev/blog/2022/03/29/react-v18)引入的並發渲染與功能。現在您可在React Native程式碼中使用Suspense資料獲取、Transitions等新API，進一步統一網頁與原生React開發的概念體系。

並發渲染器還帶來自動批次處理等開箱即用的改進，能減少React不必要的重新渲染。

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

新架構以JavaScript Interface (JSI)取代原有的[非同步橋接](https://reactnative.dev/blog/2018/06/14/state-of-react-native-2018#architecture)。JSI允許JavaScript與C++對象互相持有記憶體引用，實現免序列化的直接方法調用。

JSI使熱門相機庫[VisionCamera](https://github.com/mrousavy/react-native-vision-camera)能即時處理影像幀。典型幀緩衝區約10MB，以每秒幀率計算相當於近1GB數據傳輸。相較橋接序列化的開銷，JSI能輕鬆處理此量級的交互數據，並可暴露資料庫、圖像、音訊樣本等複雜實例型別。

新架構全面採用JSI後，所有原生-JavaScript交互場景（包括`View`、`Text`等核心元件的初始化與重新渲染）都不再需要序列化工作。您可參閱我們對[新架構渲染效能的實測分析](https://github.com/reactwg/react-native-new-architecture/discussions/123)了解具體改進指標。

## 啟用新架構後能獲得哪些優勢？

雖然新架構能實現這些功能與改進，但為您的應用程式或函式庫啟用新架構並不會立即提升效能或使用者體驗。

舉例來說，您的程式碼可能需要重構才能充分利用同步佈局效果或並行功能等新特性。儘管 JSI 能最小化 JavaScript 與原生記憶體間的開銷，但資料序列化可能並非您應用效能的瓶頸所在。

為應用程式或函式庫啟用新架構，即是選擇擁抱 React Native 的未來發展。

團隊正積極研究並開發新架構解鎖的潛在能力。例如，Meta 正在探索的網頁對齊技術，未來將釋出至 React Native 開源生態系統。

- [事件循環模型的更新](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0744-well-defined-event-loop.md)
- [節點與佈局 API](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0607-dom-traversal-and-layout-apis.md)
- [樣式與佈局一致性](https://github.com/facebook/yoga/releases/tag/v2.0.0)

您可以在專屬的[討論與提案](https://github.com/react-native-community/discussions-and-proposals/discussions/651)儲存庫追蹤進度並參與貢獻。

## 我現在應該採用新架構嗎？

自 0.76 版起，所有 React Native 專案預設均已啟用新架構。

若發現任何運作異常，請使用[此範本](https://github.com/facebook/react-native/issues/new?assignees=&labels=Needs%3A+Triage+%3Amag%3A%2CType%3A+New+Architecture&projects=&template=new_architecture_bug_report.yml)提交問題。

若因故無法使用新架構，您仍可選擇退出：

### Android 平台

1. 開啟 `android/gradle.properties` 檔案
2. 將 `newArchEnabled` 標記從 `true` 切換為 `false`

```diff title="gradle.properties"
# Use this property to enable support to the new architecture.
# This will allow you to use TurboModules and the Fabric render in
# your application. You should enable this flag either if you want
# to write custom TurboModules/Fabric components OR use libraries that
# are providing them.
-newArchEnabled=true
+newArchEnabled=false
```

### iOS 平台

1. 開啟 `ios/Podfile` 檔案
2. 在 Podfile 主作用域內新增 `ENV['RCT_NEW_ARCH_ENABLED'] = '0'`（參照範本中的[參考 Podfile](https://github.com/react-native-community/template/blob/0.76-stable/template/ios/Podfile)）

```diff
+ ENV['RCT_NEW_ARCH_ENABLED'] = '0'
# Resolve react_native_pods.rb with node to allow for hoisting
require Pod::Executable.execute_command('node', ['-p',
  'require.resolve(
```

3. 執行以下指令安裝 CocoaPods 相依套件：

```shell
bundle exec pod install
```