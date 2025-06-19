---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫的擴展，未預期的小錯誤和邊緣案例可能引發更大的故障。錯誤會導致糟糕的使用者體驗，最終造成業務損失。防止脆弱程式設計的方法之一，是在發布前對程式碼進行測試。

本指南將涵蓋多種自動化測試方法，從靜態分析到端到端測試，確保您的應用程式如預期運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為什麼要測試

人類難免會犯錯。測試的重要性在於幫助發現這些錯誤並驗證程式碼是否正常運作。更重要的是，測試能確保未來新增功能、重構現有程式碼或升級專案主要依賴項時，程式碼仍能正常運作。

測試的價值超乎想像。修復程式錯誤的最佳方法之一，就是撰寫一個能暴露該錯誤的失敗測試。當你修復錯誤後重新執行測試，若測試通過，代表錯誤已修復且不會再被引入程式碼庫。

測試也能作為新團隊成員的技術文件。對於初次接觸程式碼庫的人，閱讀測試能幫助他們理解現有程式碼的運作方式。

最後，自動化測試越多，手動<abbr title="Quality Assurance">QA</abbr>的時間就越少，能釋放寶貴時間。

## 靜態分析

提升程式碼品質的第一步是使用靜態分析工具。靜態分析會在撰寫程式碼時檢查錯誤，但不會實際執行這些程式碼。

- **Linters** 分析程式碼以捕捉常見錯誤（如未使用的程式碼），幫助避免陷阱，並標記違反風格指南的行為（例如使用製表符而非空格，或反之，取決於你的配置）。
- **型別檢查** 確保傳遞給函式的結構符合函式設計的預期，例如防止將字串傳遞給預期接收數字的計數函式。

React Native 預設配置了兩種工具：[ESLint](https://eslint.org/) 用於程式碼檢查，[Flow](https://flow.org/en/docs/) 用於型別檢查。你也可以使用 [TypeScript](typescript)，這是一種編譯為純 JavaScript 的型別化語言。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造過程為例——在模型首次起飛展示所有複雜系統協同運作前，會先測試各個零件以確保其安全性和功能。例如，機翼會在極端負載下進行彎曲測試；引擎零件會測試耐用性；擋風玻璃會模擬鳥擊測試。

軟體開發也是如此。與其將整個程式寫在一個龐大的檔案中，不如將程式碼拆分為多個小型模組，這樣能比測試整個組裝後的程式進行更徹底的測試。因此，撰寫可測試的程式碼與撰寫乾淨、模組化的程式碼息息相關。

為了讓應用程式更易測試，首先將應用的視圖部分（React 元件）與業務邏輯和應用狀態分離（無論你使用 Redux、MobX 或其他解決方案）。這樣一來，業務邏輯測試（不應依賴 React 元件）就能獨立於元件本身，而元件的主要職責是渲染應用程式的 UI！

理論上，你可以將所有邏輯和資料獲取從元件中移出。這樣一來，元件將專注於渲染。你的狀態將完全獨立於元件。應用程式的邏輯甚至可以在沒有任何 React 元件的情況下運作！

> 我們鼓勵你進一步探索其他學習資源中關於可測試程式碼的主題。

## 撰寫測試

在撰寫可測試的程式碼後，接下來就是實際編寫測試！React Native 的預設模板內建了 [Jest](https://jestjs.io) 測試框架，其中包含針對此環境量身打造的預設配置，讓您無需立即調整設定和模擬(mock)就能開始工作——稍後會詳細說明[模擬機制](#mocking)。您可以使用 Jest 來編寫本指南中提到的所有測試類型。

> 若採用測試驅動開發(Test-Driven Development)，您實際上會先寫測試！這樣能確保程式碼本身具備可測試性。

### 測試結構設計

測試應該簡潔且最好只驗證單一功能。以下是用 Jest 編寫的單元測試範例：

```js
it('given a date in the past, colorForDueDate() returns red', () => {
  expect(colorForDueDate('2000-10-20')).toBe('red');
});
```

測試描述文字需傳入 [`it`](https://jestjs.io/docs/en/api#testname-fn-timeout) 函式，請務必清晰說明測試內容。建議涵蓋以下要素：

1. **Given** - 前置條件
2. **When** - 被測函式執行的動作
3. **Then** - 預期結果

此模式也稱為 AAA 原則（Arrange, Act, Assert）。

Jest offers [`describe`](https://jestjs.io/docs/en/api#describename-fn) function to help structure your tests. Use `describe` to group together all tests that belong to one functionality. Describes can be nested, if you need that. Other functions you'll commonly use are [`beforeEach`](https://jestjs.io/docs/en/api#beforeeachfn-timeout) or [`beforeAll`](https://jestjs.io/docs/en/api#beforeallfn-timeout) that you can use for setting up the objects you're testing. Read more in the [Jest api reference](https://jestjs.io/docs/en/api).

若測試步驟過多或驗證點過雜，建議拆分成多個小測試。同時確保測試之間完全獨立——每個測試都應能單獨執行，且完整測試套件運行時，前序測試不會影響後續測試結果。

Lastly, as developers we like when our code works great and doesn't crash. With tests, this is often the opposite. Think of a failed test as of a _good thing!_ When a test fails, it often means something is not right. This gives you an opportunity to fix the problem before it impacts the users.

## 單元測試

單元測試針對程式碼最小單位，例如獨立函式或類別。

當被測對象存在依賴項時，通常需要進行模擬(mock)，如下節所述。

單元測試的優勢在於編寫與執行速度快，能即時反饋測試結果。Jest 還提供 [Watch 模式](https://jestjs.io/docs/en/cli#watch)，可持續運行與編輯程式碼相關的測試。

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### 模擬機制

當被測對象有外部依賴時，常需透過「模擬」將其替換為自訂實作。

> 原則上，測試中使用真實對象優於模擬對象，但某些情境無法避免。例如：當 JS 單元測試依賴 Java 或 Objective-C 寫的原生模組時。

假設您正在開發顯示城市天氣的應用程式，並使用某個提供天氣數據的外部服務。若服務回報下雨，您需顯示雨雲圖示。但在測試中不應直接呼叫該服務，因為：

- 這可能導致測試變慢且不穩定（因為涉及網路請求）
- 每次運行測試時，服務可能返回不同的數據
- 第三方服務可能在您急需運行測試時離線！

因此，您可以提供該服務的模擬實現，實際上取代了數千行程式碼和某些連接到網際網路的溫度計！

> Jest 內建[模擬支援](https://jestjs.io/docs/en/mock-functions#mocking-modules)，從函數層級到模組層級的模擬都能處理。

## 整合測試

在編寫較大的軟體系統時，其各個部分需要相互交互。在單元測試中，如果您的單元依賴於另一個單元，有時會需要模擬該依賴項，用假的來替代它。

在整合測試中，真實的單元會被組合（與您的應用程式相同）並一起測試，以確保它們的合作如預期般運作。這並不是說這裡不需要模擬：您仍然需要模擬（例如，模擬與天氣服務的通訊），但比單元測試中需要的少得多。

> Please note that the terminology around what integration testing means is not always consistent. Also, the line between what is a unit test and what is an integration test may not always be clear. For this guide, your test falls into "integration testing" if it:
>
> - Combines several modules of your app as described above
> - Uses an external system
> - Makes a network call to other application (such as the weather service API)
> - Does any kind of file or database <abbr title="Input/Output">I/O</abbr>

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 元件測試

React 元件負責渲染您的應用程式，用戶將直接與其輸出互動。即使您的應用程式的業務邏輯測試覆蓋率高且正確，如果沒有元件測試，您仍可能向用戶交付一個損壞的 UI。元件測試可能屬於單元測試和整合測試，但由於它們是 React Native 的核心部分，我們將單獨討論它們。

對於測試 React 元件，您可能需要測試兩件事：

- 互動：確保元件在用戶與之互動時行為正確（例如，當用戶按下按鈕時）
- 渲染：確保 React 使用的元件渲染輸出正確（例如，按鈕的外觀和在 UI 中的位置）

例如，如果您有一個帶有 `onPress` 監聽器的按鈕，您希望測試該按鈕是否正確顯示以及點擊按鈕是否被元件正確處理。

有幾個庫可以幫助您測試這些：

- React 的[測試渲染器](https://reactjs.org/docs/test-renderer.html)，與其核心一起開發，提供了一個 React 渲染器，可用於將 React 元件渲染為純 JavaScript 對象，而不依賴於 DOM 或原生移動環境。
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 建立在 React 的測試渲染器之上，並添加了下一段中描述的 `fireEvent` 和 `query` API。

> Component tests are only JavaScript tests running in Node.js environment. They do _not_ take into account any iOS, Android, or other platform code which is backing the React Native components. It follows that they cannot give you a 100% confidence that everything works for the user. If there is a bug in the iOS or Android code, they will not find it.

<img src="/docs/assets/p_tests-component.svg" alt=" " />

### 測試用戶互動

Aside from rendering some UI, your components handle events like `onChangeText` for `TextInput` or `onPress` for `Button`. They may also contain other functions and event callbacks. Consider the following example:

```jsx
function GroceryShoppingList() {
  const [groceryItem, setGroceryItem] = useState('');
  const [items, setItems] = useState([]);

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

在測試用戶互動時，從用戶的角度測試元件——頁面上有什麼？互動時會發生什麼變化？

作為一般規則，優先使用用戶可以看到或聽到的內容：

- make assertions using rendered text or [accessibility helpers](https://reactnative.dev/docs/accessibility#accessibility-properties)

相反地，你應避免：

- 對元件屬性(props)或狀態(state)進行斷言
- 使用testID查詢

避免測試實作細節如屬性或狀態——這類測試雖能運作，但未以使用者互動為導向，且容易因重構而失效（例如當你想重新命名某些內容或將類別元件改用鉤子(hooks)重寫時）。

> React class components are especially prone to testing their implementation details such as internal state, props or event handlers. To avoid testing implementation details, prefer using function components with Hooks, which make relying on component internals _harder_.

如[React Native Testing Library](https://callstack.github.io/react-native-testing-library/)等元件測試庫，透過謹慎選擇提供的API來促進撰寫以使用者為中心的測試。以下範例使用`fireEvent`方法的`changeText`和`press`模擬使用者與元件互動，並用查詢函式`getAllByText`在渲染輸出中尋找匹配的`Text`節點。

```jsx
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

此範例並非測試當你呼叫函式時某些狀態如何改變，而是測試當使用者在`TextInput`中變更文字並按下`Button`時會發生什麼！

### 測試渲染輸出

[快照測試(Snapshot testing)](https://jestjs.io/docs/en/snapshot-testing)是Jest啟用的一種進階測試方式。它是一個非常強大且底層的工具，因此使用時需格外謹慎。

A "component snapshot" is a JSX-like string created by a custom React serializer built into Jest. This serializer lets Jest translate React component trees to string that's human-readable. Put another way: a component snapshot is a textual representation of your component’s render output _generated_ during a test run. It may look like this:

```jsx
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

進行快照測試時，通常你會先實作元件，然後執行快照測試。快照測試會創建快照並將其作為參考快照保存到代碼庫中的文件裡。**該文件會被提交並在代碼審查時檢查**。未來任何對元件渲染輸出的變更都會改變其快照，導致測試失敗。此時你需要更新存儲的參考快照才能使測試通過。該變更同樣需要提交並審查。

快照有幾個弱點：

- 對開發者或審查者而言，可能難以判斷快照變更是否為預期行為或錯誤證據。特別是大型快照會迅速變得難以理解，其附加價值隨之降低。
- 當快照創建時，該時間點的快照會被視為正確——即使實際渲染輸出是錯誤的。
- 當快照失敗時，開發者容易直接使用Jest的`--updateSnapshot`選項更新快照，而未妥善檢查變更是否預期。因此需要一定的開發紀律。

快照本身並不能確保元件渲染邏輯正確，它們僅擅長防範意外變更，以及檢查測試中React樹下的元件是否收到預期的屬性（樣式等）。

We recommend that you only use small snapshots (see [`no-large-snapshots` rule](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)). If you want to test a _change_ between two React component states, use [`snapshot-diff`](https://github.com/jest-community/snapshot-diff). When in doubt, prefer explicit expectations as described in the previous paragraph.

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## 端到端測試(End-to-End Tests)

在端到端(E2E)測試中，你從使用者角度驗證應用程式在設備（或模擬器/仿真器）上是否如預期運作。

這需要將你的應用程式建置為發行版本配置，並針對其執行測試。在端對端測試中，你不再需要考慮 React 元件、React Native API、Redux 儲存或任何業務邏輯。這些並非端對端測試的目的，且在端對端測試期間甚至無法存取這些內容。

Instead, E2E testing libraries allow you to find and control elements in the screen of your app: for example, you can _actually_ tap buttons or insert text into `TextInputs` the same way a real user would. Then you can make assertions about whether or not a certain element exists in the app’s screen, whether or not it’s visible, what text it contains, and so on.

端對端測試能為你提供應用程式部分功能正常運作的最大信心。其代價包括：

- 撰寫它們比其他類型的測試更耗時
- 執行速度較慢
- 更容易出現不穩定性（「不穩定」測試是指隨機通過或失敗，而程式碼沒有任何變更的測試）

嘗試用端對端測試覆蓋應用程式的關鍵部分：身份驗證流程、核心功能、支付等。對於非關鍵部分，使用更快的 JavaScript 測試。你添加的測試越多，信心就越高，但維護和執行它們的時間也會越多。請權衡利弊，決定什麼最適合你。

有幾種可用的端對端測試工具：在 React Native 社群中，[Detox](https://github.com/wix/detox/) 是一個流行的框架，因為它是專為 React Native 應用程式設計的。另一個在 iOS 和 Android 應用程式領域受歡迎的函式庫是 [Appium](http://appium.io/)。

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## 總結

我們希望你喜歡閱讀並從本指南中學到一些東西。測試應用程式的方法有很多種。一開始可能很難決定使用什麼。然而，我們相信一旦你開始為你優秀的 React Native 應用程式添加測試，一切都會變得有意義。那麼還在等什麼？提高你的測試覆蓋率吧！

### 相關連結

- [React 測試概述](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest 文件](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](http://appium.io/)

---

_本指南最初由 [Vojtech Novak](https://twitter.com/vonovak) 完整撰寫並貢獻。_