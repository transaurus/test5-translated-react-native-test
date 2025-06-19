---
id: fast-refresh
title: Fast Refresh
---

Fast Refresh 是 React Native 的一項功能，可讓您即時獲得 React 元件變更的反饋。Fast Refresh 預設啟用，您可以在 [React Native 開發者選單](/docs/debugging#accessing-the-in-app-developer-menu) 中切換「啟用 Fast Refresh」。啟用後，大多數編輯應能在一兩秒內顯示效果。

## 運作原理

- If you edit a module that **only exports React component(s)**, Fast Refresh will update the code only for that module, and re-render your component. You can edit anything in that file, including styles, rendering logic, event handlers, or effects.
- If you edit a module with exports that _aren't_ React components, Fast Refresh will re-run both that module, and the other modules importing it. So if both `Button.js` and `Modal.js` import `Theme.js`, editing `Theme.js` will update both components.
- Finally, if you **edit a file** that's **imported by modules outside of the React tree**, Fast Refresh **will fall back to doing a full reload**. You might have a file which renders a React component but also exports a value that is imported by a **non-React component**. For example, maybe your component also exports a constant, and a non-React utility module imports it. In that case, consider migrating the constant to a separate file and importing it into both files. This will re-enable Fast Refresh to work. Other cases can usually be solved in a similar way.

## 錯誤恢復能力

若在 Fast Refresh 期間發生**語法錯誤**，修正後重新存檔即可消除紅屏。系統會阻止執行含語法錯誤的模組，無需重新載入應用程式。

If you make a **runtime error during the module initialization** (for example, typing `Style.create` instead of `StyleSheet.create`), the Fast Refresh session will continue once you fix the error. The redbox will disappear, and the module will be updated.

If you make a mistake that leads to a **runtime error inside your component**, the Fast Refresh session will _also_ continue after you fix the error. In that case, React will remount your application using the updated code.

If you have [error boundaries](https://reactjs.org/docs/error-boundaries.html) in your app (which is a good idea for graceful failures in production), they will retry rendering on the next edit after a redbox. In that sense, having an error boundary can prevent you from always getting kicked out to the root app screen. However, keep in mind that error boundaries shouldn't be _too_ granular. They are used by React in production, and should always be designed intentionally.

## 限制

Fast Refresh 會嘗試保留編輯中元件的本地 React 狀態，但僅在安全情況下生效。以下情況會導致每次編輯後本地狀態重置：

- Local state is not preserved for class components (only function components and Hooks preserve state).
- The module you're editing might have _other_ exports in addition to a React component.
- Sometimes, a module would export the result of calling higher-order component like `createNavigationContainer(MyScreen)`. If the returned component is a class, state will be reset.

長期而言，隨著程式碼庫轉向函式元件與 Hooks，狀態保留的案例將會增加。

## 技巧

- Fast Refresh preserves React local state in function components (and Hooks) by default.
- Sometimes you might want to _force_ the state to be reset, and a component to be remounted. For example, this can be handy if you're tweaking an animation that only happens on mount. To do this, you can add `// @refresh reset` anywhere in the file you're editing. This directive is local to the file, and instructs Fast Refresh to remount components defined in that file on every edit.

## Fast Refresh 與 Hooks

在可能的情況下，Fast Refresh 會嘗試保留元件狀態於編輯之間。具體而言，只要不更改參數或 Hook 調用順序，`useState` 和 `useRef` 會保留其先前的值。

Hooks with dependencies—such as `useEffect`, `useMemo`, and `useCallback`—will _always_ update during Fast Refresh. Their list of dependencies will be ignored while Fast Refresh is happening.

例如，當你將 `useMemo(() => x * 2, [x])` 編輯為 `useMemo(() => x * 10, [x])` 時，即使依賴項 `x` 未改變，它仍會重新執行。若 React 不這麼做，你的編輯將無法反映在畫面上！

有時這可能導致非預期結果。例如，即使 `useEffect` 的依賴項陣列為空，仍會在 Fast Refresh 期間執行一次。然而，撰寫能承受 `useEffect` 偶發重新執行的程式碼，即使沒有 Fast Refresh 也是良好實踐，這能讓你後續更容易為其新增依賴項。