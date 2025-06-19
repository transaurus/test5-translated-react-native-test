---
title: Releasing React Native 0.59
author: Ryan Turner
authorTitle: Core Maintainer & React Native Developer
authorURL: 'https://twitter.com/turnrye'
authorImageURL: 'https://avatars0.githubusercontent.com/u/701035?s=460&v=4'
authorTwitter: turnrye
tags: [announcement, release]
---

歡迎來到 React Native 0.59 版本！這是一個包含 88 位貢獻者提交的 644 次 commit 的重大版本。貢獻也以其他形式呈現，因此 _感謝您_ 維護 issue、培育社群，以及向人們傳授 React Native 知識。本月帶來了一些備受期待的變更，希望您會喜歡。

## 🎣 Hooks 登場

React Hooks 是此版本的一部分，它讓您能在元件間重用有狀態的邏輯。關於 hooks 有許多討論，但如果您還沒聽說過，請看看以下一些精彩的資源：

> - [Hooks 簡介](https://reactjs.org/docs/hooks-intro.html) 解釋了為什麼我們要將 Hooks 加入 React。
> - [Hooks 概覽](https://reactjs.org/docs/hooks-overview.html) 是對內建 Hooks 的快速概述。
> - [建立自訂 Hooks](https://reactjs.org/docs/hooks-custom.html) 展示了如何透過自訂 Hooks 重用程式碼。
> - [理解 React Hooks](https://medium.com/@dan_abramov/making-sense-of-react-hooks-fdbde8803889) 探討了 Hooks 解鎖的新可能性。
> - [useHooks.com](https://usehooks.com/) 展示了社群維護的 Hooks 範例和演示。

請務必在您的應用中試試這個功能。我們希望您能像我們一樣對這種重用方式感到興奮。

## 📱 更新的 JSC 帶來效能提升與 Android 64 位元支援

React Native 使用 JSC ([JavaScriptCore](https://webkit.org/)) 來驅動您的應用程式。Android 上的 JSC 版本較舊，這意味著許多現代 JavaScript 功能不受支援。更糟的是，與 iOS 的現代 JSC 相比，它的效能較差。隨著此版本的發布，這一切都將改變。

感謝 [@DanielZlotin](https://github.com/danielzlotin)、[@dulmandakh](https://github.com/dulmandakh)、[@gengjiawen](https://github.com/gengjiawen)、[@kmagiera](https://github.com/kmagiera) 和 [@kudo](https://github.com/kudo) 的出色工作，JSC 已經趕上了過去幾年的進展。這帶來了 64 位元支援、現代 JavaScript 支援，以及 [顯著的效能提升](https://github.com/react-native-community/jsc-android-buildscripts/tree/master/measure)。同時也感謝他們讓這個過程變得可維護，使我們能夠在未來輕鬆利用 WebKit 的改進，並感謝 Software Mansion 和 Expo 讓這項工作成為可能。

## 💨 透過 inline requires 加速應用啟動

我們希望幫助人們預設擁有高效能的 React Native 應用，並致力於將 Facebook 的優化帶給社群。應用程式按需載入資源，而不是減慢啟動速度。這項功能稱為「inline requires」，它讓 Metro 能夠識別需要延遲載入的元件。具有深層且多樣化元件結構的應用將看到最大的改進。

![0.59 模板中的 `metro.config.js` 檔案來源，展示了如何啟用 `inlineRequires`](/blog/assets/inline-requires.png)

在我們預設啟用此功能之前，需要社群的反饋。當您升級到 0.59 時，會看到一個新的 `metro.config.js` 檔案；將選項設為 true 並給我們 [您的意見](https://twitter.com/hashtag/inline-requires)！閱讀更多關於 inline requires 的資訊 [在效能文件中](/docs/performance#inline-requires) 以評估您的應用程式。

## 🚅 Lean core 進行中

React Native 是一個龐大且複雜的專案，擁有複雜的儲存庫。這使得程式碼庫對貢獻者不夠友好、難以測試，並且作為開發依賴顯得臃腫。[Lean Core](https://github.com/react-native-community/discussions-and-proposals/issues/6) 是我們透過將程式碼遷移到獨立的函式庫以更好地管理這些問題的努力。過去幾個版本已經看到了這方面的初步進展，但 [讓我們認真對待](https://www.youtube.com/watch?v=FMLKb4or8yg)。

您可能會注意到，現在有額外的元件已被正式棄用。這是個好消息，因為這些功能現在有維護者積極維護它們。請留意警告訊息並遷移至這些功能的新函式庫，因為它們將在未來的版本中被移除。下方表格列出了元件、其狀態以及您可以遷移使用的替代方案。

| Component            | Deprecated? | New home                                                                                                                                                 |
| -------------------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AsyncStorage**     | 0.59        | [@react-native-community/react-native-async-storage](https://github.com/react-native-community/react-native-async-storage)                               |
| **ImageStore**       | 0.59        | [expo-file-system](https://github.com/expo/expo/tree/master/packages/expo-file-system) or [react-native-fs](https://github.com/itinance/react-native-fs) |
| **MaskedViewIOS**    | 0.59        | [@react-native-community/react-native-masked-view](https://github.com/react-native-community/react-native-masked-view)                                   |
| **NetInfo**          | 0.59        | [@react-native-community/react-native-netinfo](https://github.com/react-native-community/react-native-netinfo)                                           |
| **Slider**           | 0.59        | [@react-native-community/react-native-slider](https://github.com/react-native-community/react-native-slider)                                             |
| **ViewPagerAndroid** | 0.59        | [@react-native-community/react-native-viewpager](https://github.com/react-native-community/react-native-viewpager)                                       |

在接下來的幾個月裡，還會有更多元件遵循這條精簡核心的路徑。我們正在尋求協助——請前往 [精簡核心總覽](https://github.com/facebook/react-native/issues/23313) 參與貢獻。

## 👩🏽‍💻 CLI 改進

React Native 的命令行工具是開發者進入生態系統的入口，但它們長期存在問題且缺乏官方支援。CLI 工具已移至 [新儲存庫](https://github.com/react-native-community/react-native-cli)，並且由 [專職的維護者團隊](https://blog.callstack.io/the-react-native-cli-has-a-new-home-79b63838f0e6) 進行了一些令人興奮的改進。

現在日誌的格式更加美觀。命令的執行速度幾乎是即時的——您會立即注意到差異：

![0.58 版本的 CLI 啟動緩慢](/blog/assets/0.58-cli-speed.png)![0.59 版本的 CLI 幾乎是瞬間啟動](/blog/assets/0.59-cli-speed.png)

## 🚀 升級至 0.59

我們聽取了您關於 [React Native 升級流程](https://github.com/react-native-community/discussions-and-proposals/issues/68) 的反饋，並正在採取措施在 [未來版本](https://github.com/react-native-community/discussions-and-proposals/issues/64#issuecomment-444775432) 中改善體驗。要升級至 0.59，我們建議使用 [`rn-diff-purge`](https://github.com/react-native-community/rn-diff-purge) 來確定您當前的 React Native 版本與 0.59 之間的變更，然後手動應用這些變更。一旦您將專案升級至 0.59，您將能夠使用新改進的 `react-native upgrade` 命令（基於 `rn-diff-purge`！）來升級至 0.60 及更高版本。

## 🔨 重大變更

0.59 中的 Android 支援已根據 Google 的最新建議進行了清理，這可能會導致現有應用程式出現問題。此問題可能表現為運行時崩潰並顯示訊息「You need to use a Theme.AppCompat theme (or descendant) with this activity」。我們建議更新您專案的 `AndroidManifest.xml` 文件，確保 `android:theme` 的值是 `AppCompat` 主題（例如 `@style/Theme.AppCompat.Light.NoActionBar`）。

The `react-native-git-upgrade` command has been removed in 0.59, in favor of the newly improved `react-native upgrade` command.

## 🤗 感謝

許多新貢獻者協助 [啟用從 Flow 類型生成原生代碼](https://github.com/facebook/react-native/issues/22990) 和 [解決 Xcode 警告](https://github.com/facebook/react-native/issues/22609)——這些是了解 React Native 工作原理並為公共利益做出貢獻的好方法。謝謝！請留意未來的類似問題。

雖然這些是我們注意到的亮點，但還有許多其他令人興奮的更新。要查看所有更新，請參閱 [變更日誌](https://github.com/react-native-community/react-native-releases/blob/master/CHANGELOG.md)。0.59 是一個重大版本——我們迫不及待想讓您試用。

我們在今年還會有更多改進。敬請期待！

[Ryan](https://github.com/turnrye) 及整個 [React Native 核心團隊](https://twitter.com/reactnative)