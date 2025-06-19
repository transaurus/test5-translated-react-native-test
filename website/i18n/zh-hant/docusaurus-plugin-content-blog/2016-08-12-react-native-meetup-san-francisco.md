---
title: San Francisco Meetup Recap
authors: [hectorramos]
hero: '/blog/img/rnmsf-august-2016-hero.jpg'
tags: [events]
---

上週我有幸參加了在Zynga舊金山辦公室舉辦的[React Native Meetup](https://www.meetup.com/React-Native-San-Francisco/photos/27168649/#452793854)。約有200人與會，這成為了一個絕佳的機會，讓我認識附近其他對React Native感興趣的開發者。

![](/blog/assets/rnmsf-august-2016-hero.jpg)

我特別想了解Zynga、Netflix和Airbnb等公司如何使用React和React Native。當晚的議程如下：

- 使用React進行快速原型開發
- 為React Native設計API
- 彌合差距：在現有代碼庫中使用React Native

但首先，活動以簡短的介紹和近期新聞回顧開場：

- 你知道React Native現在是[GitHub上最受歡迎的Java倉庫](https://twitter.com/jamespearce/status/759637111880359937)嗎？
- [rnpm](https://github.com/rnpm/rnpm)現在已成為React Native核心的一部分！你可以使用`react-native link`來替代`rnpm link`，以[安裝帶有原生依賴的庫](/docs/linking-libraries-ios)。
- React Native Meetup社群正在快速成長！現在全球各地有超過4,800名開發者參與各種React Native Meetup小組。

如果[這些Meetup](https://www.meetup.com/find/?allMeetups=false&keywords=react+native&radius=Infinity&userFreeform=San+Francisco%2C+CA&mcId=z94105&mcName=San+Francisco%2C+CA&sort=recommended&eventFilter=mysugg)在你附近舉辦，我強烈建議參加！

## 在Zynga使用React進行快速原型開發

第一輪新聞之後，由當晚的主辦方Zynga進行簡短介紹。Abhishek Chadha談到他們如何使用React在移動設備上快速原型化新體驗，並演示了一個類似Draw Something的應用原型。他們採用了與React Native類似的方法，通過橋接提供對原生API的訪問。當Abhishek使用設備的相機拍攝觀眾照片並在某人的頭上畫了一頂帽子時，這一點得到了展示。

## 在Netflix為React Native設計API

接下來是當晚的第一場主題演講。Netflix的高級軟體工程師[Clarence Leung](https://twitter.com/clarler)發表了關於為React Native設計API的演講。他首先指出開發者可能處理的兩類主要庫：如標籤欄和日期選擇器這樣的組件，以及提供對原生服務（如相機膠卷或應用內支付）訪問的庫。在為React Native構建庫時，有兩種主要方法：

- 提供平台特定的組件
- 構建一個跨平台庫，為Android和iOS提供相似的API

每種方法都有其考量，由你決定哪種最適合你的需求。

**方法一**

作為平台特定組件的例子，Clarence談到了React Native核心中的DatePickerIOS和DatePickerAndroid。在iOS上，日期選擇器作為UI的一部分渲染，可以輕鬆嵌入現有視圖中，而在Android上，日期選擇器以模態方式呈現。在這種情況下，提供獨立的組件是有意義的。

**方法二**

另一方面，照片選擇器在Android和iOS上的處理方式相似。雖然有一些細微差別——例如，Android不會像iOS那樣將照片分組到「自拍」等文件夾中——但這些可以輕鬆通過`if`語句和`Platform`組件來處理。

無論您最終選擇哪種方法，最小化 API 表面積並構建針對特定應用的程式庫都是個好主意。舉例來說，iOS 的應用內購買框架支援一次性消費性購買與可續訂訂閱。如果您的應用只需要支援消費性購買，或許可以在跨平台程式庫中省略訂閱功能。

![](/blog/assets/rnmsf-august-2016-netflix.jpg)

Clarence 的演講結束後進行了簡短的問答環節。其中一個有趣的細節是，Netflix 為這些程式庫編寫的 React Native 程式碼中，約有 80% 是 Android 和 iOS 共用的。

## 彌合鴻溝：在現有程式碼庫中使用 React Native

當晚最後一場演講由 Airbnb 的 [Leland Richardson](https://twitter.com/intelligibabble) 主講，主題聚焦於如何在現有程式碼庫中使用 React Native。我已經知道從頭開始用 React Native 編寫新應用有多簡單，因此對 Airbnb 在現有原生應用中採用 React Native 的經驗非常感興趣。

Leland 首先談到了「綠地專案」(greenfield) 與「棕地專案」(brownfield) 的區別。綠地專案是指無需考慮先前工作的新專案，而棕地專案則需要考量現有專案的需求、開發流程及各團隊的不同需求。

在綠地應用中，React Native CLI 會為 Android 和 iOS 設置單一儲存庫，一切都能順利運作。Airbnb 採用 React Native 的第一個挑戰是，他們的 Android 和 iOS 應用各自擁有獨立的儲存庫。多儲存庫架構的公司需要克服一些障礙才能採用 React Native。

為了解決這個問題，Airbnb 首先為 React Native 程式碼庫建立了新儲存庫。他們透過持續整合伺服器將 Android 和 iOS 儲存庫鏡像到這個新儲存庫中。在執行測試並建構套件後，建構產物會同步回 Android 和 iOS 儲存庫。這種方式讓行動工程師能在不改變開發環境的情況下處理原生程式碼，無需安裝 npm、執行封裝程式或記得建構 JavaScript 套件。而實際編寫 React Native 程式碼的工程師也無需擔心跨平台同步問題，因為他們直接在新儲存庫中工作。

這種做法存在一些缺點，主要是無法進行原子更新。需要同時修改原生和 JavaScript 程式碼的變更，必須提交三個獨立的 pull request，且都需謹慎合併。為避免衝突，當主分支在建置期間發生變更時，CI 會中止將變更同步回 Android 和 iOS 儲存庫的流程，這在高提交頻率時段（例如發布新版本時）會導致嚴重延遲。

Airbnb 後來轉向單一儲存庫架構。幸運的是這個方案原本就在考慮中，當 Android 和 iOS 團隊熟悉 React Native 後，他們樂於加速遷移至單一儲存庫。

這解決了分散式儲存庫架構的大部分問題。Leland 也指出，這會對版本控制伺服器造成較大負荷，對小型公司可能是個問題。

![](/blog/assets/rnmsf-august-2016-airbnb.jpg)

### 導航難題

Leland 演講的後半部分聚焦於我深切關注的主題：React Native 中的導航問題。他談到 React Native 中豐富的導航程式庫生態，包括官方與第三方方案。NavigationExperimental 曾被視為有潛力的解決方案，但最終不符合他們的使用情境。

事實上，現有的導航程式庫似乎都不太適合棕地應用。這類應用要求導航狀態必須完全由原生應用掌控。例如，當使用者在 React Native 視圖中操作時若會話過期，原生應用必須能接管流程並視需要顯示登入畫面。

Airbnb 同時希望避免在過渡期間用 JavaScript 版本取代原生導航欄，因為這種效果可能會令人不適。最初他們將自己限制在模態呈現的視圖中，但這顯然在更廣泛地採用 React Native 時帶來了問題。

他們決定需要開發自己的函式庫。這個函式庫名為 `airbnb-navigation`。由於該函式庫與 Airbnb 的代碼庫緊密耦合，目前尚未開源，但他們計劃在今年年底前發布。

我不會深入探討該函式庫的 API 細節，但以下是幾個關鍵要點：

- 必須事先預註冊場景
- 每個場景都在自己的 `RCTRootView` 中顯示。它們在各平台上以原生方式呈現（例如 iOS 上使用 `UINavigationController`）。
- 場景中的主要 `ScrollView` 應包裹在 `ScrollScene` 組件中。這樣可以充分利用原生行為，例如在 iOS 上點擊狀態欄以滾動至頂部。
- 場景之間的轉場由原生處理，無需擔心性能問題。
- 自動支援 Android 返回按鈕。
- 可以通過無 UI 的 Navigator.Config 組件利用基於 View Controller 的導航欄樣式設定。

還有一些需要注意的事項：

- 導航欄在 JavaScript 中不易自訂，因為它是原生組件。這是刻意設計的，因為使用原生導航欄是此類函式庫的硬性要求。
- ScreenProps 在通過橋接發送時必須序列化/反序列化，因此如果在此處發送過多數據，需格外小心。
- 導航狀態由原生應用擁有（這也是函式庫的硬性要求），因此像 Redux 這樣的工具無法操作導航狀態。

Leland 的演講結束後也進行了問答環節。總體而言，Airbnb 對 React Native 感到滿意。他們有興趣使用 Code Push 來修復問題而無需通過 App Store，且他們的工程師喜歡 Live Reload 功能，因為無需在每次小改動後等待原生應用重新構建。

## 閉幕致詞

活動最後分享了一些 React Native 的最新消息：

- Deco 宣布了他們的 [React Native Showcase](https://www.decosoftware.com/showcase)，並邀請大家將自己的應用添加到列表中。
- 最近的[文件全面翻新](/blog/2016/07/06/toward-better-documentation)獲得了特別提及！
- Deco IDE 的創建者之一 Devin Abbott 將開設一門 [React Native 入門課程](https://www.decosoftware.com/course)。

![](/blog/assets/rnmsf-august-2016-docs.jpg)

聚會提供了與社區中其他開發者見面並學習的良機。我期待未來參加更多 React Native 聚會。如果你也參加了這些活動，請留意我並告訴我我們如何能讓 React Native 更好地為你服務！