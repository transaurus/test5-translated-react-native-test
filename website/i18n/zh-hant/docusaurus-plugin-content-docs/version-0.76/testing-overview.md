---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫的擴展，未預期的小錯誤和邊緣案例可能引發更大的故障。這些錯誤會導致糟糕的使用者體驗，最終造成業務損失。預防脆弱程式設計的方法之一，就是在釋出前測試你的程式碼。

本指南將涵蓋多種自動化方法，從靜態分析到端對端測試，確保你的應用程式如預期運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為何要測試

我們是人類，而人類會犯錯。測試之所以重要，是因為它能幫助你發現這些錯誤並驗證程式碼是否正常運作。更重要的是，測試能確保未來在新增功能、重構現有程式碼或升級專案主要依賴套件時，程式碼仍能持續運作。

測試的價值超乎你的想像。修復程式錯誤的最佳方法之一，就是撰寫一個能暴露該錯誤的失敗測試。當你修正錯誤並重新執行測試時，若測試通過，代表錯誤已修復且不會再被引入程式碼庫。

測試也能作為新團隊成員的技術文件。對於初次接觸程式碼庫的人來說，閱讀測試能幫助他們理解現有程式碼的運作方式。

最後但同樣重要的是，更多自動化測試意味著更少的手動<abbr title="Quality Assurance">QA</abbr>時間，釋放出寶貴的開發時間。

## 靜態分析

提升程式碼品質的第一步，是開始使用靜態分析工具。靜態分析會在撰寫程式碼時檢查錯誤，但不會實際執行這些程式碼。

- **Linters** 會分析程式碼，捕捉常見錯誤（如未使用的程式碼），幫助避免陷阱，並標記違反風格指南的行為（例如使用 Tab 而非空格，或反之，取決於你的設定）。
- **型別檢查** 確保傳遞給函式的結構符合函式設計時預期的型別，例如防止將字串傳遞給預期接收數字的計數函式。

React Native 預設配置了兩種工具：[ESLint](https://eslint.org/) 用於程式碼檢查，[TypeScript](typescript) 用於型別檢查。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造過程為例——在模型首次起飛展示所有複雜系統協同運作前，會先測試個別零件以確保其安全性和功能正常。例如，機翼會在極端負載下進行彎曲測試；引擎零件會測試耐用性；擋風玻璃會模擬鳥擊測試。

軟體開發也是如此。與其將整個程式寫在一個龐大的檔案中，不如將程式碼拆分為多個小型模組，這樣能比測試整個組裝成品更徹底地進行測試。因此，撰寫可測試的程式碼與撰寫乾淨、模組化的程式碼息息相關。

要讓應用程式更易測試，可先將應用的視圖部分（React 元件）與業務邏輯和應用狀態分離（無論你使用 Redux、MobX 或其他解決方案）。這樣一來，你可以將業務邏輯測試（不應依賴 React 元件）與元件本身的測試分開，後者的主要職責是渲染應用程式的 UI！

理論上，你可以將所有邏輯和資料獲取完全移出元件。如此一來，元件將專注於渲染，狀態完全獨立於元件，應用程式的邏輯甚至能在沒有任何 React 元件的情況下運作！

> 我們鼓勵你透過其他學習資源進一步探索可測試程式碼的主題。

## 撰寫測試

撰寫完可測試的程式碼後，是時候撰寫實際的測試了！React Native 的預設模板內建了 [Jest](https://jestjs.io) 測試框架。它包含一個針對此環境量身打造的預設配置，讓你能立即投入測試，無需調整設定和模擬——稍後會詳細說明[模擬](#mocking)。你可以使用 Jest 撰寫本指南中提到的所有測試類型。

> 如果你採用測試驅動開發（TDD），實際上你會先寫測試！這樣一來，程式碼的可測試性自然就具備了。

### 測試結構設計

你的測試應該簡短，且理想情況下只測試單一功能。讓我們從一個用 Jest 撰寫的單元測試範例開始：

```js
it('given a date in the past, colorForDueDate() returns red', () => {
  expect(colorForDueDate('2000-10-20')).toBe('red');
});
```

測試的描述由傳遞給 [`it`](https://jestjs.io/docs/en/api#testname-fn-timeout) 函式的字串定義。請仔細撰寫描述，確保測試內容清晰明確。盡量涵蓋以下要素：

1. **Given** - 某些前置條件
2. **When** - 被測試函式執行的某個動作
3. **Then** - 預期的結果

這也被稱為 AAA 模式（Arrange, Act, Assert）。

Jest offers [`describe`](https://jestjs.io/docs/en/api#describename-fn) function to help structure your tests. Use `describe` to group together all tests that belong to one functionality. Describes can be nested, if you need that. Other functions you'll commonly use are [`beforeEach`](https://jestjs.io/docs/en/api#beforeeachfn-timeout) or [`beforeAll`](https://jestjs.io/docs/en/api#beforeallfn-timeout) that you can use for setting up the objects you're testing. Read more in the [Jest api reference](https://jestjs.io/docs/en/api).

如果你的測試步驟繁多或包含多個預期結果，建議將其拆分為多個較小的測試。同時，確保測試之間完全獨立。套件中的每個測試都必須能單獨執行，無需先運行其他測試。反之，若同時運行所有測試，第一個測試的執行不應影響第二個測試的結果。

Lastly, as developers we like when our code works great and doesn't crash. With tests, this is often the opposite. Think of a failed test as of a _good thing!_ When a test fails, it often means something is not right. This gives you an opportunity to fix the problem before it impacts the users.

## 單元測試

單元測試涵蓋程式碼的最小部分，例如個別函式或類別。

當被測試對象具有依賴項時，通常需要將其模擬（mock）出來，如下一節所述。

單元測試的優點在於它們撰寫和運行速度快。因此，在開發過程中，你能快速獲得測試是否通過的反饋。Jest 甚至提供了一個選項，可以持續運行與你正在編輯的程式碼相關的測試：[監視模式](https://jestjs.io/docs/en/cli#watch)。

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### 模擬（Mocking）

有時，當被測試對象具有外部依賴項時，你會想要「模擬它們」。「模擬」是指用你自己的實現替換程式碼的某些依賴項。

> 一般來說，在測試中使用真實對象比使用模擬更好，但在某些情況下這是不可能的。例如：當你的 JS 單元測試依賴於用 Java 或 Objective-C 編寫的原生模組時。

假設你正在開發一個顯示當前城市天氣的應用程式，並使用某個外部服務或其他依賴項來獲取天氣資訊。如果服務告訴你正在下雨，你會想顯示一個雨雲圖像。你不希望在測試中調用該服務，因為：

- 這可能使測試變慢且不穩定（因為涉及網路請求）
- 服務每次運行測試時可能返回不同的數據
- 第三方服務可能在你急需運行測試時離線！

因此，你可以提供該服務的模擬實現，有效地替換數千行程式碼和一些連接到網際網路的溫度計！

> Jest 內建[模擬支援](https://jestjs.io/docs/en/mock-functions#mocking-modules)，從函式級別到模組級別的模擬都能實現。

## 整合測試

在開發大型軟體系統時，各個組件需要相互協作。在單元測試中，若測試單元依賴其他單元，有時會需要透過模擬(mocking)來替換依賴項，使用虛擬版本代替真實組件。

整合測試則是將實際的獨立單元（如同在應用程式中）組合起來共同測試，以確保它們的協作符合預期。這並不意味著完全不需要模擬——例如仍需模擬與天氣服務的通信——但使用頻率會遠低於單元測試。

> Please note that the terminology around what integration testing means is not always consistent. Also, the line between what is a unit test and what is an integration test may not always be clear. For this guide, your test falls into "integration testing" if it:
>
> - Combines several modules of your app as described above
> - Uses an external system
> - Makes a network call to other application (such as the weather service API)
> - Does any kind of file or database <abbr title="Input/Output">I/O</abbr>

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 組件測試

React組件負責渲染應用界面並直接與用戶互動。即使業務邏輯測試覆蓋率高且正確，若缺少組件測試仍可能交付有缺陷的UI。組件測試可歸類為單元或整合測試，但由於其在React Native中的核心地位，我們單獨討論。

測試React組件時，主要關注兩個面向：

- Interaction: to ensure the component behaves correctly when interacted with by a user (eg. when user presses a button)
- Rendering: to ensure the component render output used by React is correct (eg. the button's appearance and placement in the UI)

例如，若按鈕設有`onPress`監聽器，需同時測試按鈕顯示是否正確，以及點擊事件是否被正確處理。

可用測試工具包括：

- React官方[Test Renderer](https://reactjs.org/docs/test-renderer.html)：可在不依賴DOM或原生環境下，將組件渲染為純JavaScript對象
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)：基於Test Renderer擴增，提供下文將說明的`fireEvent`與`query`API

> Component tests are only JavaScript tests running in Node.js environment. They do _not_ take into account any iOS, Android, or other platform code which is backing the React Native components. It follows that they cannot give you a 100% confidence that everything works for the user. If there is a bug in the iOS or Android code, they will not find it.

<img src="/docs/assets/p_tests-component.svg" alt=" " />

### 測試用戶互動

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

測試用戶互動時，應從用戶視角出發：頁面顯示什麼？互動後如何變化？

基本原則：優先基於用戶可感知元素進行測試：

- make assertions using rendered text or [accessibility helpers](https://reactnative.dev/docs/accessibility#accessibility-properties)

反之應避免：

- 對組件props或state進行斷言
- 使用testID查詢元素

避免測試實作細節（如props/state），這類測試雖可行，但未對齊用戶互動模式，且重構時易失效（例如重命名或改用Hooks改寫class組件時）。

> React class components are especially prone to testing their implementation details such as internal state, props or event handlers. To avoid testing implementation details, prefer using function components with Hooks, which make relying on component internals _harder_.

諸如 [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 這類元件測試函式庫，透過精心設計的 API 選擇來協助撰寫以使用者為中心的測試。以下範例使用 `fireEvent` 方法中的 `changeText` 和 `press` 來模擬使用者與元件的互動，並透過查詢函式 `getAllByText` 在渲染輸出中尋找符合的 `Text` 節點。

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

此範例並非測試呼叫函式時狀態如何變化，而是測試當使用者在 `TextInput` 中變更文字並按下 `Button` 時會發生什麼！

### 測試渲染輸出

[快照測試](https://jestjs.io/docs/en/snapshot-testing)是 Jest 提供的一種進階測試方式。由於這是極其強大且底層的工具，使用時需格外謹慎。

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

進行快照測試時，通常會先實作元件再執行快照測試。測試會建立快照並將其作為參考快照存入專案倉庫的檔案中。**該檔案需提交並在程式碼審查時檢查**。後續任何元件渲染輸出的變更都會導致快照變化，從而使測試失敗。此時需更新儲存的參考快照才能使測試通過，而此變更同樣需提交並審查。

快照存在以下弱點：

- 對開發者或審查者而言，難以判斷快照變更屬預期行為還是錯誤跡象。大型快照尤其容易變得難以理解，其附加價值隨之降低。
- 快照建立當下即被視為正確——即使實際渲染輸出存在錯誤。
- 當快照失敗時，開發者容易直接使用 Jest 的 `--updateSnapshot` 選項更新，而未妥善檢查變更是否合理。因此需要相當的開發紀律。

快照本身無法確保元件渲染邏輯正確，其主要用途在於防範意外變更，以及驗證測試中 React 樹下的元件是否接收預期的 props（樣式等）。

We recommend that you only use small snapshots (see [`no-large-snapshots` rule](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)). If you want to test a _change_ between two React component states, use [`snapshot-diff`](https://github.com/jest-community/snapshot-diff). When in doubt, prefer explicit expectations as described in the previous paragraph.

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## 端對端測試

在端對端（E2E）測試中，您需驗證應用程式在裝置（或模擬器）上是否如預期般運作，測試角度完全模擬使用者視角。

此測試需在建置發布版應用程式後執行。E2E 測試不再考量 React 元件、React Native API、Redux store 或任何業務邏輯——這些並非 E2E 測試目的，且在測試過程中亦無法觸及。

Instead, E2E testing libraries allow you to find and control elements in the screen of your app: for example, you can _actually_ tap buttons or insert text into `TextInputs` the same way a real user would. Then you can make assertions about whether or not a certain element exists in the app’s screen, whether or not it’s visible, what text it contains, and so on.

E2E 測試能提供最高程度的信心，確保應用程式的某部分正常運作。其代價包括：

- 撰寫這類測試比其他類型的測試更耗時
- 執行速度較慢
- 更容易出現不穩定性（「不穩定」測試是指隨機通過或失敗，而程式碼沒有任何變更的測試）

嘗試用端對端測試覆蓋應用的關鍵部分：身份驗證流程、核心功能、支付等。對於非關鍵部分，使用更快的 JavaScript 測試。添加的測試越多，信心就越高，但維護和執行它們的時間也會越多。請權衡利弊，決定最適合的方案。

有幾種端對端測試工具可用：在 React Native 社群中，[Detox](https://github.com/wix/detox/) 是一個流行的框架，因為它是為 React Native 應用量身定制的。在 iOS 和 Android 應用領域，另一個流行的庫是 [Appium](https://appium.io/) 或 [Maestro](https://maestro.mobile.dev/)。

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## 總結

希望您喜歡閱讀並從本指南中學到一些東西。測試應用的方法有很多種，一開始可能很難決定使用什麼。然而，我們相信一旦您開始為出色的 React Native 應用添加測試，一切都會變得清晰。那麼還在等什麼？提高您的測試覆蓋率吧！

### 相關連結

- [React 測試概述](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest 文檔](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](https://appium.io/)
- [Maestro](https://maestro.mobile.dev/)

---

_本指南最初由 [Vojtech Novak](https://twitter.com/vonovak) 撰寫並貢獻。_