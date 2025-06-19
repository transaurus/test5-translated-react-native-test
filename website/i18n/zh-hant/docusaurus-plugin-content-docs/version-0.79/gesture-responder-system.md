---
id: gesture-responder-system
title: Gesture Responder System
---

手勢響應系統負責管理應用中手勢的生命週期。當應用判斷用戶意圖時，一次觸碰可能會經歷多個階段。例如，應用需要判斷該觸碰是滾動、滑動元件還是點擊。甚至在觸碰持續期間，這個判斷也可能改變。同時也可能存在多個並發的觸碰。

觸碰響應系統的存在，是為了讓元件能在無需了解其父元件或子元件的情況下，協調這些觸碰互動。

### 最佳實踐

為了讓應用體驗更佳，每個操作都應具備以下特性：

- 反饋/高亮效果 - 向用戶展示當前正在處理觸碰的元件，以及鬆開手勢後會發生的動作
- 可取消性 - 當執行操作時，用戶應能透過將手指拖曳開來中途取消該操作

這些特性讓用戶在使用應用時更為舒適，因為它允許人們在無需擔心犯錯的情況下進行實驗和互動。

### TouchableHighlight 與 Touchable\*

響應系統可能較為複雜難用。因此我們為需要「可點擊」的元件提供了抽象的 `Touchable` 實現。它利用響應系統，並允許你以宣告式的方式配置點擊互動。在任何需要使用網頁按鈕或連結的地方，都可以使用 `TouchableHighlight`。

## 響應者生命週期

視圖可透過實作正確的協商方法來成為觸碰響應者。有兩種方法可詢問視圖是否想成為響應者：

- `View.props.onStartShouldSetResponder: evt => true,` - 此視圖是否想在觸碰開始時成為響應者？
- `View.props.onMoveShouldSetResponder: evt => true,` - 當視圖非響應者時，每次觸碰移動都會呼叫：此視圖是否想「聲明」觸碰響應權？

若視圖返回 true 並嘗試成為響應者，將會發生以下其中一種情況：

- `View.props.onResponderGrant: evt => {}` - 視圖現在正響應觸碰事件。此時應高亮並向用戶展示正在發生的狀況
- `View.props.onResponderReject: evt => {}` - 目前有其他元件正擔任響應者且不願釋放權限

若視圖正在響應，則可能會呼叫以下處理函數：

- `View.props.onResponderMove: evt => {}` - 用戶正在移動手指
- `View.props.onResponderRelease: evt => {}` - 觸碰結束時觸發，即「touchUp」
- `View.props.onResponderTerminationRequest: evt => true` - 其他元件想成為響應者。此視圖是否應釋放響應權？返回 true 表示允許釋放
- `View.props.onResponderTerminate: evt => {}` - 響應權已從視圖取走。可能是在呼叫 `onResponderTerminationRequest` 後被其他視圖取走，也可能是在未經詢問的情況下被作業系統取走（iOS 上的控制中心/通知中心會發生此情況）

`evt` 是一個合成觸碰事件，其結構如下：

- `nativeEvent`
  - `changedTouches` - 自上次事件以來所有已變更觸碰事件的陣列
  - `identifier` - 觸碰的識別碼
  - `locationX` - 觸碰的 X 座標，相對於元件
  - `locationY` - 觸碰的 Y 座標，相對於元件
  - `pageX` - 觸碰的 X 座標，相對於根元件
  - `pageY` - 觸碰的 Y 座標，相對於根元件
  - `target` - 接收觸碰事件的元件節點識別碼
  - `timestamp` - 觸碰的時間識別碼，可用於計算速度
  - `touches` - 螢幕上所有當前觸碰的陣列

### 捕獲 ShouldSet 處理函數

`onStartShouldSetResponder` 和 `onMoveShouldSetResponder` 會以冒泡模式呼叫，最深的節點會優先被觸發。這意味著當多個視圖對 `*ShouldSetResponder` 處理程序返回 true 時，最深的元件將成為響應者。在大多數情況下這是理想的，因為它能確保所有控制項和按鈕都可使用。

然而，有時父元件會希望確保自己成為響應者。這可以通過使用捕獲階段來處理。在響應系統從最深元件冒泡之前，它會先執行捕獲階段，觸發 `on*ShouldSetResponderCapture`。因此，如果父視圖希望阻止子元件在觸摸開始時成為響應者，它應該設置一個返回 true 的 `onStartShouldSetResponderCapture` 處理程序。

- `View.props.onStartShouldSetResponderCapture: evt => true,`
- `View.props.onMoveShouldSetResponderCapture: evt => true,`

### PanResponder

如需更高層級的手勢解譯，請參閱 [PanResponder](panresponder.md)。