---
id: gesture-responder-system
title: Gesture Responder System
---

手勢響應系統負責管理應用中手勢的生命週期。當應用判斷用戶意圖時，一次觸控會經歷多個階段。例如，應用需要判斷觸控是滾動、滑動元件還是點擊。這些判斷甚至可能在觸控持續期間發生變化，且可能同時存在多點觸控。

觸控響應系統讓元件能在無需了解父元件或子元件的情況下，協調這些觸控互動。

### 最佳實踐

要讓應用體驗流暢，每個操作都應具備以下特性：

- 反饋/高亮效果 - 向用戶顯示當前處理觸控的元件，以及釋放手勢後會發生的結果
- 可取消性 - 用戶在執行操作時，應能透過拖移手指中途中止觸控

這些特性讓用戶能更自在地使用應用，因為允許人們在互動時進行嘗試，而無需擔心操作失誤。

### TouchableHighlight 與 Touchable* 元件

響應系統的使用可能較為複雜。因此我們為「可點擊」元件提供了抽象的 `Touchable` 實現，該實現利用響應系統並允許以宣告式配置點擊互動。任何需要類似網頁按鈕或連結的地方，都可使用 `TouchableHighlight`。

## 響應生命週期

視圖可通過實現正確的協商方法成為觸控響應者。有兩種方法詢問視圖是否要成為響應者：

- `View.props.onStartShouldSetResponder: evt => true,` - 該視圖是否要在觸控開始時成為響應者？
- `View.props.onMoveShouldSetResponder: evt => true,` - 當視圖非響應者時，每次觸控移動都會呼叫：該視圖是否要「聲明」觸控響應權？

若視圖返回 true 並嘗試成為響應者，將發生以下情況之一：

- `View.props.onResponderGrant: evt => {}` - 視圖現在負責處理觸控事件。此時應顯示高亮效果讓用戶了解狀態
- `View.props.onResponderReject: evt => {}` - 當前已有其他元件擔任響應者且不願釋放權限

若視圖正在響應，可能會呼叫以下處理函數：

- `View.props.onResponderMove: evt => {}` - 用戶正在移動手指
- `View.props.onResponderRelease: evt => {}` - 觸控結束時觸發（即 "touchUp"）
- `View.props.onResponderTerminationRequest: evt => true` - 其他元件要求成為響應者。當前視圖是否應釋放權限？返回 true 表示允許釋放
- `View.props.onResponderTerminate: evt => {}` - 響應權限已從視圖剝離。可能因 `onResponderTerminationRequest` 呼叫後被其他視圖取得，也可能被系統未經詢問直接剝離（如 iOS 控制中心/通知中心觸發時）

`evt` 是合成觸控事件，結構如下：

- `nativeEvent`
  - `changedTouches` - 自上次事件後所有發生變化的觸控事件陣列
  - `identifier` - 觸控識別碼
  - `locationX` - 觸控點的 X 座標（相對於元素）
  - `locationY` - 觸控點的 Y 座標（相對於元素）
  - `pageX` - 觸控點的 X 座標（相對於根元素）
  - `pageY` - 觸控點的 Y 座標（相對於根元素）
  - `target` - 接收觸控事件的元素節點 ID
  - `timestamp` - 觸控時間標識符（用於計算速度）
  - `touches` - 當前螢幕上所有觸控點的陣列

### 捕獲 ShouldSet 處理函數

`onStartShouldSetResponder` 和 `onMoveShouldSetResponder` 會以冒泡模式呼叫，最深的節點會優先被呼叫。這意味著當多個 View 對 `*ShouldSetResponder` 處理程序返回 true 時，最深的組件將成為響應者。這在大多數情況下是理想的，因為它能確保所有控件和按鈕都可使用。

然而，有時父組件會希望確保自己成為響應者。這可以通過使用捕獲階段來處理。在響應系統從最深組件冒泡之前，它會先執行捕獲階段，觸發 `on*ShouldSetResponderCapture`。因此，如果父 View 希望阻止子組件在觸摸開始時成為響應者，它應該有一個返回 true 的 `onStartShouldSetResponderCapture` 處理程序。

- `View.props.onStartShouldSetResponderCapture: evt => true,`
- `View.props.onMoveShouldSetResponderCapture: evt => true,`

### PanResponder

如需更高級的手勢解譯，請參閱 [PanResponder](panresponder.md)。