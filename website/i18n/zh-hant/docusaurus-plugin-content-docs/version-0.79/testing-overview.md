---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫擴展，未預期的小錯誤和邊緣案例可能引發連鎖反應導致重大故障。程式錯誤會造成糟糕的使用者體驗，最終導致業務損失。預防脆弱程式設計的方法之一，就是在釋出前對程式碼進行測試。

本指南將涵蓋多種自動化測試方法，從靜態分析到端對端測試，確保您的應用程式如預期運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為何需要測試

人非聖賢，孰能無過。測試的重要性在於幫助發現錯誤並驗證程式碼是否正常運作。更重要的是，測試能確保未來新增功能、重構現有程式碼或升級專案主要依賴項時，程式碼仍能持續運作。

測試的價值遠超乎想像。修復程式錯誤的最佳方式之一，就是撰寫能暴露該錯誤的失敗測試。當您修復錯誤後重新執行測試，若通過則表示錯誤已修復且不會再次引入程式碼庫。

測試也能作為新成員的技術文件。對於初次接觸程式碼庫的人，閱讀測試能幫助理解現有程式運作方式。

最後但同樣重要的是，更多自動化測試意味著減少手動<abbr title="Quality Assurance">QA</abbr>時間，釋放寶貴資源。

## 靜態分析

提升程式碼品質的第一步是使用靜態分析工具。靜態分析會在撰寫程式碼時檢查錯誤，但無需實際執行該程式碼。

- **Linter** 會分析程式碼以捕捉常見錯誤（如未使用的程式碼），幫助避免陷阱，並標記違反風格指南的行為（例如使用製表符而非空格，或反之，取決於您的配置）。
- **型別檢查** 確保傳遞給函式的結構符合設計預期，例如防止將字串傳遞給預期接收數字的計數函式。

React Native 預設配置了兩種工具：[ESLint](https://eslint.org/) 用於程式碼檢查，[TypeScript](typescript) 用於型別檢查。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造流程為例——在模型首次起飛展示所有複雜系統協同運作前，會先測試個別零件以確保安全性和功能性。例如：機翼需通過極端負載下的彎曲測試；引擎零件需通過耐久性測試；擋風玻璃需通過模擬鳥擊測試。

軟體開發亦同。與其將整個程式寫在一個龐大檔案中，不如將程式碼拆分為多個小型模組，這樣能比測試組裝後的整體進行更徹底的測試。因此，撰寫可測試程式碼與撰寫乾淨、模組化的程式碼息息相關。

要提升應用程式的可測試性，首先應將應用的視圖部分（React 元件）與業務邏輯和應用狀態分離（無論您使用 Redux、MobX 或其他解決方案）。如此一來，您可以保持業務邏輯測試（不應依賴 React 元件）與元件本身的獨立性，後者的主要職責是渲染應用程式的 UI！

理論上，您可以更進一步將所有邏輯和資料獲取移出元件。這樣元件將專注於渲染，狀態完全獨立於元件，應用邏輯甚至能在沒有任何 React 元件的情況下運作！

> 我們鼓勵您透過其他學習資源深入探索可測試程式碼的主題。

## 撰寫測試

完成可測試程式碼後，是時候撰寫實際測試了！React Native 的預設模板內建 [Jest](https://jestjs.io) 測試框架，包含針對此環境量身打造的預設配置，讓您無需立即調整設定和模擬（稍後會詳述[模擬技巧](#mocking)）。您可以使用 Jest 撰寫本指南提到的所有測試類型。

> 如果你採用測試驅動開發（TDD），實際上你會先寫測試！這樣一來，程式碼的可測試性自然就具備了。

### 測試結構設計

測試應該簡潔且理想上只驗證單一功能。以下是一個使用 Jest 撰寫的單元測試範例：

```js
it('given a date in the past, colorForDueDate() returns red', () => {
  expect(colorForDueDate('2000-10-20')).toBe('red');
});
```

測試描述透過傳遞給 [`it`](https://jestjs.io/docs/en/api#testname-fn-timeout) 函式的字串定義。請仔細撰寫描述以明確說明測試內容，並盡量涵蓋以下要素：

1. **Given** - 前置條件
2. **When** - 執行待測函式的動作
3. **Then** - 預期結果

此模式亦稱為 AAA（Arrange, Act, Assert）。

Jest 提供 [`describe`](https://jestjs.io/docs/en/api#describename-fn) 函式協助組織測試。使用 `describe` 將同一功能的測試分組，必要時可嵌套描述區塊。其他常用函式如 [`beforeEach`](https://jestjs.io/docs/en/api#beforeeachfn-timeout) 或 [`beforeAll`](https://jestjs.io/docs/en/api#beforeallfn-timeout)，可用於設置測試對象。詳見 [Jest API 文件](https://jestjs.io/docs/en/api)。

若測試步驟或驗證點過多，建議拆分成多個小測試。同時確保測試間完全獨立——每個測試都應能單獨執行，且完整測試套件執行時，前序測試不會影響後續測試結果。

Lastly, as developers we like when our code works great and doesn't crash. With tests, this is often the opposite. Think of a failed test as of a _good thing!_ When a test fails, it often means something is not right. This gives you an opportunity to fix the problem before it impacts the users.

## 單元測試

單元測試針對程式碼最小單位（如獨立函式或類別）進行驗證。

當測試對象存在依賴項時，通常需要進行模擬（mock），詳見下節說明。

單元測試的優勢在於快速撰寫與執行，能即時反饋程式碼狀態。Jest 還提供 [Watch 模式](https://jestjs.io/docs/en/cli#watch)，可持續執行與編輯程式碼相關的測試。

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### 模擬技術

當測試對象依賴外部資源時，常需進行「模擬」（Mocking）——以自訂實現替代真實依賴項。

> 原則上，測試中使用真實對象優於模擬對象，但某些情境無法避免。例如：當 JS 單元測試依賴 Java 或 Objective-C 寫的原生模組時。

假設你開發的天氣應用依賴外部服務提供數據。若服務回報下雨，應用需顯示雨雲圖示。測試時不應直接呼叫真實服務，因為：

- 網路請求會導致測試變慢且不穩定
- 服務每次可能返回不同數據
- 第三方服務可能在關鍵時刻離線！

此時可提供服務的模擬實現，用幾行程式碼替代真實的聯網溫度計系統。

> Jest 內建從函式級到模組級的[完整模擬支援](https://jestjs.io/docs/en/mock-functions#mocking-modules)。

## 整合測試

在開發大型軟體系統時，各個組件需要相互協作。在單元測試中，若某個單元依賴於另一個單元，有時會透過模擬(mocking)將依賴項替換為虛擬版本。

整合測試則會將實際的獨立單元組合（如同在應用程式中一樣）並共同測試，以確保它們的協作符合預期。這並非意味著完全不需要模擬——例如仍需模擬與天氣服務的通訊——但使用頻率會遠低於單元測試。

> Please note that the terminology around what integration testing means is not always consistent. Also, the line between what is a unit test and what is an integration test may not always be clear. For this guide, your test falls into "integration testing" if it:
>
> - Combines several modules of your app as described above
> - Uses an external system
> - Makes a network call to other application (such as the weather service API)
> - Does any kind of file or database <abbr title="Input/Output">I/O</abbr>

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 元件測試

React元件負責渲染應用界面，使用者將直接與其輸出互動。即使業務邏輯測試覆蓋率高且正確，若缺乏元件測試仍可能交付有缺陷的UI。元件測試可歸類於單元或整合測試，但由於其為React Native核心部分，我們將獨立說明。

測試React元件時，主要有兩個面向需驗證：

- Interaction: to ensure the component behaves correctly when interacted with by a user (eg. when user presses a button)
- Rendering: to ensure the component render output used by React is correct (eg. the button's appearance and placement in the UI)

舉例來說，若按鈕設有`onPress`監聽器，需同時測試其視覺呈現正確性與點擊事件的處理邏輯。

以下套件可協助測試：

- React官方[Test Renderer](https://reactjs.org/docs/test-renderer.html)：能在不依賴DOM或原生環境的情況下，將元件渲染為純JavaScript物件
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

測試使用者互動時，應從使用者視角出發——頁面顯示什麼？互動後如何變化？

基本原則：優先基於使用者可感知的元素進行驗證：

- make assertions using rendered text or [accessibility helpers](https://reactnative.dev/docs/accessibility#accessibility-properties)

反之應避免：

- 對元件props或state進行斷言
- 使用testID查詢元件

避免測試實作細節（如props或state），這類測試雖可行但非以使用者互動為導向，且容易因重構（例如改用Hooks改寫class元件）而失效。

> React class components are especially prone to testing their implementation details such as internal state, props or event handlers. To avoid testing implementation details, prefer using function components with Hooks, which make relying on component internals _harder_.

諸如 [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 這類元件測試函式庫，透過精心設計的 API 選擇來協助編寫以使用者為中心的測試。以下範例使用 `fireEvent` 方法的 `changeText` 和 `press` 來模擬使用者與元件的互動，並透過查詢函式 `getAllByText` 在渲染輸出中尋找匹配的 `Text` 節點。

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

進行快照測試時，通常會先實作元件再執行快照測試。測試會建立快照並將其作為參考快照存入程式庫中的檔案。**該檔案需提交並在程式碼審查時檢查**。後續任何元件渲染輸出的變更都會導致快照變化，從而使測試失敗。此時需更新儲存的參考快照才能使測試通過，而此變更同樣需提交並審查。

快照存在若干弱點：

- 對開發者或審查者而言，難以判斷快照變更屬預期行為還是錯誤證據。特別是大型快照會迅速變得難以理解，其附加價值隨之降低。
- 快照建立當下即被視為正確版本——即使實際渲染輸出存在錯誤。
- 當快照失敗時，開發者容易未經充分檢查變更是否預期，就直接使用 `--updateSnapshot` 選項更新快照。因此需要相當的開發紀律。

快照本身無法確保元件渲染邏輯正確，其主要價值在於防範意外變更，以及驗證測試中 React 樹內的元件是否接收預期的 props（樣式等）。

We recommend that you only use small snapshots (see [`no-large-snapshots` rule](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)). If you want to test a _change_ between two React component states, use [`snapshot-diff`](https://github.com/jest-community/snapshot-diff). When in doubt, prefer explicit expectations as described in the previous paragraph.

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## 端對端測試

在端對端（E2E）測試中，您需驗證應用程式在裝置（或模擬器）上是否如預期般運作，測試角度完全模擬使用者視角。

此測試需建置發布版本的應用程式並對其執行測試。在 E2E 測試中，您無需考慮 React 元件、React Native API、Redux 儲存庫或任何業務邏輯——這些並非 E2E 測試目的，且在 E2E 測試期間也無法存取這些內容。

Instead, E2E testing libraries allow you to find and control elements in the screen of your app: for example, you can _actually_ tap buttons or insert text into `TextInputs` the same way a real user would. Then you can make assertions about whether or not a certain element exists in the app’s screen, whether or not it’s visible, what text it contains, and so on.

E2E 測試能為應用程式功能正常運作提供最高程度的信心，但需付出以下代價：

- 撰寫這類測試比其他類型的測試更耗時
- 執行速度較慢
- 更容易出現不穩定性（「不穩定」測試是指隨機通過或失敗，而程式碼沒有任何變更的測試）

嘗試用端到端測試覆蓋應用的關鍵部分：身份驗證流程、核心功能、支付等。對於非關鍵部分，使用更快的 JavaScript 測試。添加的測試越多，信心就越高，但維護和執行它們的時間也會越多。請權衡利弊，決定最適合你的方案。

有幾種端到端測試工具可供選擇：在 React Native 社群中，[Detox](https://github.com/wix/detox/) 是一個流行的框架，因為它是專為 React Native 應用設計的。另一個在 iOS 和 Android 應用領域流行的庫是 [Appium](https://appium.io/) 或 [Maestro](https://maestro.mobile.dev/)。

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## 總結

希望你在閱讀本指南時有所收穫並學到了一些東西。測試應用的方法有很多種，一開始可能很難決定使用哪種方法。但我們相信，一旦你開始為你的優秀 React Native 應用添加測試，一切都會變得清晰。那麼還在等什麼？提高你的測試覆蓋率吧！

### 相關連結

- [React 測試概述](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest 文檔](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](https://appium.io/)
- [Maestro](https://maestro.mobile.dev/)

---

_本指南最初由 [Vojtech Novak](https://twitter.com/vonovak) 撰寫並貢獻完整內容。_