---
title: Built with React Native - The Build.com app
author: Garrett McCullough
authorTitle: Senior Mobile Engineer
authorURL: 'https://twitter.com/gwmccull'
authorImageURL: 'https://pbs.twimg.com/profile_images/955503100785172486/UrMKkQXc_400x400.jpg'
authorTwitter: gwmccull
tags: [showcase]
---

[Build.com](https://www.build.com/)總部位於加州奇科市，是家裝用品領域最大的線上零售商之一。我們的團隊擁有18年以網頁為核心的業務經驗，並於2015年開始規劃行動應用程式。由於團隊規模小且原生開發經驗有限，分別開發獨立的Android和iOS應用程式並不實際。因此，我們決定冒險採用當時非常新的React Native框架。我們的初始提交於2015年8月12日，使用的是React Native v0.8.0！並於2016年10月15日在兩大應用商店上線。過去兩年來，我們持續升級並擴展應用程式功能，目前使用的版本是React Native 0.53.0。

您可以在[https://www.build.com/app](https://www.build.com/app)查看我們的應用程式。

<p align="center">
  <img src="/blog/assets/build-com-blog-image.jpg" />
</p>

## 功能特色

我們的應用程式功能完整，包含電子商務應用應有的所有功能：產品列表、搜尋與排序、複雜產品配置、收藏清單等。我們接受標準信用卡支付方式，並為iOS用戶提供PayPal和Apple Pay選項。

以下是一些您可能意想不到的亮點功能：

1. 提供約40種產品、90種表面處理的3D模型
2. 擴增實境(AR)功能可讓用戶以98%的尺寸準確度預覽燈具與水龍頭在家中的實際效果。Build.com的React Native應用程式更因此被Apple App Store選為AR購物推薦應用！此功能現已支援Android與iOS平台！
3. 協作式專案管理功能，讓用戶能為專案不同階段建立採購清單並共同篩選商品

我們正在開發多項令人興奮的新功能，包括AR沉浸式購物的下一階段升級，持續優化應用體驗。

## 開發工作流程

Build.com允許每位開發者選擇最適合自己的工具。

- 整合開發環境(IDE)包含Atom、IntelliJ、VS Code、Sublime、Eclipse等
- 單元測試方面，開發者需為新元件編寫Jest單元測試，並正透過`jest-coverage-ratchet`逐步提高舊程式碼的測試覆蓋率
- 使用Jenkins建置測試版與候選版本，此流程運作良好，但生成版本說明等附屬文件仍需大量人工
- 整合測試由跨平台(桌面/行動/網頁)的共享測試團隊執行，自動化工程師正使用Java與Appium建置自動化測試套件
- 工作流程其他環節包括詳細的eslint配置、強制測試所需屬性的自訂規則，以及阻擋問題變更的pre-push鉤子

## 應用程式使用的函式庫

Build.com應用程式依賴多個常見開源函式庫，包括：Redux、Moment、Numeral、Enzyme及多個React Native橋接模組。我們也使用許多分叉(fork)的開源專案，分叉原因可能是原專案停止維護或需要客製功能。粗略統計約有115個JavaScript與原生依賴項，未來希望探索能移除未使用函式庫的工具。

我們正在導入TypeScript靜態型別檢查，並研究可選鏈(optional chaining)功能。這些技術將有助於解決以下兩類常見錯誤：

- 資料型別錯誤
- 因物件結構不符合預期而導致的undefined資料

## 開源貢獻

由於高度依賴開源生態，團隊致力於回饋社群。Build.com不僅允許開源內部開發的函式庫，更鼓勵員工貢獻於我們使用的開源專案。

我們已發布並維護多個React Native函式庫：

- `react-native-polyfill`
- `react-native-simple-store`
- `react-native-contact-picker`

我們也貢獻了許多函式庫，包括：React 和 React Native、`react-native-schemes-manager`、`react-native-swipeable`、`react-native-gallery`、`react-native-view-transformer`、`react-native-navigation`。

## 我們的旅程

在過去幾年，我們見證了 React Native 及其生態系統的快速成長。早期，似乎每個 React Native 版本都會修復一些錯誤，但同時也引入幾個新的問題。例如，Android 上的遠端 JS 除錯功能曾連續數個月無法使用。幸運的是，2017 年後情況變得穩定許多。

### 導航函式庫

我們反覆面臨的一大挑戰是導航函式庫。很長一段時間，我們使用 Expo 的 ex-nav 函式庫。它運作良好，但最終被棄用。然而，當時我們正處於密集的功能開發階段，抽時間更換導航函式庫並不可行。這意味著我們必須分叉該函式庫並進行修補，以支援 React 16 和 iPhone X。最終，我們得以遷移到 [`react-native-navigation`](https://github.com/wix/react-native-navigation)，希望它能持續獲得支援。

### 橋接模組

另一個重大挑戰是橋接模組。剛開始時，許多關鍵橋接功能缺失。一位隊友編寫了 `react-native-contact-picker`，因為我們的應用需要存取 Android 聯絡人選擇器。我們也遇到許多橋接模組因 React Native 的變更而損壞的情況。例如，React Native v40 有一個破壞性變更，當我們升級應用時，我不得不提交 PR 來修復 3 到 4 個尚未更新的函式庫。

## 展望未來

隨著 React Native 持續發展，我們對社群的願望清單包括：

- 穩定並改進導航函式庫
- 維護 React Native 生態系統中函式庫的支援
- 改善為專案添加原生函式庫和橋接模組的體驗

React Native 社群中的公司和個人一直非常樂於奉獻時間和精力來改進我們共同使用的工具。如果你尚未參與開源，我希望你能考慮改進你所使用函式庫的程式碼或文件。有許多文章可以幫助你入門，而且可能比你想象的要容易得多！