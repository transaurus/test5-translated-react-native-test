---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫擴展，未預期的小錯誤和邊緣案例可能引發連鎖反應導致重大故障。程式錯誤會造成糟糕的使用者體驗，最終導致業務損失。預防脆弱程式設計的方法之一，是在程式碼發布前進行測試。

本指南將涵蓋多種自動化測試方法，從靜態分析到端到端測試，確保應用程式如預期運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為何需要測試

人類難免犯錯。測試的重要性在於幫助發現錯誤並驗證程式碼是否正常運作。更重要的是，當您新增功能、重構現有程式碼或升級專案主要依賴套件時，測試能確保程式碼持續正常運作。

測試的價值超乎想像。修復程式錯誤的最佳方法之一，就是撰寫能暴露該錯誤的失敗測試。當您修復錯誤後重新執行測試，若通過則表示錯誤已修復且不會再次引入程式碼庫。

測試也能作為團隊新成員的技術文件。對於初次接觸程式碼庫的人，閱讀測試能幫助理解現有程式碼的運作方式。

最後但同樣重要的是，自動化測試越多，就能節省更多手動<abbr title="Quality Assurance">QA</abbr>的時間，釋放寶貴資源。

## 靜態分析

提升程式碼品質的第一步是使用靜態分析工具。靜態分析會在撰寫程式碼時檢查錯誤，但無需實際執行該程式碼。

- **Linters** 分析程式碼以捕捉常見錯誤（如未使用的程式碼），幫助避免陷阱，並標記違反風格指南的行為（例如使用製表符而非空格，或反之，取決於您的配置）。
- **型別檢查** 確保傳遞給函式的結構符合設計預期，例如防止將字串傳遞給預期數字的計數函式。

React Native 預設配置了兩種工具：[ESLint](https://eslint.org/) 用於程式碼檢查，[TypeScript](typescript) 用於型別檢查。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造流程為例——在模型首次起飛展示所有複雜系統協同運作前，會單獨測試每個零件以確保安全性和功能性。例如：機翼需通過極端負載下的彎曲測試；引擎零件需通過耐久性測試；擋風玻璃需模擬鳥擊測試。

軟體開發亦同。與其將整個程式寫在一個龐大檔案中，不如將程式碼拆分為多個小型模組，這樣能比測試整體組裝更徹底地進行測試。因此，撰寫可測試程式碼與撰寫乾淨、模組化的程式碼息息相關。

要提升應用程式的可測試性，首先應將視圖部分（React 元件）與業務邏輯和應用狀態分離（無論您使用 Redux、MobX 或其他解決方案）。如此一來，業務邏輯測試（不應依賴 React 元件）可獨立於主要負責渲染 UI 的元件。

理論上，您甚至可以將所有邏輯和資料獲取移出元件，使元件專注於渲染。應用狀態將完全獨立於元件，業務邏輯可在完全不使用 React 元件的情況下運作！

> 我們鼓勵您透過其他學習資源進一步探索可測試程式碼的主題。

## 撰寫測試

完成可測試程式碼後，接下來就是實際撰寫測試！React Native 的預設模板內建 [Jest](https://jestjs.io) 測試框架，包含針對此環境量身打造的預設配置，讓您無需調整設定和模擬就能立即投入測試——稍後將詳細說明[模擬技巧](#mocking)。您可以使用 Jest 撰寫本指南提到的所有測試類型。

> 如果你採用測試驅動開發（TDD），實際上你會先寫測試！這樣一來，程式碼的可測試性自然就具備了。

### 測試結構設計

測試應該簡潔，且理想情況下只測試單一功能。以下是一個使用 Jest 撰寫的單元測試範例：

```js
it('given a date in the past, colorForDueDate() returns red', () => {
  expect(colorForDueDate('2000-10-20')).toBe('red');
});
```

測試描述透過傳遞給 [`it`](https://jestjs.io/docs/en/api#testname-fn-timeout) 函式的字串來定義。請仔細撰寫描述，確保測試內容清晰明確。盡量涵蓋以下要素：

1. **Given** - 測試前提條件  
2. **When** - 被測試函式執行的動作  
3. **Then** - 預期結果

此模式亦稱為 AAA 原則（Arrange, Act, Assert）。

Jest 提供 [`describe`](https://jestjs.io/docs/en/api#describename-fn) 函式協助組織測試結構。使用 `describe` 將同一功能的測試分組，必要時可嵌套描述區塊。其他常用函式如 [`beforeEach`](https://jestjs.io/docs/en/api#beforeeachfn-timeout) 或 [`beforeAll`](https://jestjs.io/docs/en/api#beforeallfn-timeout)，可用於設置測試對象。詳見 [Jest API 文件](https://jestjs.io/docs/en/api)。

若測試步驟繁複或包含多重驗證，建議拆分成多個小型測試。同時確保測試之間完全獨立——每個測試都應能單獨執行，且測試套件中前序測試不會影響後續結果。

Lastly, as developers we like when our code works great and doesn't crash. With tests, this is often the opposite. Think of a failed test as of a _good thing!_ When a test fails, it often means something is not right. This gives you an opportunity to fix the problem before it impacts the users.

## 單元測試

單元測試針對程式碼最小單位，例如獨立函式或類別。

當測試對象存在依賴項時，通常需要進行模擬（mock），詳見下節說明。

單元測試的優勢在於快速撰寫與執行，能即時反饋測試結果。Jest 還提供 [Watch 模式](https://jestjs.io/docs/en/cli#watch)，可持續監控與當前編輯程式碼相關的測試。

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### 模擬（Mocking）

當測試對象依賴外部資源時，可能需要進行「模擬」——即以自訂實作替代依賴項。

> 原則上，測試中使用真實對象優於模擬對象，但某些情境無法避免。例如：當 JS 單元測試依賴 Java 或 Objective-C 寫的原生模組時。

假設你開發的天氣應用需呼叫外部服務取得數據。測試時不應實際呼叫該服務，因為：

- 網路請求會導致測試變慢且不穩定  
- 服務每次可能返回不同數據  
- 第三方服務可能在測試時離線！

此時可建立服務的模擬實作，有效取代數千行程式碼與聯網溫度計！

> Jest 內建完整的[模擬支援](https://jestjs.io/docs/en/mock-functions#mocking-modules)，從函式級到模組級皆可模擬。

## 整合測試

在開發大型軟體系統時，各個組件需要相互協作。在單元測試中，若測試單元依賴其他單元，有時會需要透過模擬(mock)來替換依賴項，使用虛擬版本代替實際組件。

整合測試則是將實際單元組合（如同在應用程式中）並共同測試，以驗證它們的協作是否符合預期。這並非意味著完全不需要模擬——例如仍需模擬天氣服務的通訊——但使用頻率會遠低於單元測試。

> Please note that the terminology around what integration testing means is not always consistent. Also, the line between what is a unit test and what is an integration test may not always be clear. For this guide, your test falls into "integration testing" if it:
>
> - Combines several modules of your app as described above
> - Uses an external system
> - Makes a network call to other application (such as the weather service API)
> - Does any kind of file or database <abbr title="Input/Output">I/O</abbr>

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 元件測試

React元件負責渲染應用界面，使用者將直接與其輸出互動。即使業務邏輯測試覆蓋率高且正確，若缺乏元件測試，仍可能交付有缺陷的UI。元件測試可歸類為單元或整合測試，但由於其是React Native核心部分，我們將獨立說明。

測試React元件時，主要關注兩個面向：

- Interaction: to ensure the component behaves correctly when interacted with by a user (eg. when user presses a button)
- Rendering: to ensure the component render output used by React is correct (eg. the button's appearance and placement in the UI)

舉例來說，若按鈕設有`onPress`監聽器，需同時測試按鈕顯示是否正確，以及點擊事件是否被元件正確處理。

可用測試工具包括：

- React官方[Test Renderer](https://reactjs.org/docs/test-renderer.html)，可將元件渲染為純JavaScript對象，無需依賴DOM或原生環境
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)基於Test Renderer，提供後文將說明的`fireEvent`與`query`API

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

測試用戶互動時，應從用戶視角出發：頁面顯示什麼？互動後產生什麼變化？

基本原則：優先基於用戶可感知的元素進行驗證：

- make assertions using rendered text or [accessibility helpers](https://reactnative.dev/docs/accessibility#accessibility-properties)

反之應避免：

- 對元件props或state進行斷言
- 使用testID查詢

避免測試實作細節（如props或state）——這類測試雖可行，但未對齊用戶互動模式，且容易因重構（例如改用Hook改寫class元件）而失效。

> React class components are especially prone to testing their implementation details such as internal state, props or event handlers. To avoid testing implementation details, prefer using function components with Hooks, which make relying on component internals _harder_.

諸如 [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 這類元件測試函式庫，透過精心設計的 API 選擇，能協助撰寫以使用者為中心的測試。以下範例使用 `fireEvent` 的 `changeText` 和 `press` 方法模擬使用者與元件的互動，並透過查詢函式 `getAllByText` 在渲染輸出中尋找符合的 `Text` 節點。

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

此範例並非測試呼叫函式時狀態如何變化，而是測試當使用者在 `TextInput` 變更文字並按下 `Button` 時會發生什麼！

### 測試渲染輸出

[快照測試](https://jestjs.io/docs/en/snapshot-testing)是 Jest 提供的一種進階測試方式。由於這是極強大且底層的工具，使用時需格外謹慎。

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

進行快照測試時，通常會先實作元件再執行快照測試。測試會建立快照並將其作為參考快照存入專案倉庫的檔案中。**該檔案需提交並在程式碼審查時檢查**。後續任何元件渲染輸出的變更都會導致快照變化，從而使測試失敗。此時需更新儲存的參考快照才能使測試通過，而該變更同樣需提交並審查。

快照存在若干弱點：

- 對開發者或審查者而言，難以判斷快照變更屬預期修改或錯誤證據。大型快照尤其容易變得難以理解，其附加價值隨之降低。
- 快照建立當下即被視為正確——即便實際渲染輸出存在錯誤。
- 當快照失敗時，開發者容易直接使用 Jest 的 `--updateSnapshot` 選項更新，而未妥善檢查變更是否預期。因此需要一定的開發紀律。

快照本身無法確保元件渲染邏輯正確，其主要價值在於防範意外變更，並驗證測試中 React 樹內的元件是否收到預期的 props（樣式等）。

We recommend that you only use small snapshots (see [`no-large-snapshots` rule](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)). If you want to test a _change_ between two React component states, use [`snapshot-diff`](https://github.com/jest-community/snapshot-diff). When in doubt, prefer explicit expectations as described in the previous paragraph.

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## 端對端測試

在端對端（E2E）測試中，您需驗證應用程式在真實裝置（或模擬器）上是否如預期般運作，測試角度完全模擬使用者行為。

此測試需建置正式版應用程式並對其執行測試。E2E 測試不再考量 React 元件、React Native API、Redux store 或任何業務邏輯——這些並非 E2E 測試目的，且在 E2E 測試期間甚至無法存取。

Instead, E2E testing libraries allow you to find and control elements in the screen of your app: for example, you can _actually_ tap buttons or insert text into `TextInputs` the same way a real user would. Then you can make assertions about whether or not a certain element exists in the app’s screen, whether or not it’s visible, what text it contains, and so on.

E2E 測試能提供最高程度的信心，確保應用程式的某部分正常運作。其代價包括：

- 撰寫這類測試比其他類型的測試更耗時
- 執行速度較慢
- 更容易出現不穩定性（「不穩定」測試是指隨機通過或失敗，而程式碼沒有任何變更的測試）

請盡量為應用程式的關鍵部分加入端對端測試：認證流程、核心功能、支付等。對於非關鍵部分，則可使用執行速度更快的 JavaScript 測試。加入的測試越多，信心程度就越高，但維護和執行測試的時間也會隨之增加。請權衡利弊，選擇最適合您的方式。

目前有數種端對端測試工具可供選擇：在 React Native 社群中，[Detox](https://github.com/wix/detox/) 是一個熱門框架，因為它是專為 React Native 應用程式量身打造的。在 iOS 和 Android 應用程式領域，另一個熱門的函式庫是 [Appium](https://appium.io/) 或 [Maestro](https://maestro.mobile.dev/)。

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## 總結

希望您喜歡閱讀本指南並從中學到一些東西。測試應用程式的方法有很多種，一開始可能很難決定該使用哪種方法。不過，我們相信一旦您開始為出色的 React Native 應用程式加入測試，一切就會變得清晰。那麼，還在等什麼呢？快來提高測試覆蓋率吧！

### 相關連結

- [React 測試概述](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest 文件](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](https://appium.io/)
- [Maestro](https://maestro.mobile.dev/)

---

_本指南最初由 [Vojtech Novak](https://twitter.com/vonovak) 撰寫並完整貢獻。_