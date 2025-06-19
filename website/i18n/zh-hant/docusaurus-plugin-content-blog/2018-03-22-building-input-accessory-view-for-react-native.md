---
title: 'Building <InputAccessoryView> For React Native'
author: Peter Argany
authorTitle: Software Engineer at Facebook
authorURL: 'https://github.com/PeteTheHeat'
authorImageURL: 'https://avatars3.githubusercontent.com/u/6011080?s=400&u=028e28081107d0ab16a5cb22baca43c080f5fa50&v=4'
authorTwitter: peterargany
tags: [engineering]
---

## 開發動機

三年前，GitHub 上有人提出了一個 issue，希望 React Native 能支援輸入輔助視圖（input accessory view）。

<img src="/blog/assets/input-accessory-1.png" />

在接下來的幾年裡，這個 issue 累積了無數的「+1」、各種變通方案，但 React Native 本身卻始終沒有實質改變——直到今天。我們從 iOS 開始，[提供了一個 API](/docs/inputaccessoryview) 來存取原生的輸入輔助視圖，並很興奮地想分享我們是如何實現它的。

## 背景知識

究竟什麼是輸入輔助視圖？閱讀 [Apple 的開發者文件](https://developer.apple.com/documentation/uikit/uiresponder/1621119-inputaccessoryview?language=objc)後，我們了解到這是一個自訂視圖，可以在接收者成為第一響應者（first responder）時，固定在系統鍵盤的頂部。任何繼承自 `UIResponder` 的類別都可以將 `.inputAccessoryView` 屬性重新宣告為可讀寫，並在此管理一個自訂視圖。響應者基礎設施會掛載這個視圖，並讓它與系統鍵盤保持同步。像是拖曳或點擊等關閉鍵盤的手勢，會在框架層級應用到輸入輔助視圖上。這讓我們能打造出具有互動式鍵盤關閉功能的內容，這是像 iMessage 和 WhatsApp 這類頂級通訊應用程式的核心功能。

將視圖固定在鍵盤頂部有兩種常見的使用情境。第一種是建立鍵盤工具列，像是 Facebook 發文編輯器的背景選擇器。

<img src="/blog/assets/input-accessory-2.gif" style={{float: 'left', paddingRight: 70, paddingTop: 20}} />

在這個情境中，鍵盤聚焦在一個文字輸入欄位，而輸入輔助視圖則用來提供額外的鍵盤功能。這些功能會根據輸入欄位的類型而有所不同。在地圖應用程式中，可能是地址建議；在文字編輯器中，則可能是富文本格式工具。

<hr style={{clear: 'both', marginBottom: 20}} />

在這個情境中，擁有 `<InputAccessoryView>` 的 Objective-C UIResponder 應該很明確。`<TextInput>` 成為了第一響應者，在底層它會變成 `UITextView` 或 `UITextField` 的實例。

第二種常見情境是固定式文字輸入：

<img src="/blog/assets/input-accessory-3.gif" style={{float: 'left', paddingRight: 70, paddingTop: 20}} />

在這裡，文字輸入實際上是輸入輔助視圖本身的一部分。這通常用於通訊應用程式中，讓使用者在瀏覽過往訊息的同時，也能撰寫新訊息。

<hr style={{clear: 'both', marginBottom: 20}} />

Who owns the `<InputAccessoryView>` in this example? Can it be the `UITextView` or `UITextField` again? The text input is _inside_ the input accessory view, this sounds like a circular dependency. Solving this issue alone is [another blog post](https://derpturkey.com/uitextfield-docked-like-ios-messenger/) in itself. Spoilers: the owner is a generic `UIView` subclass who we manually tell to [becomeFirstResponder](https://developer.apple.com/documentation/uikit/uiresponder/1621113-becomefirstresponder?language=objc).

## API 設計

現在我們知道 `<InputAccessoryView>` 是什麼，以及我們想如何使用它。下一步是設計一個 API，能同時滿足兩種使用情境，並與現有的 React Native 元件（如 `<TextInput>`）良好配合。

針對鍵盤工具列，我們有幾點需要考慮：

1. 我們希望能將任何通用的 React Native 視圖層級提升到 `<InputAccessoryView>` 中。
2. 我們希望這個通用且獨立的視圖層級能接收觸控，並能操作應用程式狀態。
3. 我們希望將 `<InputAccessoryView>` 連結到特定的 `<TextInput>`。
4. 我們希望能跨多個文字輸入共享同一個 `<InputAccessoryView>`，而無需重複任何程式碼。

我們可以使用類似 [React portals](https://reactjs.org/docs/portals.html) 的概念來實現第 1 點。在這個設計中，我們將 React Native 視圖傳送到由響應者基礎設施管理的 `UIView` 層級中。由於 React Native 視圖會渲染為 UIViews，這其實相當直觀——我們只需要覆寫：

`- (void)insertReactSubview:(UIView *)subview atIndex:(NSInteger)atIndex`

and pipe all the subviews to a new UIView hierarchy. For #2, we set up a new [RCTTouchHandler](https://github.com/facebook/react-native/blob/master/React/Base/RCTTouchHandler.h) for the `<InputAccessoryView>`. State updates are achieved by using regular event callbacks. For #3 and #4, we use the [nativeID](https://github.com/facebook/react-native/blob/master/React/Views/UIView%2BReact.h#L28) field to locate the accessory view UIView hierarchy in native code during the creation of a `<TextInput>` component. This function uses the `.inputAccessoryView` property of the underlying native text input. Doing this effectively links `<InputAccessoryView>` to `<TextInput>` in their ObjC implementations.

Supporting sticky text inputs (scenario 2) adds a few more constraints. For this design, the input accessory view has a text input as a child, so linking via nativeID is not an option. Instead, we set the `.inputAccessoryView` of a generic off-screen `UIView` to our native `<InputAccessoryView>` hierarchy. By manually telling this generic `UIView` to become first responder, the hierarchy is mounted by responder infrastructure. This concept is explained thoroughly in the aforementioned blog post.

## 陷阱

當然，在構建這個 API 的過程中並非一帆風順。以下是我們遇到的一些陷阱，以及我們如何解決它們。

構建這個 API 的初始想法涉及監聽 `NSNotificationCenter` 的 UIKeyboardWill(Show/Hide/ChangeFrame) 事件。這種模式在一些開源庫中使用，也在 Facebook 應用的一些內部部分中使用。不幸的是，`UIKeyboardDidChangeFrame` 事件沒有及時被調用以更新 `<InputAccessoryView>` 的框架在滑動時。此外，鍵盤高度的變化沒有被這些事件捕獲。這導致了一類錯誤，表現如下：

<img src="/blog/assets/input-accessory-4.gif" style={{float: 'left', paddingRight: 70, paddingTop: 20}} />

在 iPhone X 上，文本和表情鍵盤的高度不同。大多數使用鍵盤事件來操作文本輸入框架的應用程序都必須修復上述錯誤。我們的解決方案是承諾使用 `.inputAccessoryView` 屬性，這意味著響應者基礎設施會處理這樣的框架更新。

<hr style={{clear: 'both', marginBottom: 20}} />

我們遇到的另一個棘手的錯誤是避免 iPhone X 上的主頁指示條。你可能會想，“Apple 開發了 [safeAreaLayoutGuide](https://developer.apple.com/documentation/uikit/uiview/2891102-safearealayoutguide?language=objc) 正是為此，這很簡單！”。我們也曾如此天真。第一個問題是原生的 `<InputAccessoryView>` 實現在即將出現之前沒有窗口可以錨定。這沒關係，我們可以覆蓋 `-(BOOL)becomeFirstResponder` 並在那裡強制執行佈局約束。遵守這些約束會將輔助視圖向上推，但另一個錯誤出現了：<img src="/blog/assets/input-accessory-5.gif" style={{float: 'left', paddingRight: 70, paddingTop: 20}} />

輸入輔助視圖成功避開了主頁指示條，但現在不安全區域後面的內容可見了。解決方案在於這個 [radar](https://www.openradar.me/34411433)。我將原生的 `<InputAccessoryView>` 層次結構包裹在一個不遵守 `safeAreaLayoutGuide` 約束的容器中。原生容器覆蓋了不安全區域的內容，而 `<InputAccessoryView>` 保持在安全區域的邊界內。

<hr style={{clear: 'both', marginBottom: 20}} />

## 示例用法

這裡有一個構建鍵盤工具欄按鈕來重置 `<TextInput>` 狀態的示例。

```jsx
class TextInputAccessoryViewExample extends React.Component<
  {},
  *,
> {
  constructor(props) {
    super(props);
    this.state = {text: 'Placeholder Text'};
  }

  render() {
    const inputAccessoryViewID = 'inputAccessoryView1';
    return (
      <View>
        <TextInput
          style={styles.default}
          inputAccessoryViewID={inputAccessoryViewID}
          onChangeText={text => this.setState({text})}
          value={this.state.text}
        />
        <InputAccessoryView nativeID={inputAccessoryViewID}>
          <View style={{backgroundColor: 'white'}}>
            <Button
              onPress={() =>
                this.setState({text: 'Placeholder Text'})
              }
              title="Reset Text"
            />
          </View>
        </InputAccessoryView>
      </View>
    );
  }
}
```

另一個 [粘性文本輸入的示例可以在存儲庫中找到](https://github.com/facebook/react-native/blob/84ef7bc372ad870127b3e1fb8c13399fe09ecd4d/RNTester/js/InputAccessoryViewExample.js)。

## 我什麼時候可以使用這個？

此功能的完整提交記錄可於[此處](https://github.com/facebook/react-native/commit/38197c8230657d567170cdaf8ff4bbb4aee732b8)查看。[`<InputAccessoryView>`](/docs/next/inputaccessoryview)將在即將發布的v0.55.0版本中提供。

祝您鍵盤操作愉快 :)