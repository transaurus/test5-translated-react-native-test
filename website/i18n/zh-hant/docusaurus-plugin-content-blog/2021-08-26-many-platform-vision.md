---
title: React Native's Many Platform Vision
authors: [abernathyca, Eli White, lunaleaps, yungsters]
tags: [announcement]
---

React Native 在提升移動開發標準方面取得了巨大成功，無論是在 Facebook 內部還是在業界其他領域。隨著我們與電腦互動的方式不斷演變，以及新設備的發明，我們希望 React Native 能夠為所有人服務。雖然 React Native 最初是為構建移動應用而創建的，但我們相信，專注於多平台並根據每個平台的優勢和限制進行開發，會產生共生效應。我們已經看到將這項技術擴展到桌面和虛擬現實所帶來的巨大好處，並且我們很高興能分享這對 React Native 未來的意義。

<!--truncate-->

## 尊重平台特性

我們的首要指導原則是[符合人們對每個平台的期望](https://reactnative.dev/blog/2020/07/17/react-native-principles#native-experience)。Android 用戶期望使用 TalkBack 的無障礙應用。導航應該像其他 Android 應用一樣運作。按鈕的外觀和感覺應該與 Android 上的按鈕一致，而不是像 iOS 的按鈕。儘管我們尋求提供一致的跨平台開發體驗，但我們抵制犧牲用戶期望的誘惑。

我們相信，React Native 能夠讓開發者在滿足用戶期望的同時，也能獲得更好的開發體驗。在本節中，我們分享以下內容：

1. 通過擁抱平台限制，我們實際上改善了其他平台的體驗。
2. 我們可以從機構知識中學習，構建更高層次的跨平台抽象。
3. 每個平台上的其他參與者激勵我們構建更好的開發者和用戶體驗。

### 擁抱平台限制

<!-- alex ignore easy -->

特定的設備硬件或用戶期望會帶來獨特的限制和要求。例如，Android 和 VR 頭戴設備的內存通常比 iOS、macOS 和 Windows 更受限。另一個例子是，用戶期望移動應用幾乎能立即啟動，但對桌面應用啟動時間較長的容忍度更高。**我們發現，通過用 React Native 解決這些問題，我們可以更容易地將從一個平台學到的經驗和代碼應用到其他平台。**

<figure>
  <img src="/blog/assets/many-platform-vision-facebook-dating.png" alt="Screenshot of Facebook Dating on mobile" />
  <figcaption>
    React Native and Relay power over 1000 Facebook surfaces on Android and iOS.
  </figcaption>
</figure>

例如，React Native 依賴於一種稱為「視圖扁平化」的優化，這對於減少 Android 上的內存使用至關重要。我們從未為 iOS 構建這種優化，因為它沒有相同的內存限制。在過去幾年中，當我們構建新的跨平台渲染器時，我們不得不重新實現「視圖扁平化」。但這次，它是用 C++ 而不是平台特定的 Java 編寫的。在 iOS 上嘗試這種優化現在只需切換一個開關。在 Facebook 的生產應用中，我們觀察到這提高了 iOS 的性能！我們可能永遠不會為 iOS 構建這個功能，但我們在 Android 上的投資能夠惠及 iOS。

### 從機構知識中學習

React Native 最初創建的原因之一是為了減少工程孤島。Android 工程師往往與從事同一產品的 iOS 工程師隔離。Android 工程師為 Android 工程師審查代碼，並參加 Android 的聚會和會議。iOS 工程師為 iOS 工程師審查代碼，並參加 iOS 的聚會和會議。從事不同平台的工程師帶來了關於如何構建優秀產品體驗的獨特領域和機構知識。

像 React Native 這樣的跨平台框架的最佳成果之一，是它們如何將具有完全不同領域專業知識的工程師聚集在一起。**我們相信，通過針對更多平台，我們可以加速平台專家之間機構知識的交叉傳播。**

例如，React Native 中的無障礙抽象受到 Android、iOS 和網頁各自處理無障礙的不同方式的影響。這使我們能夠構建一個共同的接口，改進兩個移動平台上無障礙提示的處理方式。

另一個例子是，我們對網頁用戶速度感知的研究導致了像 Suspense 這樣的並發功能。在過去一年中，這些功能由新的 [Facebook.com](https://facebook.com/) 網站進行了驗證。現在，隨著我們的新渲染器，這些功能正在進入 React Native，並影響我們如何設計事件調度和優先級。React 團隊在改善網頁體驗方面的投資正在惠及原生平台的 React Native。

### 競爭驅動創新

除了各領域的工程師、技術聚會和研討會外，每個平台還有其他獨特的參與者正在解決相似問題。在網頁端，React（直接驅動React Native的框架）經常從其他開源網頁框架如[Vue](https://vuejs.org/)、[Preact](https://preactjs.com/)和[Svelte](https://svelte.dev/)中汲取靈感。在行動端，React Native受到其他開源行動框架的啟發，同時我們也持續向Facebook內部開發的其他行動框架學習。

<!-- alex ignore special -->

**We believe that competition leads to better outcomes for everyone in the long run.** By studying what makes other players on each platform great, we can learn lessons that may apply to other platforms. For example, the race to simplify complex websites influenced the development of React and gave React Native a head start to offer a declarative framework for mobile apps. The demand for faster iteration cycles and build times for the web also led to the development of Fast Refresh which significantly benefited React Native. Similarly, performance optimizations in our internal mobile frameworks — especially around data fetching and parallelization — challenged us to improve React Native in a way that has also influenced React when we built the new [Facebook.com](https://facebook.com/) website.

<figure>
  <img src="/blog/assets/many-platform-vision-facebook-website.png" alt="Screenshot of the Facebook.com website" />
  <figcaption>
    React and Relay powers the <a href="https://facebook.com/">Facebook.com</a> website.
  </figcaption>
</figure>

## 擴展至新平台

React與React Native正處於轉折點。React已[啟動React 18版本的發布進程](https://reactjs.org/blog/2021/06/08/the-plan-for-react-18.html)，而[新版React Native渲染器已全面驅動Facebook行動應用](https://twitter.com/reactnative/status/1415099806507167745)。今年我們的團隊大幅擴編，以支持Facebook內部日益增長的採用需求。其他平台的開發團隊注意到這種趨勢，也看到了採用React Native能帶來的效益。

**For the past year, we have been partnering with Microsoft and the Messenger team to create a truly native video calling experience on Windows and macOS.** Due to the high scrutiny that we place on startup time for mobile apps, our initial implementation of desktop video calling using React Native completely blew away the performance of the Electron implementation that it replaced. For the first couple weeks of building this experience, we recorded videos of us resizing a window with multiple live video calls and marveled at the smooth frame rates.

為桌面平台開發令我們非常振奮。我們將行動端開發經驗應用於桌面平台，同時保持開放心態。我們拓展了視野，涵蓋多重子視窗、選單列、系統匣等桌面特性。隨著持續合作開發新的桌面版Messenger功能，我們預期將發現能反饋影響網頁與行動端開發的機會。若想掌握最新進展，我們的桌面協作專案正在[GitHub](https://github.com/microsoft/react-native-windows)進行。

<figure>
  <img src="/blog/assets/many-platform-vision-messenger-desktop.png" alt="Screenshot of the Messenger app on macOS" />
  <figcaption>
    React Native powers Video Calling in Messenger for Windows and macOS.
  </figcaption>
</figure>

**We are also partnering more closely with [Facebook Reality Labs](https://tech.fb.com/ar-vr/) to understand how React is uniquely positioned to deliver virtual reality experiences on Oculus.** Building experiences in VR with React Native will be particularly interesting because of tighter memory constraints and user sensitivity to interaction latency.

如同我們開發行動端React Native的方式，我們將先在Facebook規模驗證VR技術，再向公眾分享。目前仍有許多變動與未解問題，我們希望在確認技術達到生產環境的可靠性標準後，才會向社群發布。

雖然多數VR開發仍屬內部專案，我們期待能盡早分享更多進展。我們也預期React Native針對VR的改進將逐步開源，例如降低VR使用場景記憶體消耗的專案，同時也會改善React Native在行動與桌面端的記憶體使用效率。

<figure>
  <img src="/blog/assets/many-platform-vision-oculus-home.png" alt="Screenshot of Oculus Home in virtual reality" />
  <figcaption>
    React and Relay power the Oculus Home and many other virtual reality experiences.
  </figcaption>
</figure>

回顧業界採用React的歷程，社群始終對React擴展至更多平台展現高度興趣。早在我們正式發布React Native前，Netflix就已開發Gibbon——其基於React打造電視體驗的自訂渲染器。而在Facebook開始開發桌面版Messenger之前，[微軟早已運用React為Office和Windows 10建構原生桌面體驗](https://www.youtube.com/watch?v=IUMWFExtDSg&t=382s)。

## 總結

總而言之，我們的願景是將 React Native 的應用範圍擴展至行動裝置以外的平台，並已開始與 Facebook 的桌面及 VR 團隊展開合作。我們深知，當我們擁抱每個平台的限制、汲取機構知識，並從其他參與者身上獲得靈感時，整個生態系統中的每個平台都將受益。最重要的是，在此過程中，我們始終恪守 [我們的指導原則](https://reactnative.dev/blog/2020/07/17/react-native-principles)。

對於 React Native 在多平台探索中所開啟的可能性，我們感到無比振奮。歡迎透過 [@reactnative](https://twitter.com/reactnative) 與我們聯繫，獲取最新動態並分享您的想法！