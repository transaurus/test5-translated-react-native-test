---
title: The GAAD Pledge - March Accessibility Issues Update
authors: [alexmarlette]
tags: [announcement]
---

自從我們向GitHub社群發布經過徹底審查的差距分析與改進React Native無障礙功能的議題清單以來，已經過了四週。在React Native社群的協助下，我們在提升無障礙功能方面已取得顯著進展。社群成員們持續協助貢獻者、審查測試案例，並喚起對既有無障礙議題的關注。自3月8日起，社群已關閉六個議題，合併四個拉取請求，另有七個拉取請求正在審核流程中。

在這項工作持續進行的同時，Facebook的React Native團隊與無障礙團隊正在評估在此計劃前提交的無障礙錯誤與議題，以確認它們是否已被現有差距分析涵蓋，或是否有其他需要納入專案的新議題。目前已發現一個新議題並納入專案，四個議題直接對應到現有問題，另有兩個議題預計將透過解決根本原因的既有議題來關閉。

感謝所有參與的社群成員。你們確實推動了改變，讓React Native對所有人變得更無障礙！

<!--truncate-->

## 已合併的拉取請求 🎉

- [Added talkback support for button accessibility: disabled prop #31001](https://github.com/facebook/react-native/pull/31001) - closed by [@huzaifaaak ](https://twitter.com/huzaifaaak)

- [feat: set disabled accessibilityState when TouchableHighlight is disabled #31135](https://github.com/facebook/react-native/pull/31135) closed by [@natural_clar](https://twitter.com/natural_clar)

- [[Android] Selected State does not annonce when TextInput Component selected #31144](https://github.com/facebook/react-native/pull/31144) closed by [fabriziobertoglio1987](https://fabriziobertoglio.xyz/)

- [Added talkback support for TouchableNativeFeedback accessibility: disabled prop #31224](https://github.com/facebook/react-native/pull/31224) closed by [@kyamashiro73](https://twitter.com/kyamashiro73)

- [Accessibility/button test #31189](https://github.com/facebook/react-native/pull/31189) closed by [@huzaifaaak ](https://twitter.com/huzaifaaak)

  - Adds a test for accessibilityState for button

## 修復內容

- `Button`元件（由[#31001](https://github.com/facebook/react-native/pull/31001)修復）：

  - 現在會播報禁用狀態

  - 當按鈕禁用時，會停用螢幕閱讀器的點擊功能

  - 播報按鈕的選取狀態

- `TextInput`元件（由[#31144](https://github.com/facebook/react-native/pull/31144)修復）：

  - 當accessibilityState的"selected"設為true且元素獲得焦點時，會播報"已選取"

- `TouchableHighlight`元件（由[#31135](https://github.com/facebook/react-native/pull/31135)修復）：

  - 當元件禁用時，會停用螢幕閱讀器的點擊功能

- `TouchableNativeFeedback`元件（由[#31224](https://github.com/facebook/react-native/pull/31224)修復）：

  - 當元件禁用時，會停用螢幕閱讀器的點擊功能

## 其他進展

| Status                                  | Number of Issues |
| --------------------------------------- | :--------------: |
| Issues To Do                            |        53        |
| In Progress Issues by the Community     |        8         |
| In Progress Issues by React Native Team |        5         |
| Pull Request in Progress                |        3         |
| Pull Request in Reviews                 |        4         |

## 參與貢獻！

- 新貢獻者應閱讀[貢獻指南](https://github.com/facebook/react-native/blob/master/CONTRIBUTING.md)並瀏覽 React Native GitHub 上 37 個[適合初學者處理的議題](https://github.com/facebook/react-native/issues?q=is%3Aopen+is%3Aissue+label%3A%22Good+first+issue%22+label%3AAccessibility)清單。

- 對需要更多投入的議題感興趣的貢獻者，可造訪[改進 React Native 無障礙功能的專案頁面](https://github.com/facebook/react-native/projects/15)，查看需要其 React Native 專業知識的 GitHub 議題。

- 有意更新 React Native 文件以反映無障礙功能改進的技術寫作者，請參閱 [React Native 文件專案](https://github.com/facebook/react-native-website#-overview)。

- 請將此倡議分享給任何可能提供協助的人！

- 追蹤 React Native 開源無障礙社群經理在 [Twitter](https://twitter.com/alexmarlette) 或 [Facebook](https://www.facebook.com/React-Native-Open-Source-Accessibility-Community-Manager-102732258549941) 上的動態，以獲取最新進展資訊。