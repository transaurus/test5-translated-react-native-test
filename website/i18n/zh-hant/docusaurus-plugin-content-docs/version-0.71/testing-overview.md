---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫的擴展，未預期的小錯誤和邊緣案例可能引發更大的故障。這些錯誤會導致糟糕的使用者體驗，最終造成業務損失。預防脆弱程式設計的方法之一，是在將程式碼發布前進行測試。

本指南將涵蓋多種自動化測試方法，從靜態分析到端對端測試，確保您的應用程式如預期運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為何需要測試

人類難免會犯錯。測試的重要性在於幫助發現這些錯誤並驗證程式碼是否正常運作。更重要的是，測試能確保未來在新增功能、重構現有程式碼或升級專案主要依賴項時，程式碼仍能持續運作。

測試的價值超乎想像。修正程式錯誤的最佳方法之一，就是撰寫一個能暴露該錯誤的失敗測試。當您修正錯誤後重新執行測試，若測試通過，代表錯誤已修復且不會再次引入程式碼庫。

測試也能作為新團隊成員的技術文件。對於初次接觸程式碼庫的人來說，閱讀測試能幫助他們理解現有程式碼的運作方式。

最後但同樣重要的是，自動化測試越多，手動<abbr title="Quality Assurance">QA</abbr>的時間就越少，能釋放寶貴時間。

## 靜態分析

提升程式碼品質的第一步是使用靜態分析工具。靜態分析會在撰寫程式碼時檢查錯誤，但不會實際執行這些程式碼。

- **Linters** 分析程式碼以捕捉常見錯誤（如未使用的程式碼），幫助避免陷阱，並標記違反風格指南的行為（例如使用製表符而非空格，或反之，取決於您的配置）。
- **型別檢查** 確保傳遞給函式的結構符合其設計預期，例如防止將字串傳遞給預期接收數字的計數函式。

React Native 預設配置了兩種工具：[ESLint](https://eslint.org/) 用於程式碼檢查，[TypeScript](typescript) 用於型別檢查。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造過程為例——在模型首次起飛展示所有複雜系統協同運作前，會先測試各個零件以確保其安全性和功能性。例如，機翼會進行極端負載下的彎曲測試；引擎零件會測試耐用性；擋風玻璃會模擬鳥擊測試。

軟體開發也是如此。與其將整個程式寫在一個龐大檔案中，不如將程式碼拆分為多個小型模組，這樣能比測試整個組裝成品更徹底。因此，撰寫可測試的程式碼與撰寫乾淨、模組化的程式碼息息相關。

要讓應用程式更易測試，可先將應用的視圖部分（React 元件）與業務邏輯和應用狀態分離（無論您使用 Redux、MobX 或其他解決方案）。如此一來，業務邏輯測試（不應依賴 React 元件）就能獨立於主要負責渲染 UI 的元件之外！

理論上，您甚至可以將所有邏輯和資料獲取移出元件，讓元件專注於渲染。狀態完全獨立於元件，應用邏輯甚至能在沒有任何 React 元件的情況下運作！

> 我們鼓勵您透過其他學習資源進一步探索可測試程式碼的主題。

## 撰寫測試

撰寫完可測試的程式碼後，接下來就是實際撰寫測試！React Native 的預設模板內建 [Jest](https://jestjs.io) 測試框架，包含針對此環境量身打造的預設配置，讓您無需調整設定或立即處理模擬（稍後會[詳述模擬](#mocking)）。您可以使用 Jest 撰寫本指南提到的所有測試類型。

> 如果你採用測試驅動開發（TDD），實際上你會先寫測試！這樣一來，程式碼的可測試性自然就具備了。

### 測試結構設計

測試應該簡潔，且理想情況下只測試單一功能。以下是一個使用 Jest 撰寫的單元測試範例：

```js
it('given a date in the past, colorForDueDate() returns red', () => {
  expect(colorForDueDate('2000-10-20')).toBe('red');
});
```

測試描述由傳遞給 [`it`](https://jestjs.io/docs/en/api#testname-fn-timeout) 函式的字串定義。請仔細撰寫描述以明確測試內容，並盡量涵蓋以下要素：

1. **Given** - 前置條件
2. **When** - 被測函式執行的動作
3. **Then** - 預期結果

此模式亦稱為 AAA（Arrange, Act, Assert）。

Jest 提供 [`describe`](https://jestjs.io/docs/en/api#describename-fn) 函式協助組織測試結構。使用 `describe` 將同一功能的測試歸為一組，必要時可嵌套描述區塊。其他常用函式如 [`beforeEach`](https://jestjs.io/docs/en/api#beforeeachfn-timeout) 或 [`beforeAll`](https://jestjs.io/docs/en/api#beforeallfn-timeout)，可用於設置測試對象。詳見 [Jest API 文件](https://jestjs.io/docs/en/api)。

若測試步驟繁多或有多項預期結果，建議拆分成多個小測試。同時確保測試間完全獨立——每個測試都應能單獨執行，且測試順序不影響結果。當所有測試一起執行時，前一個測試不得改變後續測試的輸出。

Lastly, as developers we like when our code works great and doesn't crash. With tests, this is often the opposite. Think of a failed test as of a _good thing!_ When a test fails, it often means something is not right. This gives you an opportunity to fix the problem before it impacts the users.

## 單元測試

單元測試涵蓋程式碼的最小單位，例如獨立函式或類別。

當被測對象存在依賴項時，通常需要進行模擬（mock），詳見下節說明。

單元測試的優勢在於快速撰寫與執行，能即時反饋測試結果。Jest 還提供 [Watch 模式](https://jestjs.io/docs/en/cli#watch)，可持續運行與編輯程式碼相關的測試。

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### 模擬（Mocking）

當被測對象有外部依賴時，常需進行「模擬」——即用自訂實現替換依賴項。

> 原則上，測試中使用真實對象優於模擬對象，但某些情境無法避免。例如：當 JS 單元測試依賴 Java 或 Objective-C 寫的原生模組時。

假設你正在開發顯示城市天氣的應用，並使用外部天氣服務。若服務回報下雨，你需顯示雨雲圖示。測試時不應直接呼叫該服務，因為：

- 網路請求會導致測試變慢且不穩定
- 服務每次可能回傳不同數據
- 第三方服務可能在關鍵時刻離線！

此時可提供服務的模擬實現，有效取代數千行程式碼與聯網溫度計！

> Jest 內建[模擬支援](https://jestjs.io/docs/en/mock-functions#mocking-modules)，從函式級到模組級皆可模擬。

## 整合測試

在開發大型軟體系統時，各個組件需要相互協作。在單元測試中，若某個單元依賴於另一個單元，有時會透過模擬（mock）該依賴項來替換成虛擬版本。

整合測試則是將實際的獨立單元（如同在應用程式中）組合起來共同測試，以確保它們的協作符合預期。這並非表示此處完全不需要模擬——例如仍需模擬與天氣服務的通訊——但使用頻率會遠低於單元測試。

> Please note that the terminology around what integration testing means is not always consistent. Also, the line between what is a unit test and what is an integration test may not always be clear. For this guide, your test falls into "integration testing" if it:
>
> - Combines several modules of your app as described above
> - Uses an external system
> - Makes a network call to other application (such as the weather service API)
> - Does any kind of file or database <abbr title="Input/Output">I/O</abbr>

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 元件測試

React元件負責渲染應用界面，使用者將直接與其輸出互動。即使業務邏輯測試覆蓋率高且正確，若缺乏元件測試，仍可能交付有缺陷的UI。元件測試可歸類於單元或整合測試，但由於其為React Native核心，我們將獨立說明。

測試React元件時，需關注兩個層面：

- Interaction: to ensure the component behaves correctly when interacted with by a user (eg. when user presses a button)
- Rendering: to ensure the component render output used by React is correct (eg. the button's appearance and placement in the UI)

舉例來說，若按鈕設有`onPress`監聽器，需同時測試其外觀呈現與點擊事件的處理邏輯。

可用測試工具包括：

- React官方[Test Renderer](https://reactjs.org/docs/test-renderer.html)：可在不依賴DOM或原生環境下，將元件渲染為純JavaScript物件
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)：基於Test Renderer擴增，提供後文將說明的`fireEvent`與`query`API

> Component tests are only JavaScript tests running in Node.js environment. They do _not_ take into account any iOS, Android, or other platform code which is backing the React Native components. It follows that they cannot give you a 100% confidence that everything works for the user. If there is a bug in the iOS or Android code, they will not find it.

<img src="/docs/assets/p_tests-component.svg" alt=" " />

### 測試使用者互動

Aside from rendering some UI, your components handle events like `onChangeText` for `TextInput` or `onPress` for `Button`. They may also contain other functions and event callbacks. Consider the following example:

```tsx
function GroceryShoppingList() {
  const [groceryItem, setGroceryItem] = useState('');
  const [items, setItems] = useState<string[]>([]);

  const addNewItemToShoppingList = useCallback(() => {
    setItems([groceryItem, ...items]);
    setGroceryItem('');
  }, [groceryItem, items]);

  return (
    <>
      <TextInput
        value={groceryItem}
        placeholder="Enter grocery item"
        onChangeText={text => setGroceryItem(text)}
      />
      <Button
        title="Add the item to list"
        onPress={addNewItemToShoppingList}
      />
      {items.map(item => (
        <Text key={item}>{item}</Text>
      ))}
    </>
  );
}
```

測試使用者互動時，應從使用者視角出發：頁面顯示什麼？互動後如何變化？

基本原則：優先基於使用者可感知的元素進行驗證：

- make assertions using rendered text or [accessibility helpers](https://reactnative.dev/docs/accessibility#accessibility-properties)

反之應避免：

- 對元件props或state建立斷言
- 使用testID查詢

避免測試實作細節（如props或state）。此類測試雖可運行，但未對齊使用者互動模式，且重構時易失效（例如重命名或改用Hooks改寫class元件時）。

> React class components are especially prone to testing their implementation details such as internal state, props or event handlers. To avoid testing implementation details, prefer using function components with Hooks, which make relying on component internals _harder_.

像 [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 這類元件測試函式庫，透過精心選擇提供的 API 來促進撰寫以使用者為中心的測試。以下範例使用 `fireEvent` 方法的 `changeText` 和 `press` 來模擬使用者與元件的互動，以及查詢函式 `getAllByText` 來找出渲染輸出中匹配的 `Text` 節點。

```tsx
test('given empty GroceryShoppingList, user can add an item to it', () => {
  const {getByPlaceholder, getByText, getAllByText} = render(
    <GroceryShoppingList />,
  );

  fireEvent.changeText(
    getByPlaceholder('Enter grocery item'),
    'banana',
  );
  fireEvent.press(getByText('Add the item to list'));

  const bananaElements = getAllByText('banana');
  expect(bananaElements).toHaveLength(1); // expect 'banana' to be on the list
});
```

這個範例並非測試呼叫函式時某些狀態如何改變，而是測試當使用者在 `TextInput` 中變更文字並按下 `Button` 時會發生什麼！

### 測試渲染輸出

[快照測試](https://jestjs.io/docs/en/snapshot-testing) 是由 Jest 啟用的一種進階測試方式。它是一個非常強大且底層的工具，因此使用時需要格外注意。

「元件快照」是由 Jest 內建的自訂 React 序列化器所建立的 JSX 風格字串。這個序列化器讓 Jest 能將 React 元件樹轉換為人類可讀的字串。換句話說：元件快照是測試執行期間 _生成_ 的元件渲染輸出的文字表示。它可能看起來像這樣：

```tsx
<Text
  style={
    Object {
      "fontSize": 20,
      "textAlign": "center",
    }
  }>
  Welcome to React Native!
</Text>
```

使用快照測試時，通常會先實作你的元件，然後執行快照測試。快照測試接著會建立一個快照，並將其作為參考快照儲存到你的程式庫中的一個檔案裡。**該檔案會被提交並在程式碼審查時檢查**。未來對元件渲染輸出的任何變更都會改變其快照，導致測試失敗。此時你需要更新儲存的參考快照才能使測試通過。該變更同樣需要被提交並審查。

快照有幾個弱點：

- 對於開發者或審查者來說，可能難以判斷快照中的變更是有意為之還是錯誤的證據。特別是大型快照可能很快變得難以理解，其附加價值也會降低。
- 當快照建立時，該時間點的快照會被視為正確的——即使渲染輸出實際上是有問題的。
- 當快照失敗時，很容易直接使用 Jest 的 `--updateSnapshot` 選項更新快照，而沒有適當檢查變更是否預期。因此需要一定的開發者紀律。

快照本身並不能確保你的元件渲染邏輯正確，它們主要擅長防範意外變更，以及檢查測試中的 React 樹下的元件是否收到預期的 props（樣式等）。

我們建議你只使用小型快照（參見 [`no-large-snapshots` 規則](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)）。如果你想測試兩個 React 元件狀態之間的 _變更_，可以使用 [`snapshot-diff`](https://github.com/jest-community/snapshot-diff)。如有疑問，建議優先使用前一段所述的明確期望值。

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## 端對端測試

在端對端（E2E）測試中，你會在裝置（或模擬器／模擬器）上從使用者的角度驗證你的應用程式是否如預期運作。

這需要以發布配置建置你的應用程式，並針對其執行測試。在 E2E 測試中，你不再考慮 React 元件、React Native API、Redux 儲存或任何業務邏輯。這並非 E2E 測試的目的，且在 E2E 測試期間這些甚至無法存取。

相反地，E2E 測試函式庫讓你能夠找到並控制應用程式畫面中的元素：例如，你可以 _實際_ 點擊按鈕或在 `TextInputs` 中輸入文字，就像真實使用者一樣。接著你可以對應用程式畫面中是否存在特定元素、是否可見、包含什麼文字等進行斷言。

E2E 測試能為你提供應用程式某部分正在運作的最髙可信度。其權衡取捨包括：

- 撰寫這類測試比其他類型的測試更耗時
- 執行速度較慢
- 更容易出現不穩定性（「不穩定」測試是指隨機通過或失敗，而程式碼並未變更的測試）

請盡量為應用程式的關鍵部分加入端對端測試：身份驗證流程、核心功能、支付等。對於非關鍵部分，則使用執行速度更快的 JavaScript 測試。加入的測試越多，信心程度就越高，但維護和執行測試所需的時間也會增加。請權衡利弊，選擇最適合您的方式。

目前有數種端對端測試工具可供選擇：在 React Native 社群中，[Detox](https://github.com/wix/detox/) 是一個熱門框架，因為它是專為 React Native 應用程式設計的。另一個在 iOS 和 Android 應用程式領域中廣受歡迎的函式庫是 [Appium](http://appium.io/)。

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## 總結

希望您喜歡閱讀本指南並有所收穫。測試應用程式的方法有很多種，一開始可能難以決定使用哪種方式。不過，我們相信一旦您開始為出色的 React Native 應用程式加入測試，一切就會變得清晰。還在等什麼呢？快來提高測試覆蓋率吧！

### 相關連結

- [React 測試概述](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest 文件](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](http://appium.io/)

---

_本指南由 [Vojtech Novak](https://twitter.com/vonovak) 原創並完整貢獻。_