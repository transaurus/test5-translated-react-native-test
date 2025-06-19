---
title: Managing Pull Requests
---

審查一個 pull request 可能需要相當長的時間。在某些情況下，進行審查所需的時間甚至可能比撰寫和提交變更的時間還要多！因此，有必要進行一些初步工作，以確保每個 pull request 都處於適合審查的良好狀態。

一個 pull request 應包含三個主要部分：

- 摘要。這有助於我們理解變更的動機。
- 變更日誌。這有助於我們撰寫版本說明。同時也作為變更的簡要摘要。
- 測試計劃。這可能是 pull request 中最重要的部分。測試計劃應是一個可重現的逐步指南，以便審查者能夠驗證您的變更是否按預期運作。對於使用者可見的變更，附上螢幕截圖或影片也是個好主意。

任何 pull request 都可能需要對 React Native 的某些領域有更深入的理解，而您可能對此並不熟悉。即使您覺得自己不是審查某個 pull request 的合適人選，您仍然可以通過添加標籤或向作者請求更多信息來提供幫助。

## 審查 PRs

Pull Requests 需要使用 GitHub 的審查功能進行審查和批准後才能合併。雖然任何人都可以審查和批准 pull request，但我們通常只有在審查來自[貢獻者](https://github.com/facebook/react-native/blob/main/ECOSYSTEM.md)之一時，才會認為 pull request 已準備好合併。

<!-- alex ignore clearly -->

假設您找到了一個您有信心審查的 pull request。請使用 GitHub 的審查功能，並清晰且禮貌地溝通任何建議的變更。

可以考慮從那些被標記為缺少變更日誌或測試計劃的 pull request 開始。

- [缺少變更日誌的 PRs](https://github.com/facebook/react-native/pulls?utf8=%E2%9C%93&q=is%3Apr+is%3Aopen+label%3A%22Missing+Changelog%22+) - 查看並嘗試通過編輯 PR 來添加變更日誌。完成後，移除「Missing Changelog」標籤。
- [缺少測試計劃的 PRs](https://github.com/facebook/react-native/pulls?q=is%3Apr+label%3A%22Missing+Test+Plan%22+is%3Aclosed) - 打開 pull request 並尋找測試計劃。如果測試計劃看起來足夠，移除「Missing Test Plan」標籤。如果沒有測試計劃，或者看起來不完整，添加一條評論禮貌地請求作者考慮添加測試計劃。

一個 pull request 必須通過所有測試才能合併。這些測試會在每次提交到 `main` 分支和 pull request 時運行。幫助我們準備好 pull request 進行審查的一個快速方法是[搜尋那些未能通過預提交測試的 pull requests](https://github.com/facebook/react-native/pulls?utf8=%E2%9C%93&q=is%3Apr+is%3Aopen+label%3A%22CLA+Signed%22+status%3Afailure+)，並判斷是否需要進行修訂。失敗的測試通常會列在討論串的底部，標記為「Some checks were not successful」。

- 快速查看 [main 分支的最新測試運行結果](https://circleci.com/gh/facebook/react-native/tree/main)。`main` 是否顯示為綠色（通過）？如果是：
  - 測試失敗是否可能與此拉取請求中的變更有關？請要求作者進行調查。
  - 即使 `main` 目前顯示為綠色，仍需考慮拉取請求中的提交可能是基於 `main` 分支發生故障時期的某個提交。若您認為可能存在此情況，請要求作者將其變更重新基於最新的 `main` 分支進行重訂（rebase），以納入後續可能已修復的內容。
- 若 `main` 顯示為故障狀態，請查找 [標記為「CI Test Failure」的議題](https://github.com/facebook/react-native/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3A%22%E2%9D%8CCI+Test+Failure%22+)。
  - 若找到與 `main` 故障相關的議題，請返回拉取請求並感謝作者提出變更，同時說明測試失敗可能與其特定變更無關（別忘了附上 CI Test Failure 議題連結，這將幫助作者了解何時可重新執行測試）。
  - 若未找到描述 `main` 故障的現有 CI Test Failure 議題，請提交新議題並使用「CI Test Failure」標籤通知其他人 `main` 處於故障狀態（參考 [此議題](https://github.com/facebook/react-native/issues/23108) 範例）。

## 我們如何排定拉取請求的優先級

Meta 的 React Native 團隊成員致力於快速審查拉取請求，多數 PR 會在一週內獲得回覆。

## 拉取請求如何被合併？

React Native 的 GitHub 倉庫實際上是 Meta 單體倉庫（monorepo）中某個子目錄的鏡像。因此，拉取請求不會以傳統方式合併，而是需要透過 [「diff」](https://www.phacility.com/phabricator/differential/) 形式導入 Meta 內部的程式碼審查系統。

導入後，變更將通過一系列測試。部分測試為合併阻斷項（land-blocking），意味著必須通過這些測試才能合併 diff 內容。Meta 始終從 `main` 分支運行 React Native，某些變更可能需要 Facebook 員工在合併前為您的拉取請求附加內部變更。例如，若您重新命名模組名稱，必須在同一變更中更新 Facebook 內部所有呼叫點才能合併。若 diff 成功落地，變更最終將由 [ShipIt](https://github.com/facebook/fbshipit) 以單一提交形式同步回 GitHub。

Meta 員工使用 GitHub 的客製化瀏覽器擴充功能，能以兩種方式導入拉取請求：可「落地至 fbsource」（landed to fbsource），表示將自動導入並核准產生的 diff，若無失敗則變更最終會同步回 `main`；亦可「導入至 Phabricator」（imported to Phabricator），表示變更將複製到內部 diff，需進一步審查核准後才能落地。

<figure>
  <img src="/img/importing-pull-requests.png" />
  <figcaption>Screenshot of the custom browser extension. The button "Import to fbsource" is used to import a Pull Request internally.</figcaption>
</figure>

## 機器人

審查與處理拉取請求時，您可能會遇到幾個 GitHub 機器人帳號留下的評論。這些機器人旨在協助拉取請求審查流程，詳見 [機器人參考文件](/contributing/bots-reference)。

## 拉取請求標籤

- `Merged`：標記於已關閉的 PR，表示其變更已納入核心倉庫。此標籤必要是因為拉取請求不會直接在 GitHub 上合併，而是將 PR 變更以補丁形式導入並排入程式碼審查佇列。核准後，將這些變更應用於 Meta 內部單體倉庫的結果會以新提交同步至 GitHub。GitHub 不會將該提交歸因於原始 PR，因此需要此標籤傳達 PR 真實狀態。
- `Blocked on FB`：PR 已導入，但變更尚未應用。