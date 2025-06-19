---
id: render-pipeline
title: Render, Commit, and Mount
---

import FabricWarning from './\_fabric-warning.mdx';

<FabricWarning />

React Native 渲染器會執行一系列工作，將 React 邏輯渲染到[宿主平台](architecture-glossary.md#host-platform)。這一系列工作稱為渲染管線，無論是初始渲染或 UI 狀態更新時都會發生。本文件將說明渲染管線的運作方式，以及在不同情境下的差異。

渲染管線大致可分為三個階段：

1. **Render:** React executes product logic which creates a [React Element Trees](architecture-glossary.md#react-element-tree-and-react-element) in JavaScript. From this tree, the renderer creates a [React Shadow Tree](architecture-glossary.md#react-shadow-tree-and-react-shadow-node) in C++.
2. **Commit**: After a React Shadow Tree is fully created, the renderer triggers a commit. This **promotes** both the React Element Tree and the newly created React Shadow Tree as the “next tree” to be mounted. This also schedules calculation of its layout information.
3. **Mount:** The React Shadow Tree, now with the results of layout calculation, is transformed into a [Host View Tree](architecture-glossary.md#host-view-tree-and-host-view).

> 渲染管線的各階段可能在不同執行緒上執行。詳情請參閱[執行緒模型](threading-model)文件。

![React Native 渲染器資料流](/docs/assets/Architecture/renderer-pipeline/data-flow.jpg)

---

## 初始渲染

假設您要渲染以下內容：

```jsx
function MyComponent() {
  return (
    <View>
      <Text>Hello, World</Text>
    </View>
  );
}

// <MyComponent />
```

In the example above, `<MyComponent />` is a [React Element](architecture-glossary.md#react-element-tree-and-react-element). React recursively reduces this _React Element_ to a terminal [React Host Component](architecture-glossary.md#react-host-components-or-host-components) by invoking it (or its `render` method if implemented with a JavaScript class) until every _React Element_ cannot be reduced any further. Now you have a _React Element Tree_ of [React Host Components](architecture-glossary.md#react-host-components-or-host-components).

### 階段 1. 渲染

![階段一：渲染](/docs/assets/Architecture/renderer-pipeline/phase-one-render.png)

During this process of element reduction, as each _React Element_ is invoked, the renderer also synchronously creates a [React Shadow Node](architecture-glossary.md#react-shadow-tree-and-react-shadow-node). This happens only for _React Host Components_, not for [React Composite Components](architecture-glossary.md#react-composite-components). In the example above, the `<View>` leads to the creation of a `ViewShadowNode` object, and the
`<Text>` leads to the creation of a `TextShadowNode` object. Notably, there is never a _React Shadow Node_ that directly represents `<MyComponent>`.

Whenever React creates a parent-child relationship between two _React Element Nodes_, the renderer creates the same relationship between the corresponding _React Shadow Nodes_. This is how the _React Shadow Tree_ is assembled.

**補充說明**

- The operations (creation of _React Shadow Node_, creation of parent-child relationship between two _React Shadow Nodes_) are synchronous and thread-safe operations that are executed from React (JavaScript) into the renderer (C++), usually on the JavaScript thread.
- The _React Element Tree_ (and its constituent _React Element Nodes_) do not exist indefinitely. It is a temporal representation materialized by “fibers” in React. Each “fiber” that represents a host component stores a C++ pointer to the _React Shadow Node_, made possible by JSI. [Learn more about “fibers” in this document.](https://github.com/acdlite/react-fiber-architecture#what-is-a-fiber)
- The _React Shadow Tree_ is immutable. In order to update any _React Shadow Node_, the renderer creates a new _React Shadow Tree_. However, the renderer provides cloning operations to make state updates more performant (see [React State Updates](render-pipeline#react-state-updates) for more details).

在上述範例中，渲染階段的結果如下：

![步驟一](/docs/assets/Architecture/renderer-pipeline/render-pipeline-1.png)

當 _React Shadow Tree_ 完成後，渲染器會觸發 _React Element Tree_ 的提交（commit）。

### 階段二：提交（Commit）

![階段二：提交](/docs/assets/Architecture/renderer-pipeline/phase-two-commit.png)

提交階段包含兩個操作：_佈局計算（Layout Calculation）_ 和 _樹晉升（Tree Promotion）_。

- **佈局計算：** 此操作計算每個 _React Shadow Node_ 的位置和大小。在 React Native 中，這涉及調用 Yoga 來計算每個 _React Shadow Node_ 的佈局。實際計算需要每個 _React Shadow Node_ 的樣式（源自 JavaScript 中的 _React Element_），以及 _React Shadow Tree_ 根節點的佈局約束（決定節點可佔用的可用空間）。

![步驟二](/docs/assets/Architecture/renderer-pipeline/render-pipeline-2.png)

- **樹晉升（新樹 → 下一棵樹）：** 此操作將新的 _React Shadow Tree_ 晉升為「下一棵樹」，準備掛載。此晉升表示新的 _React Shadow Tree_ 已具備所有掛載所需的資訊，並代表 _React Element Tree_ 的最新狀態。「下一棵樹」會在 UI 線程的下一個「tick」時掛載。

**補充細節**

- 這些操作在背景線程上非同步執行。
- 多數佈局計算完全在 C++ 中執行。但某些元件的佈局計算依賴於 _宿主平台_（例如 `Text`、`TextInput` 等）。文字的大小和位置因 _宿主平台_ 而異，需在 _宿主平台_ 層計算。為此，Yoga 會調用 _宿主平台_ 定義的函數來計算元件的佈局。

### 階段三：掛載（Mount）

![階段三：掛載](/docs/assets/Architecture/renderer-pipeline/phase-three-mount.png)

The mount phase transforms the _React Shadow Tree_ (which now contains data from layout calculation) into a _Host_ _View Tree_ with rendered pixels on the screen. As a reminder, the _React Element Tree_ looks like this:

```jsx
<View>
  <Text>Hello, World</Text>
</View>
```

At a high level, React Native renderer creates a corresponding [Host View](architecture-glossary.md#host-view-tree-and-host-view) for each _React Shadow Node_ and mounts it on screen. In the example above, the renderer creates an instance of `android.view.ViewGroup` for the `<View>` and `android.widget.TextView` for `<Text>` and populates it with “Hello World”. Similarly for iOS a `UIView` is created and text is populated with a call to `NSLayoutManager`. Each host view is then configured to use props from its React Shadow Node, and its size and position is configured using the calculated layout information.

![步驟二](/docs/assets/Architecture/renderer-pipeline/render-pipeline-3.png)

更詳細地說，掛載階段包含以下三個步驟：

- **Tree Diffing:** This step computes the diff between the “previously rendered tree” and the “next tree” entirely in C++. The result is a list of atomic mutation operations to be performed on host views (e.g. `createView`, `updateView`, `removeView`, `deleteView`, etc). This step is also where the React Shadow Tree is flattened to avoid creating unnecessary host views. See [View Flattening](view-flattening) for details about this algorithm.
- **Tree Promotion (Next Tree → Rendered Tree)**: This step atomically promotes the “next tree” to “previously rendered tree” so that the next mount phase computes a diff against the proper tree.
- **View Mounting**: This step applies the atomic mutation operations onto corresponding host views. This step executes in the _host platform_ on UI thread.

**補充細節**

- The operations are synchronously executed on UI thread. If the commit phase executes on background thread, the mounting phase is scheduled for the next “tick” of UI thread. On the other hand, if the commit phase executes on UI thread, mounting phase executes synchronously on the same thread.
- Scheduling, implementation, and execution of the mounting phase heavily depends on the _host platform_. For example, the renderer architecture of the mounting layer currently differs between Android and iOS.
- During the initial render, the “previously rendered tree” is empty. As such, the tree diffing step will result in a list of mutation operations that consists only of creating views, setting props, and adding views to each other. Tree diffing becomes more important for performance when processing [React State Updates](#react-state-updates).
- In current production tests, a _React Shadow Tree_ typically consists of about 600-1000 _React Shadow Nodes_ (before view flattening), the trees get reduced to ~200 nodes after view flattening. On iPad or desktop apps, this quantity may increase 10-fold.

---

## React 狀態更新

Let’s explore each phase of the render pipeline when the state of a _React Element Tree_ is updated. Let’s say, you’ve rendered the following component in an initial render:

```jsx
function MyComponent() {
  return (
    <View>
      <View
        style={{backgroundColor: 'red', height: 20, width: 20}}
      />
      <View
        style={{backgroundColor: 'blue', height: 20, width: 20}}
      />
    </View>
  );
}
```

Applying what was described in the [Initial Render](#initial-render) section, you would expect the following trees to be created:

![渲染管線 4](/docs/assets/Architecture/renderer-pipeline/render-pipeline-4.png)

Notice that **Node 3** maps to a host view with a **red background**, and **Node 4** maps to a host view with a **blue background**. Assume that as the result of a state update in JavaScript product logic, the background of the first nested `<View>` changes from `'red'` to `'yellow'`. This is what the new _React Element Tree_ might look:

```jsx
<View>
  <View
    style={{backgroundColor: 'yellow', height: 20, width: 20}}
  />
  <View
    style={{backgroundColor: 'blue', height: 20, width: 20}}
  />
</View>
```

**How is this update processed by React Native?**

When a state update occurs, the renderer needs to conceptually update the _React Element Tree_ in order to update the host views that are already mounted. But in order to preserve thread safety, both the _React Element Tree_ as well as the _React Shadow Tree_ must be immutable. This means that instead of mutating the current _React Element Tree_ and _React Shadow Tree_, React must create a new copy of each tree which incorporates the new props, styles, and children.

讓我們分析狀態更新期間渲染管線的各階段。

### 階段 1. 渲染

![階段一：渲染](/docs/assets/Architecture/renderer-pipeline/phase-one-render.png)

When React creates a new _React Element Tree_ that incorporates the new state, it must clone every _React Element_ and _React Shadow Node_ that is impacted by the change. After cloning, the new _React Shadow Tree_ is committed.

React Native renderer leverages structural sharing to minimize the overhead of immutability. When a _React Element_ is cloned to include the new state, every _React Element_ that is on the path up to the root is cloned. **React will only clone a React Element if it requires an update to its props, style, or children.** Any _React Elements_ that are unchanged by the state update are shared by the old and new trees.

在上述範例中，React 透過以下操作建立新樹：

1. CloneNode(**Node 3**, `{backgroundColor: 'yellow'}`) → **Node 3'**
2. CloneNode(**Node 2**) → **Node 2'**
3. AppendChild(**Node 2'**, **Node 3'**)
4. AppendChild(**Node 2'**, **Node 4**)
5. CloneNode(**Node 1**) → **Node 1'**
6. AppendChild(**Node 1'**, **Node 2'**)

完成這些操作後，**節點 1'** 代表新的 _React 元素樹_ 的根節點。讓我們將 **T** 指定為「先前渲染的樹」，而 **T'** 指定為「新樹」：

![渲染管線 5](/docs/assets/Architecture/renderer-pipeline/render-pipeline-5.png)

請注意 **T** 和 **T'** 如何共享 **節點 4**。結構共享提升了效能並降低了記憶體使用量。

### 階段 2. 提交

![階段二：提交](/docs/assets/Architecture/renderer-pipeline/phase-two-commit.png)

在 React 建立新的 _React 元素樹_ 和 _React 影子樹_ 後，必須提交它們。

- **佈局計算：** 類似於[初始渲染](#initial-render)期間的佈局計算。一個重要差異是佈局計算可能導致共享的 _React 影子節點_ 被克隆。這可能發生在如果共享 _React 影子節點_ 的父節點發生佈局變更時，共享 _React 影子節點_ 的佈局也可能隨之改變。
- **樹晉升（新樹 → 下一棵樹）：** 類似於[初始渲染](#initial-render)期間的樹晉升。

### 階段 3. 掛載

![階段三：掛載](/docs/assets/Architecture/renderer-pipeline/phase-three-mount.png)

- **Tree Promotion (Next Tree → Rendered Tree)**: This step atomically promotes the “next tree” to “previously rendered tree” so that the next mount phase computes a diff against the proper tree.
- **Tree Diffing:** This step computes the diff between the “previously rendered tree” (**T**) and the “next tree” (**T'**). The result is a list of atomic mutation operations to be performed on _host views_.
  - In the above example, the operations consist of: `UpdateView(**Node 3**, {backgroundColor: 'yellow'})`
  - The diff can be calculated for any currently mounted tree with any new tree. The renderer can skip some intermediate versions of the tree.
- **View Mounting**: This step applies the atomic mutation operations onto corresponding _host views_. In the above example, only the `backgroundColor` of **View 3** will be updated (to yellow).

![渲染管線 6](/docs/assets/Architecture/renderer-pipeline/render-pipeline-6.png)

---

## React Native 渲染器狀態更新

對於 _影子樹_ 中的大多數資訊，React 是唯一的擁有者和單一真相來源。所有資料都源自 React，並且資料流是單向的。

然而，有一個例外且重要的機制：C++ 中的元件可以包含不直接暴露給 JavaScript 的狀態，且 JavaScript 不是真相來源。C++ 和 _宿主平台_ 控制這個 _C++ 狀態_。通常，這僅在開發需要 _C++ 狀態_ 的複雜 _宿主元件_ 時才相關。絕大多數的 _宿主元件_ 不需要此功能。

例如，`ScrollView` 使用此機制讓渲染器知道當前的偏移量。更新由 _宿主平台_ 觸發，具體來自代表 `ScrollView` 元件的宿主視圖。偏移量資訊用於像 [measure](https://reactnative.dev/docs/direct-manipulation#measurecallback) 這樣的 API 中。由於此更新源自宿主平台，且不影響 React 元素樹，此狀態資料由 _C++ 狀態_ 持有。

概念上，_C++ 狀態_ 更新與上述的 [React 狀態更新](render-pipeline#react-state-updates) 類似。
但有兩個重要差異：

1. 它們跳過「渲染階段」，因為 React 不參與其中。
2. 更新可以源自並發生在任何執行緒上，包括主執行緒。

### 階段 2. 提交

![階段二：提交](/docs/assets/Architecture/renderer-pipeline/phase-two-commit.png)

When performing a _C++ State_ update, a block of code requests an update of a `ShadowNode` (**N**) to set _C++ State_ to value **S**. React Native renderer will repeatedly attempt to get the latest committed version of **N**, clone it with a new state **S**, and commit **N’** to the tree. If React, or another _C++ State_ update, has performed another commit during this time, the _C++ State_ commit will fail and the renderer will retry the _C++ State_ update many times until a commit succeeds. This prevents source-of-truth collisions and races.

### 階段 3. 掛載

![第三階段：掛載](/docs/assets/Architecture/renderer-pipeline/phase-three-mount.png)

The _Mount Phase_ is practically identical to the [Mount Phase of React State Updates](#react-state-updates). The renderer still needs to recompute layout, perform a tree diff, etc. See the sections above for details.