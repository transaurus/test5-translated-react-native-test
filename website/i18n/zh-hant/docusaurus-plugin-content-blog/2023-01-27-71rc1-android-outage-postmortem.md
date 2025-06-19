---
title: 'React Native 0.71-RC0 Android outage postmortem'
authors: [cortinico, kelset]
tags: [engineering]
date: 2023-01-27
---

現在 0.71 版本[已正式發布](/blog/2023/01/12/version-071)，我們希望分享關於 2022 年 11 月 4 日發布首個 React Native 0.71 候選版本時，導致所有 React Native 版本 Android 構建中斷事件的關鍵資訊。

參與處理此事件的貢獻者近期參加了事後檢討會議，詳細討論事件經過、從中學到的教訓，以及我們將採取哪些行動來避免未來發生類似的中斷情況。

<!--truncate-->

## 事件經過

2022 年 11 月 4 日，我們在多個公開儲存庫發布了 React Native 的 `0.71.0-rc0` 版本，這是 0.71 的首個候選版本。

此候選版本中的一項重大變更是將構建產物發布至 Maven Central，而非從原始碼構建，以改善構建時間。更多技術細節可參閱 [RFC#508](https://github.com/react-native-community/discussions-and-proposals/pull/508) 與[相關討論](https://github.com/reactwg/react-native-new-architecture/discussions/105)。

遺憾的是，由於我們從模板生成新專案的方式，這導致所有使用舊版本的 Android 用戶構建失敗——因為系統會開始下載 `0.71.0-rc0` 的新構建產物，而非專案實際使用的版本（例如 `0.68.0`）。

## 事件原因

React Native 模板提供的 `build.gradle` 檔案包含以下 React Native Android 函式庫依賴項：
`implementation("com.facebook.react:react-native:+")`。

關鍵在於，依賴項中的 `+` 符號（屬於 [Gradle 動態版本](https://docs.gradle.org/current/userguide/dynamic_versions.html)）會指示 Gradle 取得最高可用版本的 React Native。使用 Gradle 動態版本被視為反模式，因其會導致構建結果難以重現。

我們已知動態版本可能引發的問題，因此在 `0.71` 版本中清理了新應用模板，移除了所有 `+` 依賴項。但使用舊版 React Native 的用戶仍在使用含 `+` 的版本。

這導致 React Native 版本低於 `0.71.0-rc.0` 的構建會向所有儲存庫查詢最高可用版本。由於新發布的 0.71.0-rc.0 成為 Maven Central 上的最高版本，低於此版本的構建開始錯誤使用 0.71.0-rc.0 的構建產物。本地構建版本（如 `0.68.0`）與 Maven Central 產物版本（`0.71.0-rc.0`）的不匹配導致構建失敗。

更多技術細節可參閱 [GitHub 議題](https://github.com/facebook/react-native/issues/35210)。

## 緩解與解決措施

我們在 11 月 4 日確認問題後，社群迅速找到並分享了手動解決方案——透過指定確切版本號來修正錯誤。

隨後在 11 月 5 日至 6 日的週末期間，發布團隊為所有舊版 React Native（回溯至 0.63）發布修補版本，自動套用修正方案，讓用戶能更新至已修復的版本。

同時，我們[聯繫 Sonatype](https://issues.sonatype.org/browse/OSSRH-86006) 要求移除有問題的構建產物。

11 月 8 日當所有構建產物從 Maven Central 完全移除後，事件正式解決。

## 事件時間軸

_本節為事件簡要時間軸，所有時間均為 GMT/UTC +0 時區_

- 11月4日 下午5:06: [發布0.71-RC0版本](https://github.com/facebook/react-native/releases/tag/v0.71.0-rc.0)。
- 11月4日 下午6:20: [首次收到建構問題報告](https://github.com/facebook/react-native/issues/35204)。
- 11月4日 下午7:45: [社群確認問題根源](https://github.com/facebook/react-native/issues/35204#issuecomment-1304090948)。
- 11月4日 晚上9:39: [臨時解決方案公布，Expo團隊](https://github.com/facebook/react-native/issues/35204#issuecomment-1304281740)為所有用戶部署修復。
- 11月5日 凌晨3:04: [開立新issue通報狀態與應變措施](https://github.com/facebook/react-native/issues/35210)。
- 11月6日 下午4:11: [向SonaType提交](https://issues.sonatype.org/browse/OSSRH-86006?focusedCommentId=1216303&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-1216303)移除問題構件的請求。
- 11月6日 下午4:40: [@reactnative官方推特](https://twitter.com/reactnative/status/1589296764678705155)首次發文確認問題並附上issue連結。
- 11月6日 晚上7:05: 決議回溯修復至React Native 0.63版本。
- 11月7日 凌晨12:47: 最後一個修復版本[0.63.5](https://github.com/facebook/react-native/releases/tag/v0.63.5)發布。
- 11月8日 晚上8:04: Maven Central上的問題構件[全數移除](https://issues.sonatype.org/browse/OSSRH-86006?focusedCommentId=1216303&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-1216303)。
- 11月10日 上午11:51: [事件issue正式關閉](https://github.com/facebook/react-native/issues/35210#issuecomment-1310170361)。

## 經驗教訓

雖然觸發此事件的條件自React Native 0.12.0以來就已存在，但我們將強化未來開發與發布React Native的基礎架構。以下是我們從中學到的教訓，以及為加速未來應變所採取的具體行動方案。

### 事件應對策略

本次事件暴露了我們在處理React Native開源議題時的事件應對策略缺口。

社群在2小時內迅速找到臨時解決方案。由於缺乏對問題影響範圍的可視性，加上修復舊版所需的複雜度，我們當時依賴受影響者自行在GitHub issue發現解決方案。

我們耗時48小時才認知到問題的廣泛性，意識到不能僅依賴開發者自行查找GitHub issue。必須優先實施能自動修復專案的主動緩解措施。

我們將重新評估「依賴開發者手動應用解決方案」與「部署自動修復」的決策流程，並研究如何更即時掌握生態系統健康狀態的方案。

### 版本支援政策

根據[rn-versions工具](https://rn-versions.github.io/)顯示，為涵蓋事件發生時90%以上的React Native開發者，我們必須回溯修復至0.63版本。

我們認為這歸因於React Native歷來艱難的升級體驗。目前正著手改善升級流程，以緩解生態系統碎片化現象。

新版React Native的發布絕不應影響舊版用戶，我們為此造成的工作流程中斷深表歉意。

同樣地，我們也想強調保持依賴套件和React Native版本更新的重要性，以確保能受益於我們所引入的改進和安全措施。這次事件發生時，官方[版本支援政策](https://github.com/reactwg/react-native-releases#releases-support-policy)正在制定中，尚未對外公布或強制執行。

未來，我們將透過官方溝通管道明確傳達支援政策，並考慮[在npm上標記舊版React Native為棄用](https://docs.npmjs.com/deprecating-and-undeprecating-packages-or-package-versions)。

### 改進第三方函式庫的測試與最佳實踐

這次事件突顯了強化發布測試流程與完善第三方函式庫指導方針的重要性。

在測試方面，由於缺乏現行穩定版發布的自動化測試機制，向下兼容發布至`0.63.x`版本面臨重大挑戰。我們認知到發布與測試基礎設施的關鍵性，未來將持續投入資源強化相關建設。

具體而言，我們現在鼓勵並支持第三方函式庫將測試納入[React Native發布流程](https://github.com/reactwg/react-native-releases/discussions/41)。同時也在[核心貢獻者Discord伺服器](https://github.com/facebook/react-native/blob/main/ECOSYSTEM.md#core-contributors)新增專屬頻道與角色。

此外，我們與[create-react-native-library](https://github.com/callstack/react-native-builder-bob/tree/main/packages/create-react-native-library)的維護者Callstack展開深度合作，改進函式庫模板以確保遵循所有與React Native專案整合的必要最佳實踐。新版`create-react-native-library`已完全兼容0.71專案，同時保持向後兼容性。

## 總結

我們要為此次事件對全球開發者工作流程造成的干擾致歉。如前述內容所示，我們已開始採取行動強化基礎建設，後續還會有更多改進措施。

希望透過分享這些洞察，能幫助大家更理解本次事件，並將我們的經驗轉化為您工具與專案中的最佳實踐。

最後，我們要再次感謝Sonatype協助移除有問題的構件，以及日夜不懈處理此事件的社群成員與發布團隊。