---
title: 'A Monthly Release Cadence: Releasing December and January RC'
author: Eric Vicenti
authorTitle: Engineer at Facebook
authorURL: 'https://twitter.com/EricVicenti'
authorImageURL: 'https://secure.gravatar.com/avatar/077ad5372b65567fe952a99f3b627048?s=128'
authorTwitter: EricVicenti
tags: [announcement]
---

在React Native推出後不久，我們開始每兩週發布一次版本，以幫助社群採用新功能，同時保持生產環境的版本穩定。在Facebook內部，我們必須每兩週為生產iOS應用穩定程式碼庫，因此我們決定以相同的節奏發布開源版本。現在，許多Facebook應用每週發布一次，尤其是Android版本。由於我們每週從主分支發布，我們需要保持其相當穩定。因此，雙週發布節奏甚至對內部貢獻者也不再有益。

我們經常聽到社群的意見，認為發布速度太快難以跟上。像[Expo](https://expo.io/)這樣的工具不得不跳過每隔一個版本，以管理版本的快速變更。因此，很明顯雙週發布並未很好地服務社群。

### 現在改為每月發布

我們很高興宣布新的每月發布節奏，以及2016年12月的發布版本`v0.40`，該版本在上個月已經穩定並準備好供大家採用。（請確保[更新iOS原生模組中的標頭](https://github.com/facebook/react-native/releases/tag/v0.40.0)）。

儘管可能會因避免週末或處理突發問題而有所變動，但現在您可以預期每個月的第一天會有發布候選版本，並在月底正式發布。

### 使用當月版本以獲得最佳支援

一月的發布候選版本已經準備好供試用，您可以[在此查看新功能](https://github.com/facebook/react-native/releases/tag/v0.41.0-rc.0)。

要了解即將到來的變更並向React Native貢獻者提供更好的反饋，請盡可能使用當月的發布候選版本。每個版本在月底發布時，其包含的變更已經在Facebook的生產應用中運行超過兩週。

您可以使用新的[react-native-git-upgrade](/blog/2016/12/05/easier-upgrades)命令輕鬆升級您的應用：

```
npm install -g react-native-git-upgrade
react-native-git-upgrade 0.41.0-rc.0
```

我們希望這種更簡單的方法能讓社群更容易追蹤React Native的變更，並盡快採用新版本！

（感謝[Martin Konicek](https://github.com/mkonicek)提出這個計劃，以及[Mike Grabowski](https://github.com/grabbou)使其實現）