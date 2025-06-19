---
title: Better List Views in React Native
author: Spencer Ahrens
authorTitle: Software Engineer at Facebook
authorURL: 'https://github.com/sahrens'
authorImageURL: 'https://avatars1.githubusercontent.com/u/1509831'
authorTwitter: sahrens2012
tags: [engineering]
---

許多開發者已在[社群群組的預告貼文](https://www.facebook.com/groups/react.native.community/permalink/921378591331053)後開始試用我們的新列表元件，而今天我們正式宣布它們的到來！不再需要`ListView`或`DataSource`，不再有陳舊的行、被忽略的錯誤或過高的記憶體消耗——透過最新的React Native 2017年3月候選版本（`0.43-rc.1`），您可以從這套新元件中選擇最適合您需求的組件，它們具備出色的性能和豐富的功能：

### [`<FlatList>`](/docs/flatlist)

這是處理簡單高效列表的主力元件。只需提供一個數據陣列和一個`renderItem`函數即可開始使用：

```
<FlatList
  data={[{title: 'Title Text', key: 'item1'}, ...]}
  renderItem={({item}) => <ListItem title={item.title} />}
/>
```

### [`<SectionList>`](/docs/sectionlist)

如果您需要渲染分為邏輯區段的數據集（例如按字母排序的通訊錄），可能包含區段標頭，並且數據類型和渲染方式可能不同（例如個人資料視圖，包含一些按鈕、一個編輯器、一個照片網格、一個朋友網格，最後是一個故事列表），那麼這個元件就是您的選擇。

```
<SectionList
  renderItem={({item}) => <ListItem title={item.title} />}
  renderSectionHeader={({section}) => <H1 title={section.key} />}
  sections={[ // homogeneous rendering between sections
    {data: [...], key: ...},
    {data: [...], key: ...},
    {data: [...], key: ...},
  ]}
/>

<SectionList
  sections={[ // heterogeneous rendering between sections
    {data: [...], key: ..., renderItem: ...},
    {data: [...], key: ..., renderItem: ...},
    {data: [...], key: ..., renderItem: ...},
  ]}
/>
```

### [`<VirtualizedList>`](/docs/virtualizedlist)

這是背後實現的底層元件，提供更靈活的API。特別適合數據不是簡單陣列的情況（例如不可變列表）。

## 功能

列表在許多情境下使用，因此我們為新元件加入了大量功能，以滿足大多數使用場景的需求：

- Scroll loading (`onEndReached`).
- Pull to refresh (`onRefresh` / `refreshing`).
- [Configurable](https://github.com/facebook/react-native/blob/master/Libraries/CustomComponents/Lists/ViewabilityHelper.js) viewability (VPV) callbacks (`onViewableItemsChanged` / `viewabilityConfig`).
- Horizontal mode (`horizontal`).
- Intelligent item and section separators.
- Multi-column support (`numColumns`)
- `scrollToEnd`, `scrollToIndex`, and `scrollToItem`
- Better Flow typing.

### 注意事項

- The internal state of item subtrees is not preserved when content scrolls out of the render window. Make sure all your data is captured in the item data or external stores like Flux, Redux, or Relay.

- These components are based on `PureComponent` which means that they will not re-render if `props` remains shallow-equal. Make sure that everything your `renderItem` function depends on directly is passed as a prop that is not `===` after updates, otherwise your UI may not update on changes. This includes the `data` prop and parent component state. For example:

  ```jsx
  <FlatList
    data={this.state.data}
    renderItem={({item}) => (
      <MyItem
        item={item}
        onPress={() =>
          this.setState(oldState => ({
            selected: {
              // New instance breaks `===`
              ...oldState.selected, // copy old data
              [item.key]: !oldState.selected[item.key], // toggle
            },
          }))
        }
        selected={
          !!this.state.selected[item.key] // renderItem depends on state
        }
      />
    )}
    selected={
      // Can be any prop that doesn't collide with existing props
      this.state.selected // A change to selected should re-render FlatList
    }
  />
  ```

- In order to constrain memory and enable smooth scrolling, content is rendered asynchronously offscreen. This means it's possible to scroll faster than the fill rate and momentarily see blank content. This is a tradeoff that can be adjusted to suit the needs of each application, and we are working on improving it behind the scenes.

- By default, these new lists look for a `key` prop on each item and use that for the React key. Alternatively, you can provide a custom `keyExtractor` prop.

## 性能

Besides simplifying the API, the new list components also have significant performance enhancements, the main one being nearly constant memory usage for any number of rows. This is done by 'virtualizing' elements that are outside of the render window by completely unmounting them from the component hierarchy and reclaiming the JS memory from the react components, along with the native memory from the shadow tree and the UI views. This has a catch which is that internal component state will not be preserved, so **make sure you track any important state outside of the components themselves, e.g. in Relay or Redux or Flux store.**

限制渲染視窗也減少了React和原生平台需要處理的工作量，例如視圖遍歷。即使您正在渲染百萬元素中的最後一個，使用這些新列表也無需迭代所有元素即可完成渲染。您甚至可以直接跳轉到中間位置（透過`scrollToIndex`）而無需過度渲染。

我們還對調度機制進行了改進，這有助於提升應用程式的響應速度。渲染視窗邊緣的項目會以較低優先級進行不頻繁的渲染，且會在當前手勢操作、動畫或其他互動完成後才執行。

## 進階用法

與`ListView`不同，渲染視窗內的所有項目會在props發生任何變化時重新渲染。由於視窗化技術已將項目數量限制為固定值，這通常不會造成問題。但如果您的項目結構較複雜，應遵循React效能最佳實踐，在元件內部適當使用`React.PureComponent`和/或`shouldComponentUpdate`來限制遞迴子樹的重新渲染。

若能在不實際渲染的情況下計算出行高，可通過提供`getItemLayout`屬性來改善使用者體驗。這使得滾動到特定項目（例如使用`scrollToIndex`）更加流暢，同時由於無需渲染即可確定內容高度，滾動指示器UI也會得到改善。

如果您使用替代數據類型（如不可變列表），`<VirtualizedList>`是最佳選擇。它接受`getItem`屬性，可返回任意索引對應的項目數據，且具有更寬鬆的Flow類型檢查。

對於特殊用例，還有一系列參數可供調整。例如：使用`windowSize`權衡記憶體用量與使用者體驗、透過`maxToRenderPerBatch`調整填充率與響應速度、用`onEndReachedThreshold`控制滾動載入觸發時機等。

## 未來工作

- 現有介面的遷移（最終棄用`ListView`）
- 根據需求新增更多功能（歡迎反饋！）
- 支援黏性區段標頭
- 更多效能優化
- 支援帶狀態的函數式項目元件