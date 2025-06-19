---
id: gesture-responder-system
title: Gesture Responder System
---

手勢響應系統管理著應用中手勢的生命週期。當應用判斷用戶意圖時，一次觸摸可能會經歷多個階段。例如，應用需要判斷觸摸是滾動、滑動小部件還是點擊。這甚至在觸摸持續期間也可能發生變化。同時也可能存在多個並發觸摸。

觸摸響應系統的存在，是為了讓組件能夠在無需了解其父組件或子組件的情況下，協商這些觸摸交互。

### 最佳實踐

為了讓您的應用體驗更佳，每個操作應具備以下特性：

- 反饋/高亮顯示 - 向用戶展示當前處理觸摸的內容，以及釋放手勢後會發生的結果
- 可取消性 - 執行操作時，用戶應能夠通過拖動手指離開來中途取消操作

這些特性讓用戶在使用應用時更加舒適，因為它允許人們在無需擔心犯錯的情況下進行實驗和交互。

### TouchableHighlight 與 Touchable* 組件

響應系統可能使用起來較為複雜。因此我們為需要"可點擊"的場景提供了一個抽象的 `Touchable` 實現。它利用響應系統，並允許您以聲明式的方式配置點擊交互。在任何您會在網頁上使用按鈕或鏈接的地方，都可以使用 `TouchableHighlight`。

## 響應者生命週期

視圖可以通過實現正確的協商方法來成為觸摸響應者。有兩種方法可以詢問視圖是否希望成為響應者：

- `View.props.onStartShouldSetResponder: evt => true,` - 該視圖是否希望在觸摸開始時成為響應者？
- `View.props.onMoveShouldSetResponder: evt => true,` - 當視圖不是響應者時，每次觸摸移動都會調用：該視圖是否想"聲明"觸摸響應權？

如果視圖返回 true 並嘗試成為響應者，將會發生以下情況之一：

- `View.props.onResponderGrant: evt => {}` - 該視圖現在負責響應觸摸事件。此時應高亮顯示並向用戶展示正在發生的情況
- `View.props.onResponderReject: evt => {}` - 當前有其他響應者且不願釋放響應權

如果視圖正在響應，可能會調用以下處理程序：

- `View.props.onResponderMove: evt => {}` - 用戶正在移動手指
- `View.props.onResponderRelease: evt => {}` - 在觸摸結束時觸發，即"touchUp"
- `View.props.onResponderTerminationRequest: evt => true` - 其他對象想成為響應者。該視圖是否應該釋放響應權？返回 true 表示允許釋放
- `View.props.onResponderTerminate: evt => {}` - 響應權已從視圖中取走。可能在調用 `onResponderTerminationRequest` 後被其他視圖取走，也可能被操作系統未經詢問直接取走（iOS 上的控制中心/通知中心會發生這種情況）

`evt` 是一個合成觸摸事件，具有以下結構：

- `nativeEvent`
  - `changedTouches` - 自上次事件以來所有已變化的觸摸事件數組
  - `identifier` - 觸摸的 ID
  - `locationX` - 觸摸的 X 位置，相對於元素
  - `locationY` - 觸摸的 Y 位置，相對於元素
  - `pageX` - 觸摸的 X 位置，相對於根元素
  - `pageY` - 觸摸的 Y 位置，相對於根元素
  - `target` - 接收觸摸事件的元素的節點 ID
  - `timestamp` - 觸摸的時間標識符，用於速度計算
  - `touches` - 屏幕上所有當前觸摸的數組

### 捕獲 ShouldSet 處理程序

`onStartShouldSetResponder` 和 `onMoveShouldSetResponder` 會以冒泡模式呼叫，最深的節點會最先被呼叫。這意味著當多個 View 對 `*ShouldSetResponder` 處理程序返回 true 時，最深的元件將成為響應者。在大多數情況下這是理想的，因為它能確保所有控制項和按鈕都可使用。

然而，有時父元件會希望確保自己成為響應者。這可以通過使用捕獲階段來處理。在響應系統從最深元件冒泡之前，它會先執行捕獲階段，觸發 `on*ShouldSetResponderCapture`。因此，如果父 View 想要阻止子元件在觸摸開始時成為響應者，它應該有一個返回 true 的 `onStartShouldSetResponderCapture` 處理程序。

- `View.props.onStartShouldSetResponderCapture: evt => true,`
- `View.props.onMoveShouldSetResponderCapture: evt => true,`

### PanResponder

如需更高層級的手勢解譯，請參閱 [PanResponder](panresponder.md)。