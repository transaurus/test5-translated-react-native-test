---
title: Hermes as the Default
authors: [micleo]
tags: [announcement, release]
---

# 將 Hermes 設為預設引擎的部落格公告

Last October, we [announced](/blog/2021/10/26/toward-hermes-being-the-default) that we had started work towards **making** **Hermes the default engine for all React Native apps**.

Hermes 在 Meta 內部為 React Native 提供了極大價值，我們相信開源社群也將受益。Hermes 專為資源受限的設備設計，並針對啟動時間、應用大小和記憶體消耗進行優化。Hermes 與其他 JS 引擎的關鍵差異在於其能預先將 JavaScript 原始碼編譯為位元組碼。這些預編譯的位元組碼會打包進二進位檔中，省去直譯器在應用啟動時執行此昂貴步驟的需求。

自宣布以來，我們投入大量工作改進 Hermes。如今，我們興奮地宣布**React Native 0.70 將預設搭載 Hermes 引擎**。這意味著所有基於 v0.70 的新專案都將預設啟用 Hermes。隨著七月即將到來的版本發布，我們希望與社群密切合作，確保過渡順利並為所有用戶帶來價值。本篇部落格將說明此變更的預期效益、效能基準測試、新功能等內容。您無需等待 React Native 0.70 發布，現在即可**依照[這些指示](/docs/hermes#enabling-hermes)在現有 React Native 應用中啟用 Hermes**。

請注意，雖然新 React Native 專案將預設啟用 Hermes，但我們仍會持續支援其他引擎。

<!--truncate-->

## 效能基準測試

我們針對應用開發者重視的三項指標進行量測：TTI（可互動時間）、二進位檔大小與記憶體消耗。測試採用 React Native 應用 [Mattermost](https://github.com/mattermost/mattermost-mobile)，並在 2020 年的高階硬體設備上執行 Android 與 iOS 測試。

- TTI（可互動時間）指從應用啟動到用戶可進行操作的時間長度。在此基準測試中，我們將其定義為從點擊應用圖示到首個畫面渲染完成的時間。我們同時提供 Mattermost 啟動過程的螢幕錄影。
- 二進位檔大小在 Android 上以 APK 大小量測，iOS 上則以 IPA 大小量測。
- 記憶體消耗數據透過在 Mattermost 應用中執行相同操作約數分鐘後收集，兩引擎的測試條件完全一致。

## Android 基準測試數據

所有 Android 測試均在三星 Galaxy S20 上執行。

<figure>
  <img src="/blog/assets/hermes-default-android-data.png" alt="Android Benchmarking Data" />
</figure>

### TTI 影片

<figure>
  <img src="/blog/assets/hermes-default-android-video.gif" alt="Android TTI Video" />
</figure>

## iOS 基準測試數據

所有 iOS 測試均在 iPhone 12 Pro 上執行。

<figure>
  <img src="/blog/assets/hermes-default-ios-data.png" alt="iOS Benchmarking Data" />
</figure>

### TTI 影片

<figure>
  <img src="/blog/assets/hermes-default-ios-video.gif" alt="iOS TTI Video" />
</figure>

### 減速版 TTI 影片（更清晰展示啟動時間差異）

<figure>
  <img src="/blog/assets/hermes-default-ios-slow-video.gif" alt="iOS Slowed Down TTI Video" />
</figure>

## React Native/Hermes 整合

我們解決了一個長期存在且導致相容性問題的痛點：React Native 過去透過 CocoaPods 和 npm 分發的預建二進位檔依賴 Hermes，這可能引發 API 或 [ABI 不相容](https://github.com/react-native-community/discussions-and-proposals/issues/257)問題。為解決此問題，自 React Native 0.69 起，Hermes 會與每個 React Native 版本同步建置，確保版本間的完整相容性。此舉也創造更緊密的整合，能加速功能開發與錯誤修復的迭代週期，並讓我們更有信心確保 Hermes 的重大變更正確無誤。更多關於新整合變更的深入說明請參閱[此處](https://github.com/facebook/react-native-website/pull/3159/files)。

## iOS Intl 支援

我們已完成 iOS 平台對 [`Intl`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl)（ECMAScript 國際化 API）的支援實作，該 API 提供廣泛的語言敏感功能。這項長期存在的[缺口](https://github.com/facebook/hermes/issues/23)曾阻礙部分開發者採用 Hermes。與 Microsoft 合作開發的 Android 實作已隨 React Native 0.65 發布，而 React Native 0.70 將讓開發者在雙平台都獲得原生支援。

典型 `Intl` 實作需導入大型查找表或 [Unicode CLDR](https://cldr.unicode.org/index) 等資料，但這可能導致二進位檔膨脹達 6MB。為避免 Hermes 二進位檔過大，我們透過呼叫 iOS 原生 API 來實作 `Intl`，直接利用 iOS 內建的語系與國際化資料集。

## 進行中的工作

在持續演進 Hermes 的過程中，我們希望向社群說明當前優先事項：改善開發者體驗，並確保沒有人會因 JavaScript 語言功能缺失而放棄使用 Hermes。具體而言，我們正在：

- 讓開發者能直接從 Chrome 開發者工具介面執行取樣分析器
- 新增對 [`BigInt`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) 的支援（此社群長期需求無法透過 polyfill 實現）
- 新增對 [`WeakRef`](https://github.com/facebook/hermes/issues/658) 的支援，將提供開發者新的記憶體管理控制項

## 總結

Hermes 成為預設引擎標誌著長期旅程的開端。我們正在開發新功能，讓社群能持續打造高效應用程式。歡迎在 [GitHub 儲存庫](https://github.com/facebook/react-native) 回報錯誤、提問或提供建議！我們已建立 `hermes` 標籤供 Hermes 相關討論使用。