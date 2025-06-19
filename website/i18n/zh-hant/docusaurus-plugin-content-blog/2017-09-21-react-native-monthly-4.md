---
title: 'React Native Monthly #4'
authors: [grabbou]
tags: [engineering]
---

React Native 每月會議持續進行中！以下是各團隊的會議記錄：

### Callstack

- [React Native EU](https://react-native.eu) 已圓滿結束。來自 33 個國家、超過 300 位參與者齊聚弗羅茨瓦夫。演講內容可於 [YouTube 頻道](https://www.youtube.com/channel/UCUNE_g1mQPuyW975WjgjYxA/videos) 觀看。
- 我們正逐步恢復會議後的開源專案進度。值得一提的是，我們正在開發 [react-native-opentok](https://github.com/callstack/react-native-opentok) 的下一個版本，該版本將修復大多數現有問題。

### GeekyAnts

我們致力於降低開發者採用 React Native 的門檻，具體措施包括：

- Announced [BuilderX.io](https://builderx.io/) at [React Native EU](https://react-native.eu). BuilderX is a design tool that directly works with JavaScript files (only React Native is supported at the moment) to generate beautiful, readable, and editable code.
- Launched [ReactNativeSeed.com](https://reactnativeseed.com/) which provides a set of boilerplates for your next React Native project. It comes with a variety of options that include TypeScript & Flow for data-types, MobX, Redux, and mobx-state-tree for state-management with CRNA and plain React-Native as the stack.

### Expo

- Will release SDK 21 shortly, which adds support for react-native 0.48.3 and a bunch of bugfixes/reliability improvements/new features in the Expo SDK, including video recording, a new splash screen API, support for `react-native-gesture-handler`, and improved error handling.
- Re: [react-native-gesture-handler](https://github.com/kmagiera/react-native-gesture-handler), [Krzysztof Magiera](https://github.com/kmagiera) of [Software Mansion](https://swmansion.com/) continues pushing this forward and we've been helping him with testing it and funding part of his development time. Having this integrated in Expo in SDK21 will allow people to play with it easily in Snack, so we're excited to see what people come up with.
- Re: improved error logging / handling - see [this gist of an internal Expo PR](https://gist.github.com/brentvatne/00407710a854627aa021fdf90490b958) for details on logging, (in particular, "Problem 2"), and [this commit](https://github.com/expo/xdl/commit/1d62eca293dfb867fc0afc920c3dad94b7209987) for a change that handles failed attempts to import npm standard library modules. There is plenty of opportunity to improve error messages upstream in React Native in this way and we will work on follow up upstream PRs. It would be great for the community to get involved too.
- [native.directory](https://native.directory/) continues to grow, you can add your projects from [the GitHub repo](https://github.com/react-community/native-directory).
- Visit hackathons around North America, including [PennApps](https://pennapps.com/), [Hack The North](https://hackthenorth.com/), [HackMIT](https://hackmit.org/), and soon [MHacks](https://mhacks.org/).

### Facebook

- 正在改進 Android 平台的 `<Text>` 和 `<TextInput>` 元件（例如 `<TextInput>` 的原生自動調整高度、深層嵌套 `<Text>` 元件的佈局問題、更好的程式碼結構與效能優化）。
- 我們仍在尋找願意協助處理議題和 PR 的貢獻者。

### Microsoft

- Released Code Signing feature for CodePush. React Native developers are now able to sign their application bundles in CodePush. The announcement can be found [here](https://microsoft.github.io/code-push/articles/CodeSigningAnnouncement.html)
- Working on completing integration of CodePush to Mobile Center. Considering test/crash integration as well.

## 下次會議

下一次會議定於2017年10月10日星期三舉行。由於這只是我們的第四次會議，我們想知道這些筆記如何讓React Native社群受益。如果您對如何改進會議成果有任何建議，歡迎透過[Twitter](https://twitter.com/grabbou)與我聯繫。