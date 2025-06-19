---
id: gesture-responder-system
title: Gesture Responder System
---

手勢響應系統負責管理應用中手勢的生命週期。當應用判斷用戶意圖時，一次觸碰可能會經歷多個階段。例如，應用需要判斷觸碰是滾動、滑動小部件還是點擊。甚至在觸碰持續期間，這種判斷也可能改變。同時還可能存在多個並發的觸碰。

觸碰響應系統的作用是讓組件能夠協調這些觸碰交互，而無需了解其父組件或子組件的額外信息。

### 最佳實踐

為了讓應用體驗更佳，每個操作都應具備以下特性：

- 反饋/高亮顯示 - 向用戶展示當前正在處理其觸碰的內容，以及釋放手勢後會發生的結果
- 可取消性 - 執行操作時，用戶應能夠通過將手指拖離來中途取消觸碰

這些特性讓用戶在使用應用時更加舒適，因為它允許人們進行實驗和交互，而無需擔心犯錯。

### TouchableHighlight 與 Touchable* 組件

響應系統可能使用起來較為複雜。因此我們為需要"可點擊"的元素提供了抽象的 `Touchable` 實現。它利用響應系統，並允許您以聲明式配置點擊交互。在任何需要使用網頁按鈕或鏈接的地方，都可以使用 `TouchableHighlight`。

## 響應者生命週期

視圖可以通過實現正確的協商方法來成為觸碰響應者。有兩種方法可以詢問視圖是否希望成為響應者：

- `View.props.onStartShouldSetResponder: evt => true,` - 該視圖是否希望在觸碰開始時成為響應者？
- `View.props.onMoveShouldSetResponder: evt => true,` - 當視圖不是響應者時，每次觸碰移動都會調用：該視圖是否想"聲明"觸碰響應權？

如果視圖返回 true 並嘗試成為響應者，將會發生以下情況之一：

- `View.props.onResponderGrant: evt => {}` - 視圖現在負責響應觸碰事件。此時應高亮顯示並向用戶展示正在發生的情況
- `View.props.onResponderReject: evt => {}` - 當前有其他元素是響應者且不會釋放權限

如果視圖正在響應，可能會調用以下處理程序：

- `View.props.onResponderMove: evt => {}` - 用戶正在移動手指
- `View.props.onResponderRelease: evt => {}` - 在觸碰結束時觸發，即"touchUp"
- `View.props.onResponderTerminationRequest: evt => true` - 其他元素想成為響應者。該視圖是否應該釋放響應權？返回 true 允許釋放
- `View.props.onResponderTerminate: evt => {}` - 響應權已從視圖中取走。可能是在調用 `onResponderTerminationRequest` 後被其他視圖取走，也可能是在未詢問的情況下被操作系統取走（iOS 上的控制中心/通知中心會發生這種情況）

`evt` 是一個合成觸碰事件，具有以下結構：

- `nativeEvent`
  - `changedTouches` - 自上次事件以來所有已變更觸碰事件的數組
  - `identifier` - 觸碰的 ID
  - `locationX` - 觸碰的 X 位置，相對於元素
  - `locationY` - 觸碰的 Y 位置，相對於元素
  - `pageX` - 觸碰的 X 位置，相對於根元素
  - `pageY` - 觸碰的 Y 位置，相對於根元素
  - `target` - 接收觸碰事件的元素的節點 ID
  - `timestamp` - 觸碰的時間標識符，用於速度計算
  - `touches` - 屏幕上所有當前觸碰的數組

### 捕獲 ShouldSet 處理程序

`onStartShouldSetResponder` 和 `onMoveShouldSetResponder` 是以冒泡模式呼叫的，最深的節點會最先被呼叫。這意味著當多個視圖對 `*ShouldSetResponder` 處理程序返回 true 時，最深的組件將成為響應者。這在大多數情況下是理想的，因為它確保所有控件和按鈕都是可用的。

然而，有時父組件會希望確保自己成為響應者。這可以通過使用捕獲階段來處理。在響應系統從最深組件冒泡之前，它會進行一個捕獲階段，觸發 `on*ShouldSetResponderCapture`。因此，如果父視圖想要阻止子組件在觸摸開始時成為響應者，它應該有一個返回 true 的 `onStartShouldSetResponderCapture` 處理程序。

- `View.props.onStartShouldSetResponderCapture: evt => true,`
- `View.props.onMoveShouldSetResponderCapture: evt => true,`

### PanResponder

對於更高級的手勢解釋，請查看 [PanResponder](panresponder.md)。