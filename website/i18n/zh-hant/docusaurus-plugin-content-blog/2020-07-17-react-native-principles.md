---
title: React Native Team Principles
authors: [Eli White]
tags: [announcement]
---

Facebook 的 React Native 團隊遵循一系列原則，這些原則幫助我們決定如何優先處理 React Native 的工作。這些原則代表我們團隊的特定觀點，並不一定代表 React Native 社群中的每一位利益相關者。我們在此分享這些原則，是為了更透明地展示我們的驅動力、決策方式以及我們如何集中精力。

## **原生體驗**

我們對 React Native 的首要任務是**滿足人們對每個平台的期望**。這就是為什麼 React Native 會渲染到平台的原生元件。我們重視原生外觀和感覺，勝過跨平台的一致性。

例如，React Native 中的 TextInput 在 iOS 上會渲染為 UITextField。這確保了與密碼管理器和鍵盤控制的整合能夠開箱即用。通過使用平台原生元件，React Native 應用也能夠跟上 Android 和 iOS 新版本的設計和行為變化。

為了匹配原生應用的外觀和感覺，我們還必須匹配其性能。這是我們投入最多雄心的領域。例如，Facebook 創建了 Hermes，[一個專為 Android 上的 React Native 從頭構建的 JavaScript 引擎](https://facebook.github.io/react-native/blog/2019/07/17/hermes)。Hermes 顯著提高了 React Native 應用的啟動時間。我們還在進行重大的架構變更，以優化線程模型並使 React Native 更容易與原生代碼互操作。

## 大規模應用

Hundreds of screens in the Facebook app are implemented with React Native. The Facebook app is used by billions of people on a huge range of devices. **This is why** **we invest in the most challenging problems at scale.**

在我們的應用中部署 React Native 讓我們能夠發現小規模應用中看不到的問題。例如，Facebook 專注於提高從最新 iPhone 到許多舊款 Android 設備的性能。這一重點指導了我們的架構項目，如 Hermes、Fabric 和 TurboModules。

我們已經證明 React Native 也可以擴展到大型組織。當數百名開發人員在同一個應用上工作時，逐步採用是必須的。這就是為什麼我們確保 React Native 可以一個屏幕一個屏幕地採用。很快，我們將更進一步，允許將現有原生屏幕的單個原生視圖遷移到 React Native。

對大規模應用的關注意味著有許多事情我們的團隊目前沒有在進行。例如，我們的團隊不推動 React Native 在行業中的採用。我們也不積極構建解決方案來解決我們在大規模應用中沒有看到的問題。我們為擁有[許多合作夥伴和核心貢獻者](https://github.com/facebook/react-native/blob/master/ECOSYSTEM.md)而感到自豪，他們能夠專注於社群中這些重要的領域。

## 開發速度

優秀的用戶體驗是通過迭代創造的。**從代碼更改到在運行中的應用中看到結果應該只需要幾秒鐘**。React Native 的架構使其能夠在開發過程中提供近乎即時的反饋。

我們經常聽到團隊表示，採用 React Native 顯著提高了他們的開發速度。這些團隊發現，開發過程中的即時反饋使得嘗試不同想法和添加額外潤色變得更加容易，因為他們不必為每一個小更改中斷編碼會話。當我們對 React Native 進行更改時，我們確保保留這種開發體驗的品質。

即時反饋並不是 React Native 提高開發速度的唯一方式。團隊可以利用快速增長的高質量開源套件生態系統。團隊還可以在 Android、iOS 和網頁之間共享業務邏輯。這幫助他們更快地發布更新並減少平台團隊之間的組織孤島。

## 所有平台

When we introduced React Native in 2014, we presented it with our motto “Learn once, write anywhere” — and we mean _anywhere_. **Developers should be able to reach as many people as possible without being limited by device model or operating system.**

React Native 的目標平台非常多元，涵蓋行動裝置、桌面、網頁、電視、虛擬實境、遊戲主機等。我們希望讓開發者能在各平台上打造豐富的體驗，而非被迫為最低共通標準開發。為實現此目標，我們專注於支援每個平台的獨特功能，從不同的輸入機制（如觸控、手寫筆、滑鼠）到根本不同的使用情境（如 VR 中的 3D 環境）。

此原則促使我們決定以跨平台的 C++ 實作 React Native 的新核心架構，以促進各平台間的對等性。我們也正針對其他平台維護者（如微軟的 Windows 和 macOS）精進公開介面，致力讓任何平台都能支援 React Native。

## 宣告式使用者介面

我們不主張在所有平台部署完全相同的使用者介面，而是相信**透過相同的宣告式程式模型來展現每個平台的獨特能力**。我們的宣告式程式模型就是 React。

根據我們的經驗，React 推廣的單向資料流能讓應用程式更易理解。我們傾向將畫面表達為宣告式元件的組合，而非以命令式管理的視圖。React 在網頁端的成功，以及新版原生 Android 和 iOS 框架的發展方向，都顯示產業同樣擁抱了宣告式 UI。

React 讓宣告式使用者介面普及化，但仍有許多未解難題是 React 特別適合解決的。React Native 將持續建立在 React 的創新之上，保持站在宣告式使用者介面運動的最前線。