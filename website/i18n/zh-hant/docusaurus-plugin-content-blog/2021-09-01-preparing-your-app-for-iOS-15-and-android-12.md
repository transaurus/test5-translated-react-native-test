---
title: Preparing Your App for iOS 15 and Android 12
authors: [SamuelSusla]
tags: [engineering]
---

大家好！

隨著新版行動作業系統將於今年底發布，我們建議您預先準備好 React Native 應用程式，以避免正式版本推出時出現功能倒退的情況。

<!--truncate-->

## iOS 15

iOS 15 的發布日期尚未公布，但根據過往 iOS 版本的發布時間，很可能會落在 9 月 16 日左右。若您的應用程式需要調整以相容 iOS 15，請務必預留 App Store 審核所需的時間。

### 需注意事項

#### 快速輸入欄 (QuickType Bar)

The way to disable _QuickType_ bar in _[TextInput](/docs/textinput)_ has changed. _QuickType_ bar is the bar above keyboard with three suggested words. In case your UI needs to have the bar hidden, setting [autoCorrect](/docs/textinput#autocorrect) to `false` no longer disables _QuickType_ bar in iOS 15 like earlier versions. In order to hide the _QuickType_ bar, you need to also set [spellCheck](/docs/textinput#spellcheck-ios) to `false`. This will disable spell check, the red underlines, in your _TextInput_. Disabling QuickType bar with spell check enabled is no longer an option.

<figure>
  <img src="/blog/assets/ios-15-quicktype-bar.png" alt="Screenshot of QuickType bar" />
  <figcaption>
    QuickType bar with three suggested words
  </figcaption>
</figure>

若要在 iOS 15 停用 QuickType 欄位，請將屬性 [spellCheck](/docs/textinput#spellcheck-ios) 和 [autoCorrect](/docs/textinput#autocorrect) 皆設為 `false`。

```jsx
<TextInput
  placeholder="something"
  autoCorrect={false}
  spellCheck={false}
/>
```

#### 透明導覽列

iOS 15 變更了導覽列的預設行為。與 iOS 14 不同，當內容滾動至頂部時，導覽列會變為透明。請注意此變化可能導致內容難以閱讀。解決方法可參考 [此討論串](https://developer.apple.com/forums/thread/682420)。

![iOS 14 與 iOS 15 導覽列對照截圖](/blog/assets/ios-15-navigation-bar.jpg)

### 如何安裝 iOS 15

#### 實體裝置

若您有備用裝置，可加入 [測試版計畫](https://beta.apple.com/sp/betaprogram/) 安裝 iOS 15。目前測試版已趨於穩定，但請注意 **升級至 iOS 15 後將無法降版**。

#### 模擬器

若要在 iOS 15 模擬器測試應用程式，您需下載 Xcode 13。下載連結請見 [此處](https://developer.apple.com/xcode/)。

## Android 12

Android 12 將於今年秋季發布，其變更可能影響應用程式體驗。依慣例，Google Play 會要求應用程式的目標 SDK 需在隔年 11 月前完成升級（過往要求請參閱 [此文件](https://developer.android.com/distribute/best-practices/develop/target-sdk)）。

### 需注意事項

#### 過度滾動效果

Android 12 新增了影響所有滾動容器的 [過度滾動效果](https://developer.android.com/about/versions/12/overscroll)。由於 React Native 的滾動視圖基於原生視圖，建議檢查您的可滾動容器是否正確套用此效果。您可透過將 [`overScrollMode`](/docs/scrollview#overscrollmode-android) 屬性設為 `never` 來停用此效果。

#### 權限更新

當應用程式請求 **`ACCESS_FINE_LOCATION`** 權限時，Android 12 允許使用者僅提供「近似位置」存取權限。詳細說明請見 [此處](https://developer.android.com/about/versions/12/approximate-location)。

查看 Google 提供的 [Android 12 所有應用行為變更詳情](https://developer.android.com/about/versions/12/behavior-changes-all)。

### 如何安裝 Android 12

#### 實體裝置

If you have a spare Android device, check if you’re able to install Android 12 Beta via [instructions here.](https://developer.android.com/about/versions/12/get)

#### 模擬器

若無可用裝置，可依照[此處指示](https://developer.android.com/about/versions/12/get#on_emulator)設定模擬器。