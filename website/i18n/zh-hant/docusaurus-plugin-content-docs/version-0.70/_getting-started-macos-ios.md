import RemoveGlobalCLI from './\_remove-global-cli.md';

## 安裝依賴套件

您需要安裝 Node、Watchman、React Native 命令行工具、Ruby 版本管理器、Xcode 和 CocoaPods。

雖然您可以使用任何喜歡的編輯器開發應用程式，但必須安裝 Xcode 才能設置必要的工具來為 iOS 構建 React Native 應用程式。

### Node 與 Watchman

我們建議使用 [Homebrew](http://brew.sh/) 安裝 Node 和 Watchman。安裝 Homebrew 後，在終端機中執行以下命令：

```shell
brew install node
brew install watchman
```

如果系統已安裝 Node，請確保版本為 14 或更新。

[Watchman](https://facebook.github.io/watchman) 是 Facebook 開發的檔案系統監控工具。強烈建議安裝以獲得更好的效能。

### Ruby

React Native 使用 `.ruby-version` 檔案確保 Ruby 版本符合需求。目前 macOS 13.2 預裝的 Ruby 2.6.10 版本**不符合**此版 React Native 的需求 (2.7.5)。建議安裝 Ruby 版本管理器並在系統中安裝正確的 Ruby 版本。

常見的 Ruby 版本管理器包括：

- [rbenv](https://github.com/rbenv/rbenv)
- [RVM](https://rvm.io/)
- [chruby](https://github.com/postmodern/chruby)
- [asdf-vm](https://github.com/asdf-vm) 搭配 [asdf-ruby](https://github.com/asdf-vm/asdf-ruby) 插件

若要檢查當前 Ruby 版本，可執行以下命令：

```
ruby --version
```

React Native 使用[此版本](https://github.com/facebook/react-native/blob/v0.70.7/.ruby-version)的 Ruby。您也可以在 RN 專案根目錄的 `.ruby-version` 檔案中找到專案所需的版本。

### Bundler

[Bundler](https://bundler.io/) 是管理專案 Ruby 依賴套件的 Ruby gem。我們需要 Ruby 來安裝 CocoaPods，使用 Bundler 可確保所有依賴套件版本一致，使專案正常運作。

若想了解更多關於此工具的必要性，可閱讀[這篇文章](https://bundler.io/guides/rationale.html#bundlers-purpose-and-rationale)。

### Xcode

請使用 Xcode 的**最新版本**。

最簡單的安裝方式是透過 [Mac App Store](https://itunes.apple.com/us/app/xcode/id497799835?mt=12)。安裝 Xcode 時會同時安裝 iOS 模擬器和構建 iOS 應用程式所需的所有工具。

#### 命令行工具

您還需要安裝 Xcode 命令行工具。打開 Xcode，從選單中選擇「Preferences...」，進入 Locations 面板，從 Command Line Tools 下拉選單中選擇最新版本進行安裝。

![Xcode 命令行工具](/docs/assets/GettingStartedXcodeCommandLineTools.png)

#### 在 Xcode 中安裝 iOS 模擬器

要安裝模擬器，打開<strong>Xcode > Preferences...</strong> 並選擇 <strong>Components</strong> 標籤頁。選擇與您想使用的 iOS 版本對應的模擬器。

#### CocoaPods

[CocoaPods](https://cocoapods.org/) 是用 Ruby 開發的，可透過 macOS 預設的 Ruby 安裝。

如需更多資訊，請參閱 [CocoaPods 入門指南](https://guides.cocoapods.org/using/getting-started.html)。

### React Native 命令列介面

React Native 內建了命令列介面。與其全域安裝並管理特定版本的 CLI，我們建議您使用 Node.js 自帶的 `npx` 在執行時存取當前版本。透過 `npx react-native <command>`，系統會在執行命令時下載並執行當前穩定版的 CLI。

## 建立新應用程式

<RemoveGlobalCLI />

您可以使用 React Native 內建的命令列介面來生成新專案。讓我們建立一個名為「AwesomeProject」的新 React Native 專案：

```shell
npx react-native init AwesomeProject
```

如果您是將 React Native 整合到現有應用程式中，或已在專案中安裝 [Expo](https://docs.expo.dev/bare/installing-expo-modules/)，或是為現有 React Native 專案新增 Android 支援（請參閱[與現有應用程式整合](integration-with-existing-apps.md)），則無需此步驟。您也可以使用第三方 CLI（如 [Ignite CLI](https://github.com/infinitered/ignite)）來初始化 React Native 應用程式。

:::info

If you are having trouble with iOS, try to reinstall the dependencies by running:

1. `cd ios` to navigate to the
2. `bundle install` to install Bundler
   1. If needed: install a [Ruby Version Manager](#ruby) and update the Ruby version
3. `bundle exec pod install` to install the iOS dependencies.

:::

### [選用] 使用特定版本或模板

若想使用特定 React Native 版本建立新專案，可使用 `--version` 參數：

```shell
npx react-native init AwesomeProject --version X.XX.X
```

您也可以透過 `--template` 參數使用自訂模板（如 TypeScript）建立專案：

```shell
npx react-native init AwesomeTSProject --template react-native-template-typescript
```

> **注意** 若上述指令執行失敗，可能是您的電腦全域安裝了舊版 `react-native` 或 `react-native-cli`。請嘗試解除安裝 CLI 並改用 `npx` 執行。

### [選用] 設定環境

從 React Native 0.69 版開始，可透過模板提供的 `.xcode.env` 檔案設定 Xcode 環境。

The `.xcode.env` file contains an environment variable to export the path to the `node` executable in the `NODE_BINARY` variable.
This is the **suggested approach** to decouple the build infrastructure from the system version of `node`. You should customize this variable with your own path or your own `node` version manager, if it differs from the default.

此外，您還可新增其他環境變數，並在建置指令碼階段引入 `.xcode.env` 檔案。若需執行特定環境的指令碼，這是**建議做法**：它能讓建置階段與特定環境解耦。

## 執行 React Native 應用程式

### 步驟 1：啟動 Metro

首先需啟動 Metro——React Native 內建的 JavaScript 打包工具。Metro「接收一個入口檔案和各種選項，並回傳包含所有程式碼及其依賴項的單一 JavaScript 檔案」——[Metro 文件](https://metrobundler.dev/docs/concepts)

請在 React Native 專案目錄中執行 `npx react-native start` 以啟動 Metro：

```shell
npx react-native start
```

`react-native start` 會啟動 Metro 打包工具。

> 若使用 Yarn 套件管理器，在現有專案中執行 React Native 指令時，可用 `yarn` 替代 `npx`。

> 如果你熟悉網頁開發，Metro 的功能類似於 React Native 應用中的 webpack。與 Swift 或 Objective-C 不同，JavaScript 不需要編譯——React Native 也是如此。打包（bundling）不等同於編譯，但它能提升啟動效能，並將某些平台特定的 JavaScript 轉換為更廣泛支援的 JavaScript 程式碼。

### 步驟 2：啟動你的應用程式

讓 Metro Bundler 在它自己的終端機中持續運行。在你的 React Native 專案資料夾內開啟一個新的終端機，然後執行以下指令：

```shell
npx react-native run-ios
```

你應該很快就能在 iOS 模擬器中看到新應用程式運行。

![AwesomeProject on iOS](/docs/assets/GettingStartediOSSuccess.png)

`npx react-native run-ios` 是運行應用程式的方式之一，你也可以直接透過 Xcode 來運行。

> 如果無法順利運行，請參閱 [疑難排解](troubleshooting.md) 頁面。

### 在實體裝置上運行

上述指令預設會將你的應用程式運行在 iOS 模擬器上。如果想在實際的 iOS 裝置上運行，請按照 [這裡](running-on-device.md) 的說明操作。

### 修改你的應用程式

現在你已成功運行應用程式，接下來讓我們對它進行一些修改。

- 用你選擇的文字編輯器開啟 `App.js` 並編輯部分內容。
- 在 iOS 模擬器中按下 `⌘R` 重新載入應用程式，即可看到變更效果！

### 大功告成！

恭喜！你已成功運行並修改了第一個 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

## 接下來呢？

- 如果想將這段新的 React Native 程式碼整合到現有應用中，請參閱 [整合指南](integration-with-existing-apps.md)。

如果想更深入瞭解 React Native，請查看 [React Native 簡介](getting-started)。