---
id: app-extensions
title: App Extensions
---

應用程式擴充功能讓您能在主應用程式之外提供自訂功能與內容。iOS 上有不同類型的應用程式擴充功能，這些都在 [App Extension Programming Guide](https://developer.apple.com/library/content/documentation/General/Conceptual/ExtensibilityPG/index.html#//apple_ref/doc/uid/TP40014214-CH20-SW1) 中有詳細說明。本指南將簡要介紹如何在 iOS 上利用這些擴充功能。

## 擴充功能中的記憶體使用

由於這些擴充功能是在常規應用沙箱外載入的，很可能會同時載入多個擴充功能。如您所料，這些擴充功能有嚴格的記憶體使用限制。開發時請務必注意這些限制。強烈建議在實體裝置上測試您的應用程式，開發擴充功能時更是如此：開發者經常發現擴充功能在 iOS 模擬器上運作正常，但實際裝置上的使用者卻回報無法載入。

我們強烈建議觀看 Conrad Kramer 關於 [擴充功能的記憶體使用](https://www.youtube.com/watch?v=GqXMqn6MXrM) 的演講，以深入了解此主題。

### 今日小工具

今日小工具的記憶體限制為 16 MB。使用 React Native 實作的今日小工具可能因記憶體用量過高而運作不穩定。若小工具顯示「無法載入」訊息，即表示超出記憶體限制：

![](/docs/assets/TodayWidgetUnableToLoad.jpg)

務必在實體裝置上測試擴充功能，但請注意這可能仍不足夠，尤其是今日小工具。除錯版建置較易超出記憶體限制，而正式版雖不會立即失敗，仍可能接近臨界值。強烈建議使用 [Xcode Instruments](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/index.html) 分析實際記憶體用量，因為正式版建置很可能已逼近 16 MB 上限。執行常見操作（如從 API 獲取資料）時，很容易就會超過限制。

若要實驗 React Native 今日小工具的極限，可嘗試擴充 [react-native-today-widget](https://github.com/matejkriz/react-native-today-widget/) 的範例專案。

### 其他應用程式擴充功能

其他類型的擴充功能有更高的記憶體限制。例如自訂鍵盤擴充限制為 48 MB，分享擴充則為 120 MB。使用 React Native 實作這類擴充功能更為可行。[react-native-ios-share-extension](https://github.com/andrewsardone/react-native-ios-share-extension) 即為概念驗證範例。