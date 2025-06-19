---
id: gesture-responder-system
title: Gesture Responder System
---

手勢響應系統負責管理應用程式中的手勢生命週期。一個觸控操作可能會經歷多個階段，應用程式需要判斷使用者的意圖。例如，判斷觸控是滾動、滑動元件還是點擊。這些判斷甚至可能在觸控過程中改變。同時也可能存在多個並行的觸控操作。

觸控響應系統讓元件能夠協調這些觸控互動，而無需了解其父元件或子元件的額外資訊。

### 最佳實踐

為了提升應用體驗，每個操作都應具備以下特性：

- 反饋/高亮效果 - 向使用者顯示正在處理觸控的元件，以及釋放手勢後會發生的結果
- 可取消性 - 執行操作時，使用者應能通過拖移手指來中途取消操作

這些特性讓使用者能更自在地操作應用，因為允許人們嘗試互動而不必擔心操作失誤。

### TouchableHighlight 與 Touchable* 元件

響應系統可能較複雜。因此我們提供了抽象的 `Touchable` 實現，用於需要「可點擊」的場景。它利用響應系統並允許以宣告式配置點擊互動。任何需要按鈕或連結的場景都可使用 `TouchableHighlight`，類似網頁中的用法。

## 響應者生命週期

視圖可通過實現正確的協商方法成為觸控響應者。有兩種方法詢問視圖是否願意成為響應者：

- `View.props.onStartShouldSetResponder: evt => true,` - 此視圖是否希望在觸控開始時成為響應者？
- `View.props.onMoveShouldSetResponder: evt => true,` - 當視圖非響應者時，每次觸控移動都會呼叫：此視圖是否想「聲明」觸控響應權？

若視圖返回 true 並嘗試成為響應者，將發生以下情況之一：

- `View.props.onResponderGrant: evt => {}` - 視圖現在負責處理觸控事件。此時應顯示高亮效果，向使用者反饋操作狀態
- `View.props.onResponderReject: evt => {}` - 當前已有其他響應者且不願釋放權限

若視圖正在響應，可能會呼叫以下處理程序：

- `View.props.onResponderMove: evt => {}` - 使用者正在移動手指
- `View.props.onResponderRelease: evt => {}` - 觸控結束時觸發，即「touchUp」
- `View.props.onResponderTerminationRequest: evt => true` - 其他元件請求成為響應者。此視圖是否應釋放權限？返回 true 表示允許釋放
- `View.props.onResponderTerminate: evt => {}` - 響應權限被剝奪。可能因 `onResponderTerminationRequest` 呼叫後被其他視圖取得，也可能被系統直接奪權（如 iOS 控制中心/通知中心）

`evt` 是合成觸控事件，結構如下：

- `nativeEvent`
  - `changedTouches` - 自上次事件後所有變更的觸控事件陣列
  - `identifier` - 觸控操作的識別碼
  - `locationX` - 觸控點的 X 座標，相對於元件
  - `locationY` - 觸控點的 Y 座標，相對於元件
  - `pageX` - 觸控點的 X 座標，相對於根元件
  - `pageY` - 觸控點的 Y 座標，相對於根元件
  - `target` - 接收觸控事件的元件節點 ID
  - `timestamp` - 觸控時間標記，用於計算速度
  - `touches` - 當前螢幕上所有觸控點的陣列

### 捕獲 ShouldSet 處理程序

`onStartShouldSetResponder` and `onMoveShouldSetResponder` are called with a bubbling pattern, where the deepest node is called first. That means that the deepest component will become responder when multiple Views return true for `*ShouldSetResponder` handlers. This is desirable in most cases, because it makes sure all controls and buttons are usable.

However, sometimes a parent will want to make sure that it becomes responder. This can be handled by using the capture phase. Before the responder system bubbles up from the deepest component, it will do a capture phase, firing `on*ShouldSetResponderCapture`. So if a parent View wants to prevent the child from becoming responder on a touch start, it should have a `onStartShouldSetResponderCapture` handler which returns true.

- `View.props.onStartShouldSetResponderCapture: evt => true,`
- `View.props.onMoveShouldSetResponderCapture: evt => true,`

### PanResponder

如需更高層級的手勢解譯，請參閱 [PanResponder](panresponder.md)。