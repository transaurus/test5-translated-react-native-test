---
title: Toward Better Documentation
author: Kevin Lacker
authorTitle: Engineering Manager at Facebook
authorURL: 'https://twitter.com/lacker'
authorImageURL: 'https://www.gravatar.com/avatar/9b790592be15d4f55a5ed7abb5103304?s=128'
authorTwitter: lacker
tags: [announcement]
---

優質的開發者體驗部分來自於優質的文件。要打造好的文件需要投入許多心力——理想的文件應當簡潔、實用、準確、完整且令人愉悅。近期我們根據您的反饋努力改進文件，並希望分享一些我們所做的改進。

## 內嵌範例

When you learn a new library, a new programming language, or a new framework, there's a beautiful moment when you first write a bit of code, try it out, see if it works... and it _does_ work. You created something real. We wanted to put that visceral experience right into our docs. Like this:

```ReactNativeWebPlayer
import React, { Component } from 'react';
import { AppRegistry, Text, View } from 'react-native';

class ScratchPad extends Component {
  render() {
    return (
      <View style={{flex: 1}}>
        <Text style={{fontSize: 30, flex: 1, textAlign: 'center'}}>
          Isn't this cool?
        </Text>
        <Text style={{fontSize: 100, flex: 1, textAlign: 'center'}}>
          👍
        </Text>
      </View>
    );
  }
}

AppRegistry.registerComponent('ScratchPad', () => ScratchPad);
```

我們認為這些內嵌範例（使用[`react-native-web-player`](https://github.com/dabbott/react-native-web-player)模組，並感謝[Devin Abbott](https://twitter.com/devinaabbott)的協助）是學習React Native基礎的絕佳方式，並已更新[新手教學](/docs/tutorial)盡可能採用此形式。試試看吧——如果您曾好奇修改一小段範例代碼會發生什麼，這將是探索的好方法。此外，若您正在開發工具並希望在自己的網站展示即時React Native範例，[`react-native-web-player`](https://github.com/dabbott/react-native-web-player)能簡化此流程。

核心模擬引擎由[Nicolas Gallagher](https://twitter.com/necolas)的[`react-native-web`](https://github.com/necolas/react-native-web)專案提供，該專案能在網頁上顯示如`Text`和`View`等React Native組件。如果您有興趣建構共享大量代碼的行動與網頁體驗，請查看[`react-native-web`](https://github.com/necolas/react-native-web)。

## 更完善的指南

在React Native某些部分存在多種實作方式，我們收到反饋認為需要提供更明確的指引。

我們新增了[導航指南](/docs/navigation)，比較不同方案並建議使用方式——包括`Navigator`、`NavigatorIOS`、`NavigationExperimental`。中期目標是改進並整合這些介面，短期則希望更好的指南能減輕您的負擔。

另新增了[觸控處理指南](/docs/handling-touches)，解釋建立按鈕式介面的基礎知識，並簡述處理觸控事件的各種方法。

另一個重點是Flexbox，包含[使用Flexbox佈局](/docs/flexbox)教學、[控制組件尺寸](/docs/height-and-width)指南，以及雖不炫酷但實用的[React Native佈局屬性完整列表](/docs/layout-props)。

## 入門指引

當您在電腦上建立React Native開發環境時，確實需要安裝和配置許多項目。雖然難以讓安裝過程變得有趣，但我們至少能使其快速且無痛。

我們建立了[新版入門流程](/docs/next/getting-started)，讓您預先選擇開發作業系統與行動作業系統，提供集中化的設定指引。我們也實際測試安裝流程以確保每個步驟都有明確建議。經過同事測試後，我們確信這是項改進。

同時強化了[將React Native整合至現有應用程式](/docs/integration-with-existing-apps)的指南。許多大型應用程式（如Facebook主應用）實際採用混合開發模式。我們希望此指南能幫助更多人實踐此模式。

## 需要您的協助

您的反饋讓我們知道應該優先處理哪些事項。我知道有些人讀完這篇部落格文章後會想：「更好的文件？哼！X 的文件還是很糟糕！」這很好——我們需要這種能量。根據反饋的類型，提供反饋的最佳方式也有所不同。

如果您在文件中發現錯誤，例如描述不準確或程式碼無法運作，請[提交問題](https://github.com/facebook/react-native/issues)。標記為「Documentation」，以便更容易將其路由給正確的人員。

如果沒有具體的錯誤，但文件中的某些內容從根本上令人困惑，這不太適合提交 GitHub 問題。相反，請在 [Canny](https://react-native.canny.io/feature-requests) 上發布有關需要幫助的文件區域的意見。這有助於我們在進行指南編寫等更一般性工作時確定優先順序。

感謝您閱讀至此，也感謝您使用 React Native！