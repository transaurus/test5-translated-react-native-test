---
id: gesture-responder-system
title: Gesture Responder System
---

手勢響應系統負責管理應用程式中的手勢生命週期。當應用程式判斷使用者意圖時，一次觸碰可能會經歷多個階段。例如，應用程式需要判斷觸碰是捲動、滑動元件還是點擊。甚至在觸碰持續期間，這個判斷也可能改變。同時也可能存在多個觸碰點。

觸碰響應系統的設計，是為了讓元件能在不須了解其父元件或子元件的情況下，協調這些觸碰互動。

### 最佳實踐

為了讓應用程式體驗更佳，每個操作都應具備以下特性：

- 回饋/高亮效果 - 向使用者顯示當前處理觸碰的元件，以及手勢釋放後會發生的動作
- 可取消性 - 當執行操作時，使用者應能透過將手指拖曳開來中途取消該操作

這些特性讓使用者操作應用程式時更安心，因為它允許人們在不怕犯錯的情況下進行嘗試和互動。

### TouchableHighlight 與 Touchable* 元件

響應系統可能較複雜難用。因此我們為「可點擊」元素提供了抽象的 `Touchable` 實作。它利用響應系統，並允許您以宣告式配置點擊互動。任何在網頁上會使用按鈕或連結的地方，都可以使用 `TouchableHighlight`。

## 響應者生命週期

視圖可透過實作正確的協商方法來成為觸碰響應者。有兩種方法可詢問視圖是否想成為響應者：

- `View.props.onStartShouldSetResponder: evt => true,` - 此視圖是否想在觸碰開始時成為響應者？
- `View.props.onMoveShouldSetResponder: evt => true,` - 當視圖不是響應者時，每次觸碰移動都會呼叫：此視圖是否想「聲明」觸碰響應權？

若視圖返回 true 並嘗試成為響應者，將會發生以下其中一種情況：

- `View.props.onResponderGrant: evt => {}` - 視圖現在負責處理觸碰事件。此時應顯示高亮效果讓使用者了解當前狀態
- `View.props.onResponderReject: evt => {}` - 目前已有其他元件擔任響應者且不願釋放權限

若視圖正在響應，則可能呼叫以下處理程序：

- `View.props.onResponderMove: evt => {}` - 使用者正在移動手指
- `View.props.onResponderRelease: evt => {}` - 在觸碰結束時觸發，即「放開(touchUp)」
- `View.props.onResponderTerminationRequest: evt => true` - 其他元件想成為響應者。此視圖是否應釋放響應權？返回 true 表示允許釋放
- `View.props.onResponderTerminate: evt => {}` - 響應權已從視圖中取走。可能因呼叫 `onResponderTerminationRequest` 後被其他視圖取得，也可能被作業系統未經詢問直接取走（iOS 上的控制中心/通知中心會發生此情況）

`evt` 是一個合成觸碰事件，其結構如下：

- `nativeEvent`
  - `changedTouches` - 自上次事件後所有發生變化的觸碰事件陣列
  - `identifier` - 觸碰點的識別碼
  - `locationX` - 觸碰點的 X 座標，相對於元素
  - `locationY` - 觸碰點的 Y 座標，相對於元素
  - `pageX` - 觸碰點的 X 座標，相對於根元素
  - `pageY` - 觸碰點的 Y 座標，相對於根元素
  - `target` - 接收觸碰事件的元素節點 ID
  - `timestamp` - 觸碰的時間標識，用於計算速度
  - `touches` - 當前螢幕上所有觸碰點的陣列

### 捕獲 ShouldSet 處理程序

`onStartShouldSetResponder` 和 `onMoveShouldSetResponder` 會以冒泡模式呼叫，最深的節點會優先被觸發。這意味著當多個視圖對 `*ShouldSetResponder` 處理程序返回 true 時，最深的元件將成為響應者。在大多數情況下這是理想的，因為它能確保所有控制項和按鈕都可使用。

然而，有時父元件會希望確保自己成為響應者。這可以通過使用捕獲階段來處理。在響應系統從最深元件冒泡之前，它會先執行捕獲階段，觸發 `on*ShouldSetResponderCapture`。因此，如果父視圖希望阻止子元件在觸摸開始時成為響應者，它應該設置一個返回 true 的 `onStartShouldSetResponderCapture` 處理程序。

- `View.props.onStartShouldSetResponderCapture: evt => true,`
- `View.props.onMoveShouldSetResponderCapture: evt => true,`

### PanResponder

如需更高層次的手勢解譯，請參閱 [PanResponder](panresponder.md)。