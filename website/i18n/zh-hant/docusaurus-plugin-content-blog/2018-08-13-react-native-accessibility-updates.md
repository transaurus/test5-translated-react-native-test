---
title: Accessibility API Updates
author: Ziqi Chen
authorTitle: Student at UC Berkeley
authorURL: 'https://ziqichen.com/'
authorImageURL: 'https://avatars2.githubusercontent.com/u/13990087?s=400&u=5841da1b6064341d52ecab70a586b6701d9f6978&v=4'
tags: [engineering]
---

## 動機

隨著科技進步，行動應用程式在日常生活中扮演越來越重要的角色，開發無障礙應用程式的需求也隨之增長。

React Native 有限的無障礙 API 一直是開發者的痛點，因此我們對無障礙 API 進行了一些更新，讓開發者能更輕鬆地打造包容性行動應用程式。

## 現有 API 的問題

### 問題一：兩個完全不同卻又相似的屬性 - accessibilityComponentType (Android) 與 accessibilityTraits (iOS)

`accessibilityComponentType` 和 `accessibilityTraits` 這兩個屬性分別用於告知 Android 的 TalkBack 和 iOS 的 VoiceOver 使用者正在互動的 UI 元素類型。這兩個屬性最大的問題在於：

1. **它們是兩個不同屬性，使用方式各異，卻有相同目的**。在舊版 API 中，這兩個屬性分別對應不同平台，不僅不便，也讓許多開發者感到困惑。iOS 的 `accessibilityTraits` 允許 17 種不同值，而 Android 的 `accessibilityComponentType` 僅允許 4 種值。此外，這些值大多沒有重疊。甚至這兩個屬性的輸入類型也不同：`accessibilityTraits` 允許傳入特徵陣列或單一特徵，而 `accessibilityComponentType` 僅允許單一值。
2. **Android 的功能非常有限**。舊版屬性中，Talkback 僅能識別「button」、「radiobutton_checked」和「radiobutton_unchecked」這三種 UI 元素。

### 問題二：缺乏無障礙提示 (Accessibility Hints)

無障礙提示能幫助使用 TalkBack 或 VoiceOver 的使用者理解當他們對無障礙元素執行操作時會發生的結果（這些結果無法僅透過無障礙標籤顯現）。這些提示可以在設定面板中開啟或關閉。先前 React Native 的 API 完全不支援無障礙提示。

### 問題三：忽略反轉色彩

部分視力受損使用者會在手機上啟用色彩反轉功能以增強螢幕對比度。Apple 為 iOS 提供的 API 允許開發者忽略特定視圖，如此一來，當使用者開啟色彩反轉設定時，圖片和影片不會失真。此 API 目前不受 React Native 支援。

## 新版 API 設計

### 解決方案一：合併 accessibilityComponentType (Android) 與 accessibilityTraits (iOS)

為了解決 `accessibilityComponentType` 和 `accessibilityTraits` 之間的混淆問題，我們決定將它們合併為單一屬性。這很合理，因為它們在技術上具有相同的預期功能，且透過合併，開發者在建置無障礙功能時無需再擔心平台特定細節。

**背景**

在 iOS 上，`UIAccessibilityTraits` 是可以設定在任何 NSObject 上的屬性。透過 JavaScript 屬性傳遞到原生的 17 種特徵，每種都會映射到 Objective-C 的 `UIAccessibilityTraits` 元素。每個特徵由一個長整數表示，所有設定的特徵會透過 OR 運算結合。

而在 Android 上，`AccessibilityComponentType` 是 React Native 自創的概念，並未直接對應到 Android 的任何屬性。無障礙功能由無障礙委派處理。每個視圖都有預設的無障礙委派。若要自訂任何無障礙操作，必須建立新的無障礙委派，覆寫你想自訂的特定方法，然後將處理中的視圖的無障礙委派設定為與新委派關聯。當開發者設定 `AccessibilityComponentType` 時，原生程式碼會根據傳入的元件建立新委派，並將視圖設定為使用該無障礙委派。

**所做的變更**

針對新屬性，我們希望建立這兩個屬性的超集。我們決定讓新屬性主要沿用現有屬性 `accessibilityTraits` 的模型，因為 `accessibilityTraits` 的值明顯更多。這些特徵在 Android 上的功能將透過修改無障礙委派來實現填充。

There are 17 values of UIAccessibilityTraits that `accessibilityTraits` on iOS can be set to. However, we didn't include all of them as possible values to our new property. This is because the effect of setting some of these traits is actually not very well known, and many of these values are virtually never used.

The values UIAccessibilityTraits were set to generally took on one of two purposes. They either described a role that UI element had, or they described the state a UI element was in. Most uses of the previous properties we observed usually used one value that represented a role and combined it with either “state selected,” “state disabled,” or both. Therefore, we decided to create two new accessibility properties: `accessibilityRole` and `accessibilityState`.

**`accessibilityRole`**

新屬性`accessibilityRole`用於告知Talkback或Voiceover某個UI元素的角色。此屬性可接受以下其中一個值：

- `none`
- `button`
- `link`
- `search`
- `image`
- `keyboardkey`
- `text`
- `adjustable`
- `header`
- `summary`
- `imagebutton`

This property only allows one value to be passed in because UI elements generally don't logically take on more than one of these. The exception is image and button, so we've added a role imagebutton that is a combination of both.

**`accessibilityStates`**

新屬性`accessibilityStates`用於告知Talkback或Voiceover某個UI元素的狀態。此屬性接受一個包含以下一個或多個值的陣列：

- `selected`
- `disabled`

### 解決方案二：新增無障礙提示

為此，我們新增了一個屬性`accessibilityHint`。設置此屬性將讓Talkback或Voiceover向用戶朗讀提示。

**`accessibilityHint`**

此屬性接受一個字串形式的無障礙提示，供朗讀使用。

On iOS, setting this property will set the corresponding native property AccessibilityHint on the view. The hint will then be read by Voiceover if Accessibility Hints are turned on in the iPhone.

在Android上，設置此屬性會將提示值附加到無障礙標籤的末尾。這種實現方式的優點是模擬了iOS上的提示行為，但缺點是這些提示無法像iOS那樣在系統設置中關閉。

我們在Android上做出此決定的原因是，無障礙提示通常對應特定操作（例如點擊），而我們希望保持跨平台行為的一致性。

### 問題三的解決方案

**`accessibilityIgnoresInvertColors`**

We exposed Apple's api AccessibilityIgnoresInvertColors to JavaScript, so now when you have a view where you don't want colors to be inverted (e.g image), you can set this property to true, and it won't be inverted.

## 新用法

這些新屬性將在React Native 0.57版本中提供。

### 如何升級

如果你目前正在使用`accessibilityComponentType`和`accessibilityTraits`，以下是升級到新屬性的步驟。

#### 1. 使用jscodeshift

最簡單的使用案例可以通過運行jscodeshift腳本來替換。

此[腳本](https://gist.github.com/ziqichen6/246e5778617224d2b4aff198dab0305d)會替換以下實例：

```
accessibilityTraits=“trait”
accessibilityTraits={[“trait”]}
```

替換為

```
accessibilityRole= “trait”
```

此腳本還會移除 `AccessibilityComponentType` 的實例（假設所有設定 `AccessibilityComponentType` 的地方，您也會設定 `AccessibilityTraits`）。

#### 2. 使用手動代碼修改

對於那些使用 `AccessibilityTraits` 但沒有對應 `AccessibilityRole` 值的情況，以及那些傳入多個特性到 `AccessibilityTraits` 的情況，需要進行手動代碼修改。

一般來說，

```tsx
accessibilityTraits= {[“button”, “selected”]}
```

會手動替換為

```tsx
accessibilityRole=“button”
accessibilityStates={[“selected”]}
```

這些屬性已經在 Facebook 的代碼庫中使用。Facebook 的代碼修改出人意料地簡單。jscodeshift 腳本解決了大約一半的實例，另一半則手動修復。整個過程總共只花了不到幾個小時。

希望您會發現更新後的 API 很有用！並請繼續讓應用程式保持可訪問性！#包容性