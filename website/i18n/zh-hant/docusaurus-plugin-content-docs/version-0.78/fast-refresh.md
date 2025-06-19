---
id: fast-refresh
title: Fast Refresh
---

Fast Refresh 是 React Native 的一項功能，可讓您幾乎即時獲得 React 元件變更的反饋。Fast Refresh 預設為啟用狀態，您可以在 [React Native 開發者選單](/docs/debugging#accessing-the-in-app-developer-menu) 中切換「啟用 Fast Refresh」。啟用後，大多數編輯應能在一兩秒內顯示變更。

## 運作原理

- If you edit a module that **only exports React component(s)**, Fast Refresh will update the code only for that module, and re-render your component. You can edit anything in that file, including styles, rendering logic, event handlers, or effects.
- If you edit a module with exports that _aren't_ React components, Fast Refresh will re-run both that module, and the other modules importing it. So if both `Button.js` and `Modal.js` import `Theme.js`, editing `Theme.js` will update both components.
- Finally, if you **edit a file** that's **imported by modules outside of the React tree**, Fast Refresh **will fall back to doing a full reload**. You might have a file which renders a React component but also exports a value that is imported by a **non-React component**. For example, maybe your component also exports a constant, and a non-React utility module imports it. In that case, consider migrating the constant to a separate file and importing it into both files. This will re-enable Fast Refresh to work. Other cases can usually be solved in a similar way.

## 錯誤恢復能力

若您在 Fast Refresh 階段發生**語法錯誤**，修正後再次儲存檔案即可。紅屏錯誤會消失。含語法錯誤的模組會被阻止執行，因此無需重新載入應用程式。

若您在**模組初始化時發生執行階段錯誤**（例如誤輸入 `Style.create` 而非 `StyleSheet.create`），修正錯誤後 Fast Refresh 階段會繼續。紅屏會消失，模組將被更新。

If you make a mistake that leads to a **runtime error inside your component**, the Fast Refresh session will _also_ continue after you fix the error. In that case, React will remount your application using the updated code.

If you have [error boundaries](https://reactjs.org/docs/error-boundaries.html) in your app (which is a good idea for graceful failures in production), they will retry rendering on the next edit after a redbox. In that sense, having an error boundary can prevent you from always getting kicked out to the root app screen. However, keep in mind that error boundaries shouldn't be _too_ granular. They are used by React in production, and should always be designed intentionally.

## 限制

Fast Refresh tries to preserve local React state in the component you're editing, but only if it's safe to do so. Here's a few reasons why you might see local state being reset on every edit to a file:

- Local state is not preserved for class components (only function components and Hooks preserve state).
- The module you're editing might have _other_ exports in addition to a React component.
- Sometimes, a module would export the result of calling higher-order component like `createNavigationContainer(MyScreen)`. If the returned component is a class, state will be reset.

長期來看，隨著更多程式碼改用函式元件和 Hooks，預期狀態保留會適用更多情況。

## 技巧

- Fast Refresh preserves React local state in function components (and Hooks) by default.
- Sometimes you might want to _force_ the state to be reset, and a component to be remounted. For example, this can be handy if you're tweaking an animation that only happens on mount. To do this, you can add `// @refresh reset` anywhere in the file you're editing. This directive is local to the file, and instructs Fast Refresh to remount components defined in that file on every edit.

## Fast Refresh 與 Hooks

在可能的情況下，Fast Refresh 會嘗試保留元件在編輯之間的狀態。特別是，只要你不改變參數或 Hook 調用的順序，`useState` 和 `useRef` 會保留它們之前的值。

Hooks with dependencies—such as `useEffect`, `useMemo`, and `useCallback`—will _always_ update during Fast Refresh. Their list of dependencies will be ignored while Fast Refresh is happening.

例如，當你將 `useMemo(() => x * 2, [x])` 編輯為 `useMemo(() => x * 10, [x])` 時，即使 `x`（依賴項）沒有改變，它也會重新運行。如果 React 不這樣做，你的編輯就不會反映在螢幕上！

有時，這可能會導致意想不到的結果。例如，即使是一個具有空依賴項數組的 `useEffect` 也會在 Fast Refresh 期間重新運行一次。然而，即使沒有 Fast Refresh，編寫能夠偶爾重新運行 `useEffect` 的彈性程式碼也是一個好習慣。這使得你以後更容易為它引入新的依賴項。