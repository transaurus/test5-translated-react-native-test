---
title: Labeling GitHub Issues
---

Most of [our labels](https://github.com/facebook/react-native/issues/labels) have a prefix that provides a hint of their purpose.

您會立即注意到列表中有兩個主導性的標籤前綴：[API:](https://github.com/facebook/react-native/labels?utf8=%E2%9C%93&q=API%3A) 和 [Component:](https://github.com/facebook/react-native/labels?utf8=%E2%9C%93&q=Component%3A)。

這些通常表示與 React Native 核心函式庫中的 API 或元件相關的問題和拉取請求。這幫助我們一目了然地了解哪些元件急需文件或支援。

這些標籤由我們的[機器人](/contributing/bots-reference)自動添加，但如果機器人錯誤歸類了某個問題，請隨時調整它們。

- The `p:` class of labels denote a company with whom with maintain some sort of [relationship](https://github.com/facebook/react-native/blob/main/ECOSYSTEM.md). These include Microsoft and Expo, for example. These are also added automatically by our tooling, based on the issue author.
- The `DX:` class of labels denote areas that deal with the developer experience. Use these for issues that negatively impact people who use React Native.
- The `Tool:` class of labels denote tooling. CocoaPods, Buck...
- The `Resolution:` labels help us communicate the status of an issue. Does it need more information? What needs to be done before it can move forward?
- The `Type:` labels are added by a bot, based on the changelog field in a pull request. They may also refer to types of issues that are not bug reports.
- The `Platform:` labels help us identify which development platform or target OS is affected by the issue.

如果不確定某個標籤的含義，請訪問 https://github.com/facebook/react-native/labels 查看描述欄位。我們會盡力妥善記錄這些標籤。

### 標籤操作

應用以下標籤之一可能會觸發機器人互動。這些標籤的目的是在必要時提供預設回應，以協助問題分類。

- 指示機器人留下下一步操作評論的標籤：

  - `Needs: Issue Template`
  - `Needs: Environment Info`
  - `Needs: Verify on Latest Version`
  - `Needs: Repro`

- 指示機器人在留下解釋性評論後關閉問題的標籤：

  - `Resolution: For Stack Overflow`
  - `Type: Question`
  - `Type: Docs`

- 直接關閉問題且不留下評論的標籤：
  - `Type: Invalid`