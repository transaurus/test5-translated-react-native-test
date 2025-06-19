---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫的擴展，未預期的小錯誤和邊緣案例可能引發更大的故障。這些錯誤會導致糟糕的使用者體驗，最終造成業務損失。預防脆弱程式設計的方法之一，就是在將程式碼發布前進行測試。

本指南將涵蓋多種自動化測試方法，從靜態分析到端到端測試，確保您的應用程式如預期運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為何需要測試

我們是人類，而人類會犯錯。測試之所以重要，是因為它能幫助您發現這些錯誤並驗證程式碼是否正常運作。更重要的是，測試能確保在未來新增功能、重構現有程式碼或升級專案主要依賴項時，程式碼仍能持續運作。

測試的價值可能超出您的想像。修復程式錯誤的最佳方法之一，就是撰寫一個能暴露該錯誤的失敗測試。當您修復錯誤並重新執行測試時，若測試通過，就表示錯誤已被修復且不會再被重新引入程式碼庫。

測試也能作為新團隊成員的技術文件。對於初次接觸程式碼庫的人來說，閱讀測試能幫助他們理解現有程式碼的運作方式。

最後但同樣重要的是，更多的自動化測試意味著更少的手動<abbr title="Quality Assurance">QA</abbr>時間，釋放出寶貴的開發資源。

## 靜態分析

提升程式碼品質的第一步是使用靜態分析工具。靜態分析會在撰寫程式碼時檢查錯誤，但無需實際執行這些程式碼。

- **Linters** 會分析程式碼以捕捉常見錯誤（如未使用的程式碼），幫助避免陷阱，並標記違反風格指南的行為（例如使用製表符而非空格，或反之，取決於您的配置）。
- **型別檢查** 確保傳遞給函式的結構符合函式的設計預期，例如防止將字串傳遞給預期接收數字的計數函式。

React Native 預設配置了兩種工具：[ESLint](https://eslint.org/) 用於程式碼檢查，[TypeScript](typescript) 用於型別檢查。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造過程為例——在模型首次起飛以展示所有複雜系統協同運作前，會先測試各個零件以確保其安全性和功能正常。例如，機翼會在極端負載下進行彎曲測試；引擎零件會測試耐用性；擋風玻璃會模擬鳥擊測試。

軟體開發也是如此。與其將整個程式寫在一個龐大檔案中，不如將程式碼拆分為多個小型模組，這樣能比測試組裝後的整體更徹底地進行測試。因此，撰寫可測試的程式碼與撰寫乾淨、模組化的程式碼息息相關。

為了讓應用程式更易測試，首先應將應用的視圖部分（React 元件）與業務邏輯和應用狀態分離（無論您使用 Redux、MobX 或其他解決方案）。這樣一來，您可以將業務邏輯測試（不應依賴 React 元件）與元件本身的測試分開，後者的主要職責是渲染應用程式的 UI！

理論上，您甚至可以將所有邏輯和資料獲取移出元件。這樣一來，元件將專注於渲染，而狀態將完全獨立於元件。應用的邏輯甚至可以在沒有任何 React 元件的情況下運作！

> 我們鼓勵您透過其他學習資源進一步探索可測試程式碼的主題。

## 撰寫測試

撰寫完可測試的程式碼後，就是時候撰寫實際的測試了！React Native 的預設模板附帶了 [Jest](https://jestjs.io) 測試框架。它包含一個針對此環境量身定制的預設配置，讓您無需立即調整配置和模擬（mock）就能開始工作——稍後會[詳細介紹模擬](#mocking)。您可以使用 Jest 撰寫本指南中提到的所有類型的測試。

> 如果你採用測試驅動開發（TDD），實際上你會先寫測試！這樣一來，程式碼的可測試性自然就具備了。

### 測試結構設計

測試應該簡潔且理想上只驗證單一事項。以下是一個使用 Jest 撰寫的單元測試範例：

```js
it('given a date in the past, colorForDueDate() returns red', () => {
  expect(colorForDueDate('2000-10-20')).toBe('red');
});
```

測試描述透過傳遞給 [`it`](https://jestjs.io/docs/en/api#testname-fn-timeout) 函式的字串定義。請仔細撰寫描述以明確說明測試內容，並盡量涵蓋以下要素：

1. **Given** - 測試前提條件  
2. **When** - 執行待測函式的動作  
3. **Then** - 預期結果

此模式亦稱為 AAA（Arrange, Act, Assert）。

Jest 提供 [`describe`](https://jestjs.io/docs/en/api#describename-fn) 函式協助組織測試。使用 `describe` 將同一功能的測試歸組，必要時可嵌套描述。其他常用函式如 [`beforeEach`](https://jestjs.io/docs/en/api#beforeeachfn-timeout) 或 [`beforeAll`](https://jestjs.io/docs/en/api#beforeallfn-timeout)，可用於設置測試對象。詳見 [Jest API 文件](https://jestjs.io/docs/en/api)。

若測試步驟或驗證點過多，建議拆分成多個小測試。同時確保測試完全獨立——每個測試都應能單獨執行，且測試間不互相影響（例如前一個測試不改變後續測試的執行環境）。

Lastly, as developers we like when our code works great and doesn't crash. With tests, this is often the opposite. Think of a failed test as of a _good thing!_ When a test fails, it often means something is not right. This gives you an opportunity to fix the problem before it impacts the users.

## 單元測試

單元測試針對程式最小單位（如獨立函式或類別）進行驗證。

當測試對象存在依賴項時，通常需要模擬（mock）這些依賴（詳見下節說明）。

單元測試的優勢在於快速撰寫與執行，能即時反饋程式碼狀態。Jest 還提供 [Watch 模式](https://jestjs.io/docs/en/cli#watch)，可持續執行與修改程式碼相關的測試。

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### 模擬（Mocking）

當測試對象有外部依賴時，常需進行「模擬」——即用自訂實作替代原始依賴。

> 原則上，測試中使用真實對象優於模擬，但某些情境無法避免。例如：當 JS 單元測試依賴 Java 或 Objective-C 寫的原生模組時。

假設你開發的天氣 App 需呼叫外部服務取得數據。測試時不應實際呼叫該服務，因為：

- 網路請求會導致測試變慢且不穩定  
- 服務每次可能返回不同數據  
- 第三方服務可能在關鍵時刻離線！

此時可提供模擬服務，用幾行程式碼替代真實的網路連線與感測器陣列。

> Jest 內建[多層級模擬支援](https://jestjs.io/docs/en/mock-functions#mocking-modules)，從函式到模組皆可模擬。

## 整合測試

在開發大型軟體系統時，各個部分需要相互協作。在單元測試中，如果某個單元依賴於另一個單元，有時會需要模擬(mock)該依賴項，用假物件替代它。

整合測試則是將實際的單元組合在一起（與應用程式中相同）並共同測試，以確保它們的協作符合預期。這並非表示整合測試完全不需要模擬——你仍可能需要模擬（例如模擬與天氣服務的通訊），但使用頻率會遠低於單元測試。

> Please note that the terminology around what integration testing means is not always consistent. Also, the line between what is a unit test and what is an integration test may not always be clear. For this guide, your test falls into "integration testing" if it:
>
> - Combines several modules of your app as described above
> - Uses an external system
> - Makes a network call to other application (such as the weather service API)
> - Does any kind of file or database <abbr title="Input/Output">I/O</abbr>

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 元件測試

React元件負責渲染應用界面，使用者將直接與其輸出互動。即使應用的業務邏輯測試覆蓋率高且正確，若缺乏元件測試，仍可能向使用者交付有缺陷的UI。元件測試可歸類為單元測試或整合測試，但由於它們是React Native的核心部分，我們將單獨討論。

測試React元件時，你可能需要關注兩個面向：

- 互動：確保元件在使用者操作時行為正確（例如使用者點擊按鈕）
- 渲染：確保React使用的元件渲染輸出正確（例如按鈕的外觀與UI中的位置）

舉例來說，若按鈕設有`onPress`監聽器，你需同時測試按鈕是否正確顯示，以及點擊事件是否被元件正確處理。

以下套件可協助進行這類測試：

- React官方提供的[Test Renderer](https://reactjs.org/docs/test-renderer.html)，能將元件渲染為純JavaScript物件，無需依賴DOM或原生行動環境
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)基於React測試渲染器，增加了下一節將說明的`fireEvent`與`query`API

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

測試使用者互動時，應從使用者角度測試元件——頁面顯示什麼？互動後會如何變化？

基本原則建議使用使用者可感知的元素：

- make assertions using rendered text or [accessibility helpers](https://reactnative.dev/docs/accessibility#accessibility-properties)

反之應避免：

- 對元件props或state進行斷言
- 使用testID查詢

避免測試實作細節如props或state——雖然這類測試可行，但未聚焦於使用者互動方式，且容易因重構（例如重新命名或改用Hooks改寫class元件）而失效。

> React class components are especially prone to testing their implementation details such as internal state, props or event handlers. To avoid testing implementation details, prefer using function components with Hooks, which make relying on component internals _harder_.

諸如 [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 這類元件測試函式庫，透過精心選擇提供的 API 來促進撰寫以使用者為中心的測試。以下範例使用 `fireEvent` 方法的 `changeText` 和 `press` 來模擬使用者與元件的互動，以及查詢函式 `getAllByText` 來尋找渲染輸出中匹配的 `Text` 節點。

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

這個範例並非測試當你呼叫某個函式時狀態如何變化，而是測試當使用者在 `TextInput` 中變更文字並按下 `Button` 時會發生什麼！

### 測試渲染輸出

[快照測試](https://jestjs.io/docs/en/snapshot-testing) 是由 Jest 啟用的一種進階測試方式。它是一個非常強大且底層的工具，因此使用時需要格外注意。

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

使用快照測試時，通常你會先實作你的元件，然後執行快照測試。快照測試接著會建立一個快照，並將其作為參考快照儲存到你的程式庫中的一個檔案裡。**該檔案會被提交並在程式碼審查時檢查**。未來對元件渲染輸出的任何變更都會改變其快照，導致測試失敗。此時你需要更新儲存的參考快照才能使測試通過。該變更同樣需要被提交和審查。

快照有幾個弱點：

- 對於開發者或審查者來說，可能難以判斷快照中的變更是有意為之還是錯誤的證據。特別是大型快照可能很快變得難以理解，其附加價值也會降低。
- 當快照被建立時，它在那個時間點被視為正確的——即使渲染輸出實際上可能是錯誤的。
- 當快照失敗時，開發者可能會傾向於使用 Jest 的 `--updateSnapshot` 選項來更新快照，而沒有適當檢查變更是否預期。因此需要一定的開發紀律。

快照本身並不能確保你的元件渲染邏輯是正確的，它們主要擅長防範意外變更，以及檢查測試中的 React 樹下的元件是否接收到預期的 props（樣式等）。

We recommend that you only use small snapshots (see [`no-large-snapshots` rule](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)). If you want to test a _change_ between two React component states, use [`snapshot-diff`](https://github.com/jest-community/snapshot-diff). When in doubt, prefer explicit expectations as described in the previous paragraph.

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## 端對端測試

在端對端（E2E）測試中，你從使用者的角度驗證你的應用程式在裝置（或模擬器/仿真器）上是否如預期運作。

這通過在發布配置中建置你的應用程式並對其執行測試來完成。在 E2E 測試中，你不再考慮 React 元件、React Native API、Redux 儲存或任何業務邏輯。這不是 E2E 測試的目的，且在 E2E 測試期間這些甚至無法存取。

Instead, E2E testing libraries allow you to find and control elements in the screen of your app: for example, you can _actually_ tap buttons or insert text into `TextInputs` the same way a real user would. Then you can make assertions about whether or not a certain element exists in the app’s screen, whether or not it’s visible, what text it contains, and so on.

E2E 測試能為你提供應用程式某部分是否正常運作的最大信心。其代價包括：

- 撰寫這類測試比其他類型的測試更耗時
- 執行速度較慢
- 更容易出現不穩定性（「不穩定」測試是指隨機通過或失敗，而程式碼沒有任何變更的測試）

請盡量為應用程式的關鍵部分進行端對端測試：身份驗證流程、核心功能、支付等。對於非關鍵部分，則使用更快速的 JavaScript 測試。添加的測試越多，信心就越高，但維護和執行它們的時間也會增加。請權衡利弊，選擇最適合您的方式。

目前有幾種端對端測試工具可供選擇：在 React Native 社群中，[Detox](https://github.com/wix/detox/) 是一個熱門框架，因為它是專為 React Native 應用程式設計的。另一個在 iOS 和 Android 應用程式領域中流行的庫是 [Appium](https://appium.io/) 或 [Maestro](https://maestro.mobile.dev/)。

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## 總結

希望您喜歡閱讀並從本指南中學到一些東西。測試應用程式的方法有很多種，一開始可能很難決定使用哪種方法。不過，我們相信一旦您開始為出色的 React Native 應用程式添加測試，一切都會變得清晰。那麼還在等什麼呢？提高您的測試覆蓋率吧！

### 相關連結

- [React 測試概述](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest 文件](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](https://appium.io/)
- [Maestro](https://maestro.mobile.dev/)

---

_本指南最初由 [Vojtech Novak](https://twitter.com/vonovak) 撰寫並貢獻完整內容。_