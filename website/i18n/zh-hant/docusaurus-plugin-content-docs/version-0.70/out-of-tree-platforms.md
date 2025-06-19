---
id: out-of-tree-platforms
title: Out-of-Tree Platforms
---

React Native 不僅適用於 Android 和 iOS 裝置——我們的合作夥伴與社群還維護了將 React Native 帶到其他平台的專案，例如：

**來自合作夥伴**

- [React Native macOS](https://github.com/microsoft/react-native-macos) - 適用於 macOS 和 Cocoa 的 React Native。
- [React Native Windows](https://github.com/microsoft/react-native-windows) - 適用於微軟通用 Windows 平台 (UWP) 的 React Native。

**來自社群**

- [alita](https://github.com/areslabs/alita) - 一個實驗性的、全面的 React Native 移植到微信小程式的專案。
- [React Native tvOS](https://github.com/react-native-tvos/react-native-tvos) - 適用於 Apple TV 和 Android TV 裝置的 React Native。
- [React Native Web](https://github.com/necolas/react-native-web) - 使用 React DOM 在網頁上執行 React Native。
- [Valence Native](https://github.com/valence-native/valence-native) - 一個 React Native 的封裝，使用 Qt 來支援 Linux、macOS 和 Windows。此專案是從已停止維護的 [Proton Native](https://github.com/kusti8/proton-native) 分叉而來。

## 建立你自己的 React Native 平台

目前從頭開始建立 React Native 平台的過程尚未有完善的文件記錄——即將到來的重新架構（[Fabric](/blog/2018/06/14/state-of-react-native-2018)）目標之一就是讓維護平台變得更容易。

### 打包

從 React Native 0.57 開始，你可以將你的 React Native 平台註冊到 React Native 的 JavaScript 打包工具 [Metro](https://metrobundler.dev/)。這意味著你可以將 `--platform example` 傳遞給 `npx react-native bundle`，它會尋找副檔名為 `.example.js` 的 JavaScript 檔案。

要將你的平台註冊到 RNPM，你的模組名稱必須符合以下模式之一：

- `react-native-example` - It will search all top-level modules that start with `react-native-`
- `@org/react-native-example` - It will search for modules that start with `react-native-` under any scope
- `@react-native-example/module` - It will search in all modules under scopes with names starting with `@react-native-`

你還必須在你的 `package.json` 中加入以下條目：

```json
{
  "rnpm": {
    "haste": {
      "providesModuleNodeModules": ["react-native-example"],
      "platforms": ["example"]
    }
  }
}
```

`"providesModuleNodeModules"` 是一個會被加入 Haste 模組搜尋路徑的模組陣列，而 `"platforms"` 是一個會被加入為有效平台的平台後綴陣列。