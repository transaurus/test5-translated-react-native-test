---
id: out-of-tree-platforms
title: Out-of-Tree Platforms
---

React Native 不僅適用於 Android 和 iOS 設備——我們的合作夥伴與社群還維護了將 React Native 帶入其他平台的專案，例如：

**來自合作夥伴**

- [React Native macOS](https://github.com/microsoft/react-native-macos) - 適用於 macOS 和 Cocoa 的 React Native。
- [React Native Windows](https://github.com/microsoft/react-native-windows) - 適用於微軟通用 Windows 平台 (UWP) 的 React Native。
- [React Native visionOS](https://github.com/callstack/react-native-visionos) - 適用於 Apple visionOS 的 React Native。

**來自社群**

- [React Native tvOS](https://github.com/react-native-tvos/react-native-tvos) - 適用於 Apple TV 和 Android TV 設備的 React Native。
- [React Native Web](https://github.com/necolas/react-native-web) - 使用 React DOM 在網頁上運行的 React Native。
- [React Native Skia](https://github.com/react-native-skia/react-native-skia) - 使用 [Skia](https://skia.org/) 作為渲染器的 React Native。目前支援 Linux 和 macOS。

## 創建自己的 React Native 平台

目前從頭開始創建 React Native 平台的過程尚未有完善的文檔記錄——即將到來的重新架構（[Fabric](/blog/2018/06/14/state-of-react-native-2018)）目標之一就是讓維護平台變得更加容易。

### 打包

As of React Native 0.57 you can now register your React Native platform with React Native's JavaScript bundler, [Metro](https://metrobundler.dev/). This means you can pass `--platform example` to `npx react-native bundle`, and it will look for JavaScript files with the `.example.js` suffix.

要將您的平台註冊到 RNPM，您的模組名稱必須符合以下模式之一：

- `react-native-example` - It will search all top-level modules that start with `react-native-`
- `@org/react-native-example` - It will search for modules that start with `react-native-` under any scope
- `@react-native-example/module` - It will search in all modules under scopes with names starting with `@react-native-`

您還需要在 `package.json` 中添加如下條目：

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

`"providesModuleNodeModules"` 是一個會被添加到 Haste 模組搜尋路徑的模組陣列，而 `"platforms"` 是一個會被添加為有效平台的後綴陣列。