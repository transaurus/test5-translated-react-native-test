---
title: Triaging GitHub Issues
---

首先查看標記為["Needs: Triage"標籤](https://github.com/facebook/react-native/issues?q=is%3Aissue+is%3Aopen+label%3A%22Needs%3A+Triage+%3Amag%3A%22)的待分類問題。

- 這是否為針對個別應用程式的程式碼層級求助？是否更適合發佈在Stack Overflow？如果是，請加上「Resolution: For Stack Overflow」標籤。
- 此問題是否正確使用了問題模板？若否，請加上「Needs: Template」標籤。
- 此問題是否提及使用的React Native版本？若否，請加上「Needs: Environment Info」標籤。
- 此問題是否包含Snack連結、程式碼範例或重現問題的步驟清單？若否，請加上「Needs: Repro」標籤。

:::note
我們偶爾會收到不適合GitHub問題追蹤系統的內容。請加上「Type: Invalid」標籤，機器人會自動關閉該問題。
:::

完成上述步驟後，即可開始審視問題內容本身。此問題是否包含對問題的**清晰描述**？

If not, _politely_ ask the issue author to update their issue with the necessary information, and apply a "Needs: Author Feedback" label.

我們始終秉持友善互助的態度，並期待社群每位成員皆能如此。

## 改進問題

若問題已包含所有必要資訊，請思考是否仍有改進空間。格式是否正確？可視需要輕度編輯問題以提升可讀性。

若問題包含未格式化的程式碼區塊，請用三個反引號（```）將其包圍以轉換為Markdown程式碼區塊。

是否有適合的分類標籤可添加？若問題僅影響Android應用，可加上「Platform: Android」標籤；若僅在Windows開發時出現，則可加上「Platform: Windows」標籤。

我們有完整的[標籤清單](https://github.com/facebook/react-native/issues/labels)，請參考並添加適用標籤！

## 處理重複問題

隨著處理經驗累積，您將逐漸熟悉常見問題類型，甚至會發現相同問題被重複回報。

此時可關閉重複問題並留言「Duplicate of #issue」。遵循此慣例後，GitHub會自動標記為重複問題。

## 評估影響層級

接下來需判斷問題嚴重程度。

### 是否可能成為**版本發布阻斷因素**？

此類問題應在一兩週內處理，因其可能阻礙發布協調者產出乾淨的候選版本。

適合此標籤的情況包括導致預提交測試失敗的退化問題。若問題已存在多個版本中（依定義不可能成為RC阻斷因素），則不應標記為發布阻斷。

### 是否導致應用程式**崩潰**？

此類問題會造成React Native意外終止。若未及時修復可能導致不良用戶體驗。

### 是否為**錯誤**？

描述未預期行為的問題。雖需修復但不致阻礙發布流程。即使導致崩潰，若有合理解決方案仍可歸類為一般錯誤。

### 是否為**新手友善問題**？

這些問題不需要對代碼庫有深入的理解和熟悉度。GitHub 會將這些問題展示給有意成為貢獻者的人。請注意，標記為此類型的問題可能不會立即得到修復。