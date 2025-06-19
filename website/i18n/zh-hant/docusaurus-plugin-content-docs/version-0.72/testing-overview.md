---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫的擴展，未預期的小錯誤和邊緣案例可能引發更大的故障。這些錯誤會導致糟糕的使用者體驗，最終造成業務損失。防止脆弱程式設計的方法之一，就是在將程式碼發布前進行測試。

本指南將涵蓋多種自動化方法，從靜態分析到端對端測試，確保您的應用程式如預期運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為何需要測試

我們是人類，而人類會犯錯。測試之所以重要，是因為它能幫助您發現這些錯誤並驗證程式碼是否正常運作。更重要的是，測試能確保當您新增功能、重構現有程式碼或升級專案的主要依賴項時，程式碼仍能持續運作。

測試的價值可能超出您的想像。修復程式錯誤的最佳方法之一，就是撰寫一個能暴露該錯誤的失敗測試。當您修復錯誤並重新執行測試時，若測試通過，就表示錯誤已修復且不會再被引入程式碼庫。

測試也能作為新團隊成員的參考文件。對於初次接觸程式碼庫的人來說，閱讀測試能幫助他們理解現有程式碼的運作方式。

最後但同樣重要的是，更多的自動化測試意味著減少手動<abbr title="Quality Assurance">QA</abbr>的時間，釋放寶貴資源。

## 靜態分析

提升程式碼品質的第一步，是開始使用靜態分析工具。靜態分析會在您撰寫程式碼時檢查錯誤，但無需實際執行該程式碼。

- **Linters** 會分析程式碼，捕捉常見錯誤（如未使用的程式碼），幫助避免陷阱，並標記風格指南中的禁忌（例如使用製表符而非空格，或反之，取決於您的配置）。
- **型別檢查** 確保您傳遞給函式的結構符合函式的設計要求，例如防止將字串傳遞給預期數字的計數函式。

React Native 預設配置了兩種工具：[ESLint](https://eslint.org/) 用於程式碼檢查，[TypeScript](typescript) 用於型別檢查。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造過程為例——在模型首次起飛以證明其複雜系統協同運作前，會先測試各個零件以確保其安全性和功能。例如，機翼會在極端負載下進行彎曲測試；引擎零件會測試耐用性；擋風玻璃會模擬鳥擊測試。

軟體開發也是如此。與其將整個程式寫在一個龐大的檔案中，不如將程式碼拆分為多個小型模組，這樣能比測試整個組裝後的程式更徹底地進行測試。因此，撰寫可測試的程式碼與撰寫乾淨、模組化的程式碼息息相關。

為了讓應用程式更易於測試，首先應將應用的視圖部分（即 React 元件）與業務邏輯和應用狀態分離（無論您使用 Redux、MobX 或其他解決方案）。這樣一來，您可以將業務邏輯測試（不應依賴 React 元件）與元件本身分開，後者的主要職責是渲染應用程式的 UI！

理論上，您可以將所有邏輯和資料獲取完全移出元件。這樣一來，元件將專注於渲染，狀態完全獨立於元件，而應用程式的邏輯甚至可以在沒有任何 React 元件的情況下運作！

> 我們鼓勵您進一步探索其他學習資源中關於可測試程式碼的主題。

## 撰寫測試

撰寫完可測試的程式碼後，接下來就是撰寫實際的測試！React Native 的預設模板內建了 [Jest](https://jestjs.io) 測試框架。它包含一個針對此環境量身打造的預設配置，讓您無需調整設定和模擬就能立即投入工作——稍後會詳細介紹[模擬](#mocking)。您可以使用 Jest 撰寫本指南中提到的所有測試類型。

> 如果你採用測試驅動開發（TDD），實際上你會先寫測試！這樣一來，程式碼的可測試性自然就具備了。

### 測試結構設計

測試應該簡潔，且理想情況下只測試單一功能。讓我們從一個用 Jest 撰寫的單元測試範例開始：

```js
it('given a date in the past, colorForDueDate() returns red', () => {
  expect(colorForDueDate('2000-10-20')).toBe('red');
});
```

測試的描述由傳遞給 [`it`](https://jestjs.io/docs/en/api#testname-fn-timeout) 函式的字串定義。請仔細撰寫描述，確保測試內容清晰明確。盡量涵蓋以下要素：

1. **Given** - 某些前提條件  
2. **When** - 被測試函式執行的某個動作  
3. **Then** - 預期的結果

這也被稱為 AAA 模式（Arrange 準備、Act 執行、Assert 斷言）。

Jest offers [`describe`](https://jestjs.io/docs/en/api#describename-fn) function to help structure your tests. Use `describe` to group together all tests that belong to one functionality. Describes can be nested, if you need that. Other functions you'll commonly use are [`beforeEach`](https://jestjs.io/docs/en/api#beforeeachfn-timeout) or [`beforeAll`](https://jestjs.io/docs/en/api#beforeallfn-timeout) that you can use for setting up the objects you're testing. Read more in the [Jest api reference](https://jestjs.io/docs/en/api).

若測試步驟繁多或有多個斷言，建議拆分成多個小測試。同時確保測試之間完全獨立——每個測試都應能單獨執行，不需依賴其他測試的執行順序。反之，當所有測試一起運行時，前一個測試不應影響後續測試的結果。

Lastly, as developers we like when our code works great and doesn't crash. With tests, this is often the opposite. Think of a failed test as of a _good thing!_ When a test fails, it often means something is not right. This gives you an opportunity to fix the problem before it impacts the users.

## 單元測試

單元測試涵蓋程式碼的最小單位，例如獨立函式或類別。

當被測對象有依賴項時，通常需要將其模擬（mock）替換，詳見下節說明。

單元測試的優勢在於快速撰寫和執行，能即時反饋測試結果。Jest 還提供 [Watch 模式](https://jestjs.io/docs/en/cli#watch)，可持續運行與當前編輯程式碼相關的測試。

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### 模擬（Mocking）

當被測對象有外部依賴時，你可能需要「模擬替換」這些依賴。「模擬」是指用自訂實現取代程式碼的某些依賴項。

> 通常，在測試中使用真實對象比模擬更好，但某些情況下無法實現。例如：當 JS 單元測試依賴於用 Java 或 Objective-C 編寫的原生模組時。

假設你正在開發顯示城市天氣的應用，並使用某個提供天氣資訊的外部服務。若服務回報正在下雨，你需要顯示雨雲圖示。但測試中不應直接呼叫該服務，因為：

- 網路請求會導致測試變慢且不穩定  
- 服務每次可能返回不同數據  
- 第三方服務可能在關鍵時刻離線！

因此，你可以提供該服務的模擬實現，有效取代數千行程式碼和聯網溫度計！

> Jest 內建 [模擬支援功能](https://jestjs.io/docs/en/mock-functions#mocking-modules)，從函式級別到模組級別的模擬都能實現。

## 整合測試

在開發大型軟體系統時，各個部分需要相互協作。在單元測試中，如果某個單元依賴於另一個單元，有時會需要模擬(mock)該依賴項，用假物件替代它。

整合測試則是將實際的獨立單元組合起來（與應用程式中的方式相同）並一起測試，以確保它們的協作符合預期。這並不意味著在此完全不需要模擬：你仍可能需要模擬（例如模擬與天氣服務的通訊），但相比單元測試會少很多。

> Please note that the terminology around what integration testing means is not always consistent. Also, the line between what is a unit test and what is an integration test may not always be clear. For this guide, your test falls into "integration testing" if it:
>
> - Combines several modules of your app as described above
> - Uses an external system
> - Makes a network call to other application (such as the weather service API)
> - Does any kind of file or database <abbr title="Input/Output">I/O</abbr>

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 元件測試

React元件負責渲染應用程式介面，使用者會直接與其輸出互動。即使應用程式的業務邏輯測試覆蓋率高且正確，若缺乏元件測試，仍可能向使用者交付有問題的UI。元件測試可歸類為單元測試或整合測試，但由於它們是React Native的核心部分，我們將單獨說明。

測試React元件時，你可能需要關注兩個面向：

- Interaction: to ensure the component behaves correctly when interacted with by a user (eg. when user presses a button)
- Rendering: to ensure the component render output used by React is correct (eg. the button's appearance and placement in the UI)

舉例來說，若有一個帶有`onPress`監聽器的按鈕，你需要測試該按鈕是否正確顯示，以及點擊按鈕時元件的處理邏輯是否正確。

以下套件可協助進行這類測試：

- React官方開發的[Test Renderer](https://reactjs.org/docs/test-renderer.html)，提供能將React元件渲染為純JavaScript物件的渲染器，無需依賴DOM或原生行動環境。
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)基於React的測試渲染器，並增加了下一節將說明的`fireEvent`與`query` API。

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

測試使用者互動時，應從使用者角度測試元件——頁面上顯示什麼？互動後會如何變化？

基本原則是優先使用使用者能感知的元素：

- make assertions using rendered text or [accessibility helpers](https://reactnative.dev/docs/accessibility#accessibility-properties)

反之，應避免：

- 對元件props或state進行斷言
- 使用testID查詢

避免測試實作細節如props或state——雖然這類測試可行，但未聚焦於使用者如何與元件互動，且容易因重構（例如重新命名或將類別元件改寫為Hook）而失效。

> React class components are especially prone to testing their implementation details such as internal state, props or event handlers. To avoid testing implementation details, prefer using function components with Hooks, which make relying on component internals _harder_.

像 [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 這類元件測試函式庫，透過精心設計的 API 來協助撰寫以使用者為中心的測試。以下範例使用 `fireEvent` 的 `changeText` 和 `press` 方法模擬使用者與元件的互動，並透過 `getAllByText` 查詢函式在渲染輸出中尋找符合的 `Text` 節點。

```tsx
test('given empty GroceryShoppingList, user can add an item to it', () => {
  const {getByPlaceholderText, getByText, getAllByText} = render(
    <GroceryShoppingList />,
  );

  fireEvent.changeText(
    getByPlaceholderText('Enter grocery item'),
    'banana',
  );
  fireEvent.press(getByText('Add the item to list'));

  const bananaElements = getAllByText('banana');
  expect(bananaElements).toHaveLength(1); // expect 'banana' to be on the list
});
```

這個範例並非測試呼叫函式時狀態如何改變，而是測試當使用者在 `TextInput` 中變更文字並按下 `Button` 時會發生什麼！

### 測試渲染輸出

[快照測試](https://jestjs.io/docs/en/snapshot-testing)是 Jest 提供的一種進階測試方式。由於它是非常強大且底層的工具，使用時需格外謹慎。

A "component snapshot" is a JSX-like string created by a custom React serializer built into Jest. This serializer lets Jest translate React component trees to string that's human-readable. Put another way: a component snapshot is a textual representation of your component’s render output _generated_ during a test run. It may look like this:

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

進行快照測試時，通常會先實作元件再執行快照測試。測試會建立快照並將其作為參考快照存入程式庫中的檔案。**該檔案會被提交並在程式碼審查時檢查**。未來任何元件渲染輸出的變更都會導致快照改變，使測試失敗。此時需更新儲存的參考快照才能讓測試通過，而此變更同樣需要提交並審查。

快照有以下弱點：

- 對開發者或審查者而言，難以判斷快照變更是預期修改還是錯誤跡象。大型快照尤其容易變得難以理解，其附加價值隨之降低。
- 快照建立當下即被視為正確——即使實際渲染輸出有誤。
- 當快照失敗時，開發者容易直接使用 `--updateSnapshot` 選項更新快照，而未妥善檢查變更是否合理，因此需要相當的開發紀律。

快照本身無法確保元件渲染邏輯正確，其主要用途在於防範意外變更，以及檢查測試中 React 樹下的元件是否收到預期的 props（如樣式等）。

We recommend that you only use small snapshots (see [`no-large-snapshots` rule](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)). If you want to test a _change_ between two React component states, use [`snapshot-diff`](https://github.com/jest-community/snapshot-diff). When in doubt, prefer explicit expectations as described in the previous paragraph.

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## 端對端測試

在端對端（E2E）測試中，您需驗證應用程式在裝置（或模擬器）上是否能如預期般運作，測試角度完全模擬使用者行為。

這需要以發布配置建置應用程式，並對其執行測試。E2E 測試不再考量 React 元件、React Native API、Redux 儲存或任何業務邏輯——這些並非 E2E 測試的目的，且在 E2E 測試期間也無法存取這些內容。

Instead, E2E testing libraries allow you to find and control elements in the screen of your app: for example, you can _actually_ tap buttons or insert text into `TextInputs` the same way a real user would. Then you can make assertions about whether or not a certain element exists in the app’s screen, whether or not it’s visible, what text it contains, and so on.

E2E 測試能提供最高程度的信心，確保應用程式的某部分正常運作。其代價包括：

- 撰寫這類測試比其他類型的測試更耗時
- 執行速度較慢
- 更容易出現不穩定性（「不穩定」測試是指隨機通過或失敗，而程式碼沒有任何變更的測試）

請盡量為應用程式的關鍵部分進行端對端測試：身份驗證流程、核心功能、支付等。對於非關鍵部分，則使用更快速的 JavaScript 測試。添加的測試越多，信心就越高，但維護和執行它們的時間也會增加。請權衡利弊，決定最適合您的方式。

有幾種端對端測試工具可供選擇：在 React Native 社群中，[Detox](https://github.com/wix/detox/) 是一個受歡迎的框架，因為它是專為 React Native 應用程式設計的。另一個在 iOS 和 Android 應用程式領域中流行的庫是 [Appium](http://appium.io/) 或 [Maestro](https://maestro.mobile.dev/)。

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## 總結

希望您喜歡閱讀並從本指南中學到一些東西。測試應用程式的方法有很多種，一開始可能很難決定使用哪種方法。然而，我們相信一旦您開始為您出色的 React Native 應用程式添加測試，一切都會變得有意義。那麼還在等什麼呢？提高您的測試覆蓋率吧！

### 相關連結

- [React 測試概述](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest 文件](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](http://appium.io/)
- [Maestro](https://maestro.mobile.dev/)

---

_本指南最初由 [Vojtech Novak](https://twitter.com/vonovak) 撰寫並貢獻完整內容。_