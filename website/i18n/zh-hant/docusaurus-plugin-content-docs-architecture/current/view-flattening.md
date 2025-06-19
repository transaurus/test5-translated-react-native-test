---
id: view-flattening
title: View Flattening
---

import FabricWarning from './\_fabric-warning.mdx';

<FabricWarning />

#### React Native 渲染器透過視圖扁平化優化來避免深層佈局樹結構。

The React API is designed to be declarative and reusable through composition. This provides a great model for intuitive development. However, in implementation, these qualities of the API lead to the creation of deep [React Element Trees](architecture-glossary.md#react-element-tree-and-react-element), where a large majority of React Element Nodes only affect the layout of a View and don’t render anything on the screen. We call these types of nodes **“Layout-Only”** Nodes.

理論上，React 元素樹的每個節點與螢幕視圖存在 1:1 對應關係，因此當深層 React 元素樹包含大量「僅佈局」節點時，會導致渲染效能下降。

以下是常見受「僅佈局」視圖成本影響的使用案例：

假設您需要渲染一個圖片與由 `TitleComponent` 處理的標題，並將此組件作為具有邊距樣式的 `ContainerComponent` 子項。分解組件後，React 代碼如下所示：

```jsx
function MyComponent() {
  return (
    <View>                          // ReactAppComponent
      <View style={{margin: 10}} /> // ContainerComponent
        <View style={{margin: 10}}> // TitleComponent
          <Image {...} />
          <Text {...}>This is a title</Text>
        </View>
      </View>
    </View>
  );
}
```

在渲染過程中，React Native 會產生以下樹狀結構：

![圖表一](/docs/assets/Architecture/view-flattening/diagram-one.png)

Note that the Views (2) and (3) are “Layout Only” views, because they are rendered on the screen but they only render a `margin` of `10 px` on top of their children.

為提升此類 React 元素樹的效能，渲染器實作了視圖扁平化機制，透過合併或壓平這類節點來減少螢幕上渲染的 [宿主視圖](architecture-glossary.md#host-view-tree-and-host-view) 層級深度。此演算法會綜合考量如 `margin`、`padding`、`backgroundColor`、`opacity` 等屬性。

視圖扁平化演算法被設計整合為渲染器差異比對階段的一部分，這意味著我們無需消耗額外 CPU 週期來優化 React 元素樹的扁平化處理。與核心模組相同，視圖扁平化演算法以 C++ 實作，其優化效益預設適用於所有支援平台。

在前述範例中，視圖 (2) 和 (3) 將在「差異比對演算法」過程中被扁平化，其樣式最終會合併至視圖 (1)：

![圖表二](/docs/assets/Architecture/view-flattening/diagram-two.png)

需特別說明的是，此優化使渲染器能避免創建與渲染兩個宿主視圖。從使用者角度觀察，螢幕顯示效果不會有任何視覺變化。