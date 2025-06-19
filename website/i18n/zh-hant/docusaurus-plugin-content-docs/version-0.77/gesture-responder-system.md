---
id: gesture-responder-system
title: Gesture Responder System
---

手勢響應系統負責管理應用中手勢的生命週期。當應用判斷用戶意圖時，一次觸碰可能會經歷多個階段。例如，應用需要判斷觸碰是滾動、滑動元件還是點擊。甚至在觸碰持續期間，這個判斷也可能改變。同時還可能存在多點觸碰的情況。

觸碰響應系統的設計目的，是讓元件能在無需了解父元件或子元件的情況下，協調這些觸碰互動行為。

### 最佳實踐

為了讓應用體驗更佳，每個操作都應具備以下特性：

- 反饋/高亮效果 - 向用戶展示當前處理觸碰的元件，以及釋放手勢後將觸發的動作
- 可取消性 - 用戶在執行操作時，應能透過拖移手指中途中斷觸碰

這些特性能提升用戶使用應用的舒適度，因為它允許人們在無需擔心操作失誤的情況下進行實驗性互動。

### TouchableHighlight 與 Touchable* 元件

響應系統可能較複雜難用。因此我們為需要「可點擊」的元件提供了抽象的 `Touchable` 實現。它利用響應系統，讓您能以宣告式配置點擊互動。任何在網頁上會使用按鈕或連結的地方，都可改用 `TouchableHighlight`。

## 響應者生命週期

視圖可透過實作正確的協商方法成為觸碰響應者。有兩種方法可詢問視圖是否願意成為響應者：

- `View.props.onStartShouldSetResponder: evt => true,` - 此視圖是否希望在觸碰開始時成為響應者？
- `View.props.onMoveShouldSetResponder: evt => true,` - 當視圖非響應者時，每次觸碰移動都會呼叫：此視圖是否想「聲明」觸碰響應權？

若視圖返回 true 並嘗試成為響應者，將會發生以下其中一種情況：

- `View.props.onResponderGrant: evt => {}` - 視圖現在開始響應觸碰事件。此時應顯示高亮效果讓用戶了解當前狀態
- `View.props.onResponderReject: evt => {}` - 目前已有其他元件擔任響應者且不願釋放權限

若視圖正在響應，則可能呼叫以下處理程序：

- `View.props.onResponderMove: evt => {}` - 用戶正在移動手指
- `View.props.onResponderRelease: evt => {}` - 觸碰結束時觸發（即「touchUp」）
- `View.props.onResponderTerminationRequest: evt => true` - 其他元件要求成為響應者。此視圖是否應釋放響應權？返回 true 表示允許釋放
- `View.props.onResponderTerminate: evt => {}` - 響應權已從視圖剝離。可能因 `onResponderTerminationRequest` 呼叫後被其他視圖取得，也可能被系統未經詢問直接剝離（iOS 上的控制中心/通知中心會發生此情況）

`evt` 是合成觸碰事件，其結構如下：

- `nativeEvent`
  - `changedTouches` - 自上次事件後所有變更的觸碰事件陣列
  - `identifier` - 觸碰識別碼
  - `locationX` - 觸碰的 X 座標（相對於元件）
  - `locationY` - 觸碰的 Y 座標（相對於元件）
  - `pageX` - 觸碰的 X 座標（相對於根元件）
  - `pageY` - 觸碰的 Y 座標（相對於根元件）
  - `target` - 接收觸碰事件的元件節點 ID
  - `timestamp` - 觸碰時間標記（用於計算速度）
  - `touches` - 當前螢幕上所有觸碰點的陣列

### 捕獲 ShouldSet 處理程序

`onStartShouldSetResponder` 和 `onMoveShouldSetResponder` 會以冒泡模式被調用，最深的節點會優先被觸發。這意味著當多個視圖對 `*ShouldSetResponder` 處理程序返回 true 時，最內層的組件將成為響應者。在大多數情況下這是理想的，因為它能確保所有控件和按鈕都可使用。

然而，有時父組件會希望確保自己成為響應者。這可以通過使用捕獲階段來處理。在響應系統從最深組件冒泡之前，它會先執行捕獲階段，觸發 `on*ShouldSetResponderCapture`。因此，如果父視圖希望阻止子組件在觸摸開始時成為響應者，它應該設置一個返回 true 的 `onStartShouldSetResponderCapture` 處理程序。

- `View.props.onStartShouldSetResponderCapture: evt => true,`
- `View.props.onMoveShouldSetResponderCapture: evt => true,`

### PanResponder

如需更高層級的手勢解析，請參閱 [PanResponder](panresponder.md)。