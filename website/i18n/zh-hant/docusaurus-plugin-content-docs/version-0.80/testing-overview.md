---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫的擴展，未預期的小錯誤和邊緣案例可能引發更大的故障。這些錯誤會導致糟糕的使用者體驗，最終造成業務損失。預防脆弱程式設計的方法之一，就是在將程式碼發布前進行測試。

本指南將涵蓋多種自動化方法，從靜態分析到端對端測試，確保您的應用程式如預期運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為何需要測試

我們是人類，而人類會犯錯。測試之所以重要，是因為它能幫助您發現這些錯誤並驗證程式碼是否正常運作。更重要的是，測試能確保在未來新增功能、重構現有程式碼或升級專案主要依賴套件時，程式碼仍能持續運作。

測試的價值遠超乎您的想像。修正程式錯誤的最佳方法之一，就是撰寫一個能暴露該錯誤的失敗測試。當您修正錯誤後重新執行測試，若測試通過，就表示錯誤已被修正且不會再被引入程式碼庫。

測試也能作為新團隊成員的技術文件。對於初次接觸程式碼庫的人來說，閱讀測試能幫助他們理解現有程式碼的運作方式。

最後但同樣重要的是，更多自動化測試意味著減少手動<abbr title="Quality Assurance">QA</abbr>的時間，釋放寶貴資源。

## 靜態分析

提升程式碼品質的第一步，是開始使用靜態分析工具。靜態分析會在您撰寫程式碼時檢查錯誤，但無需實際執行這些程式碼。

- **Linter** 會分析程式碼以捕捉常見錯誤（如未使用的程式碼），幫助避免陷阱，並標記違反風格指南的行為（例如使用 Tab 而非空格，或相反情況，取決於您的配置）。
- **型別檢查** 確保傳遞給函式的結構符合函式設計預期，例如防止將字串傳遞給預期接收數字的計數函式。

React Native 預設配置了兩種工具：[ESLint](https://eslint.org/) 用於程式碼檢查，[TypeScript](typescript) 用於型別檢查。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造流程為例——在模型首次起飛展示所有複雜系統協同運作前，會先測試各個零件以確保其安全性和功能正常。例如：機翼需通過極端負載下的彎曲測試；引擎零件需通過耐久性測試；擋風玻璃需模擬鳥擊測試。

軟體開發也是如此。與其將整個程式寫在一個龐大檔案中，不如將程式碼拆分為多個小型模組，這樣能比測試完整組裝的系統進行更徹底的測試。因此，撰寫可測試的程式碼與撰寫乾淨、模組化的程式碼息息相關。

要讓應用程式更具可測試性，首先應將視圖部分（React 元件）與業務邏輯和應用狀態分離（無論您使用 Redux、MobX 或其他解決方案）。如此一來，您可以讓業務邏輯測試（不應依賴 React 元件）獨立於元件本身，而元件的主要職責是渲染應用程式的 UI！

理論上，您甚至可以將所有邏輯和資料獲取移出元件。這樣元件將專注於渲染，狀態完全獨立於元件，應用程式邏輯甚至能在沒有任何 React 元件的情況下運作！

> 我們鼓勵您透過其他學習資源進一步探索可測試程式碼的主題。

## 撰寫測試

撰寫完可測試的程式碼後，接下來就是撰寫實際測試！React Native 的預設模板內建 [Jest](https://jestjs.io) 測試框架。它包含針對此環境量身打造的預設配置，讓您無需立即調整設定和模擬（mock）就能開始工作——稍後將詳細說明[模擬](#mocking)。您可以使用 Jest 撰寫本指南中提到的所有測試類型。

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

如果你的測試步驟繁多或有多個預期結果，建議將其拆分為多個較小的測試。同時，確保測試之間完全獨立。套件中的每個測試都必須能單獨執行，無需先運行其他測試。反之，若同時運行所有測試，第一個測試的執行不應影響第二個測試的結果。

Lastly, as developers we like when our code works great and doesn't crash. With tests, this is often the opposite. Think of a failed test as of a _good thing!_ When a test fails, it often means something is not right. This gives you an opportunity to fix the problem before it impacts the users.

## 單元測試

單元測試涵蓋程式碼的最小部分，例如個別函式或類別。

當被測試對象具有依賴項時，通常需要將其模擬（mock）出來，如下一段所述。

單元測試的優點在於撰寫和運行速度快。因此，在開發過程中，你能快速獲得測試是否通過的反饋。Jest 甚至提供持續運行與編輯程式碼相關測試的選項：[監視模式](https://jestjs.io/docs/en/cli#watch)。

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### 模擬（Mocking）

有時，當被測試對象有外部依賴時，你會需要「模擬」這些依賴。「模擬」是指用你自己的實作替換程式碼的某些依賴項。

> 一般來說，在測試中使用真實對象比使用模擬更好，但在某些情況下這不可行。例如：當你的 JS 單元測試依賴於用 Java 或 Objective-C 編寫的原生模組時。

假設你正在開發一個顯示當前城市天氣的應用程式，並使用某個提供天氣資訊的外部服務或其他依賴項。如果服務告訴你正在下雨，你會想顯示一個雨雲圖像。你不希望在測試中調用該服務，因為：

- 這可能使測試變慢且不穩定（因為涉及網路請求）
- 服務每次運行測試時可能返回不同的數據
- 第三方服務可能在你急需運行測試時離線！

因此，你可以提供該服務的模擬實作，有效取代數千行程式碼和某些連網的溫度計！

> Jest 內建[模擬支援](https://jestjs.io/docs/en/mock-functions#mocking-modules)，從函式層級到模組層級的模擬都能實現。

## 整合測試

在開發大型軟體系統時，各個組件需要相互協作。在單元測試中，若測試單元依賴其他組件，往往需要透過模擬(mock)來替換這些依賴項。

整合測試則是將實際單元組合（如同在應用程式中）並共同測試，以驗證其協作是否符合預期。這並非意味著完全不需要模擬——例如仍需模擬天氣服務的通信——但使用頻率會遠低於單元測試。

> Please note that the terminology around what integration testing means is not always consistent. Also, the line between what is a unit test and what is an integration test may not always be clear. For this guide, your test falls into "integration testing" if it:
>
> - Combines several modules of your app as described above
> - Uses an external system
> - Makes a network call to other application (such as the weather service API)
> - Does any kind of file or database <abbr title="Input/Output">I/O</abbr>

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 元件測試

React元件直接負責渲染介面並與用戶互動。即使業務邏輯測試覆蓋率高且正確，若缺乏元件測試仍可能交付有缺陷的UI。元件測試可歸類為單元或整合測試，但由於其在React Native中的核心地位，我們單獨說明。

React元件測試主要關注兩個面向：

- 互動行為：驗證用戶操作時元件的響應（如按鈕點擊）
- 渲染輸出：確保React使用的渲染結果正確（如按鈕外觀與佈局）

例如，若按鈕設有`onPress`監聽器，需同時測試其視覺呈現與點擊處理邏輯。

常用測試工具包括：

- React官方[Test Renderer](https://reactjs.org/docs/test-renderer.html)，可將元件渲染為純JavaScript對象，無需DOM或原生環境
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)基於Test Renderer擴充，提供`fireEvent`與`query`API

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

測試原則建議優先採用用戶可感知的元素：

- make assertions using rendered text or [accessibility helpers](https://reactnative.dev/docs/accessibility#accessibility-properties)

反之應避免：

- 對元件props或state進行斷言
- 使用testID查詢

避免測試實作細節（如props/state），這類測試雖可行，但未對齊用戶交互模式，且重構時易失效（例如改用Hook改寫class元件時）。

> React class components are especially prone to testing their implementation details such as internal state, props or event handlers. To avoid testing implementation details, prefer using function components with Hooks, which make relying on component internals _harder_.

諸如 [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 這類元件測試函式庫，透過謹慎選擇提供的 API 來促進撰寫以使用者為中心的測試。以下範例使用 `fireEvent` 方法 `changeText` 和 `press` 來模擬使用者與元件的互動，以及查詢函式 `getAllByText` 來尋找渲染輸出中匹配的 `Text` 節點。

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

此範例並非測試當你呼叫某個函式時狀態如何變化，而是測試當使用者在 `TextInput` 中變更文字並按下 `Button` 時會發生什麼！

### 測試渲染輸出

[快照測試](https://jestjs.io/docs/en/snapshot-testing) 是 Jest 啟用的一種進階測試方式。它是一個非常強大且底層的工具，因此使用時需格外謹慎。

「元件快照」是由 Jest 內建的自訂 React 序列化器所產生的類 JSX 字串。此序列化器讓 Jest 能將 React 元件樹轉換為人類可讀的字串。換句話說：元件快照是測試執行期間 _生成_ 的元件渲染輸出的文字表示。它可能看起來像這樣：

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

進行快照測試時，通常會先實作你的元件，然後執行快照測試。快照測試會建立快照並將其作為參考快照儲存到你的代碼庫中的檔案裡。**該檔案會被提交並在代碼審查時檢查**。未來任何對元件渲染輸出的變更都會改變其快照，導致測試失敗。此時你需要更新儲存的參考快照才能使測試通過。該變更同樣需要被提交和審查。

快照有幾個弱點：

- 對開發者或審查者而言，可能難以判斷快照中的變更是否為預期行為或錯誤證據。特別是大型快照可能很快變得難以理解，其附加價值隨之降低。
- 當快照建立時，該時間點的快照會被視為正確——即使實際渲染輸出是錯誤的。
- 當快照失敗時，開發者容易直接使用 `--updateSnapshot` jest 選項更新快照，而沒有適當檢查變更是否預期。因此需要一定的開發紀律。

快照本身並不能確保元件渲染邏輯正確，它們主要用於防範意外變更，以及檢查測試中的 React 樹下元件是否收到預期的 props（樣式等）。

我們建議僅使用小型快照（參見 [`no-large-snapshots` 規則](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)）。若需測試兩個 React 元件狀態間的 _差異_，可使用 [`snapshot-diff`](https://github.com/jest-community/snapshot-diff)。如有疑問，建議優先採用前文所述的明確斷言方式。

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## 端對端測試

在端對端（E2E）測試中，你需驗證應用程式在裝置（或模擬器）上是否如預期般運作，從使用者角度進行測試。

這需要以發布配置建置你的應用程式，並對其執行測試。在 E2E 測試中，你不再考慮 React 元件、React Native API、Redux store 或任何業務邏輯。這並非 E2E 測試的目的，且在 E2E 測試期間這些甚至無法被存取。

Instead, E2E testing libraries allow you to find and control elements in the screen of your app: for example, you can _actually_ tap buttons or insert text into `TextInputs` the same way a real user would. Then you can make assertions about whether or not a certain element exists in the app’s screen, whether or not it’s visible, what text it contains, and so on.

E2E 測試能為你提供應用程式某部分正常運作的最高可信度。其代價包括：

- 撰寫這類測試比其他類型的測試更耗時
- 執行速度較慢
- 更容易出現不穩定性（「不穩定」測試是指隨機通過或失敗，而程式碼沒有任何變更的測試）

請嘗試用端對端測試覆蓋應用程式的關鍵部分：身份驗證流程、核心功能、支付等。對於非關鍵部分，則使用更快的 JavaScript 測試。添加的測試越多，信心就越高，但維護和執行它們的時間也會增加。請權衡利弊，選擇最適合您的方式。

目前有幾種端對端測試工具可供選擇：在 React Native 社群中，[Detox](https://github.com/wix/detox/) 是一個熱門框架，因為它是專為 React Native 應用程式設計的。在 iOS 和 Android 應用程式領域，另一個流行的庫是 [Appium](https://appium.io/) 或 [Maestro](https://maestro.mobile.dev/)。

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## 總結

希望您喜歡閱讀並從本指南中學到一些東西。測試應用程式的方法有很多種，一開始可能很難決定使用哪種方法。然而，我們相信一旦您開始為出色的 React Native 應用程式添加測試，一切都會變得清晰。那麼還在等什麼呢？提高您的測試覆蓋率吧！

### 相關連結

- [React 測試概述](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest 文件](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](https://appium.io/)
- [Maestro](https://maestro.mobile.dev/)

---

_本指南最初由 [Vojtech Novak](https://twitter.com/vonovak) 撰寫並貢獻完整內容。_