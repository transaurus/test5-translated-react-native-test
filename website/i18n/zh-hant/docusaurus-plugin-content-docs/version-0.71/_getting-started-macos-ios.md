import RemoveGlobalCLI from './\_remove-global-cli.md';

## 安裝依賴套件

您需要安裝 Node、Watchman、React Native 命令行界面、Ruby 版本管理器、Xcode 和 CocoaPods。

雖然您可以選擇任何編輯器來開發應用程式，但必須安裝 Xcode 才能設置必要的工具來為 iOS 構建 React Native 應用程式。

### Node 與 Watchman

我們建議使用 [Homebrew](http://brew.sh/) 安裝 Node 和 Watchman。安裝 Homebrew 後，在終端機中執行以下命令：

```shell
brew install node
brew install watchman
```

如果您的系統已安裝 Node，請確保版本為 14 或更新。

[Watchman](https://facebook.github.io/watchman) 是 Facebook 開發的檔案系統監控工具。強烈建議安裝以獲得更好的效能。

### Ruby

[Ruby](https://www.ruby-lang.org/en/) 是一種通用程式語言。React Native 在某些與 iOS 依賴管理相關的腳本中使用 Ruby。與所有程式語言一樣，Ruby 也有多個不同版本。

React Native 使用 `.ruby-version` 文件確保您的 Ruby 版本符合需求。目前 macOS 13.2 預裝的 Ruby 2.6.10 **不符合** 此版本 React Native 的要求 (2.7.6)。建議安裝 Ruby 版本管理器並在系統中安裝正確的 Ruby 版本。

常見的 Ruby 版本管理器包括：

- [rbenv](https://github.com/rbenv/rbenv)
- [RVM](https://rvm.io/)
- [chruby](https://github.com/postmodern/chruby)
- [asdf-vm](https://github.com/asdf-vm) 搭配 [asdf-ruby](https://github.com/asdf-vm/asdf-ruby) 插件

要檢查當前 Ruby 版本，可執行以下命令：

```
ruby --version
```

React Native 使用 [此版本](https://github.com/facebook/react-native/blob/v0.71.3/.ruby-version) 的 Ruby。您也可以在 RN 專案根目錄的 `.ruby-version` 文件中找到專案所需的版本。

### Ruby 的 Bundler

Ruby 使用 **gems** 的概念來管理其依賴項。您可以將 gem 視為 NPM 中的套件、Homebrew 中的配方或 CocoaPods 中的單個 pod。

Ruby 的 [Bundler](https://bundler.io/) 是一個幫助管理專案 Ruby 依賴項的 gem。我們需要 Ruby 來安裝 CocoaPods，使用 Bundler 可確保所有依賴項一致且專案正常運作。

若想了解更多關於此工具的必要性，可閱讀 [這篇文章](https://bundler.io/guides/rationale.html#bundlers-purpose-and-rationale)。

### Xcode

請使用 Xcode 的 **最新版本**。

最簡單的安裝方式是透過 [Mac App Store](https://itunes.apple.com/us/app/xcode/id497799835?mt=12)。安裝 Xcode 時會同時安裝 iOS 模擬器和構建 iOS 應用程式所需的所有工具。

#### 命令行工具

您還需要安裝 Xcode 命令行工具。打開 Xcode，從選單中選擇「Preferences...」，進入 Locations 面板，從 Command Line Tools 下拉選單中選擇最新版本進行安裝。

![Xcode 命令行工具](/docs/assets/GettingStartedXcodeCommandLineTools.png)

#### 在 Xcode 中安裝 iOS 模擬器

To install a simulator, open <strong>Xcode > Preferences...</strong> and select the <strong>Components</strong> tab. Select a simulator with the corresponding version of iOS you wish to use.

If you are using Xcode version 14.0 or greater to install a simulator, open **Xcode > Settings > Platforms** tab, then click "+" icon and select **iOS…** option.

#### CocoaPods

[CocoaPods](https://cocoapods.org/)是iOS可用的依賴管理系統之一。它是用Ruby構建的，您可以使用在前幾步中配置的Ruby版本來安裝它。

更多資訊，請參閱[CocoaPods入門指南](https://guides.cocoapods.org/using/getting-started.html)。

### React Native 命令行界面

React Native內建了一個命令行界面。我們建議您不要全局安裝和管理特定版本的CLI，而是使用Node.js自帶的`npx`在運行時訪問當前版本。使用`npx react-native <command>`時，將在運行命令時下載並執行當前穩定版本的CLI。

## 創建新應用程式

<RemoveGlobalCLI />

您可以使用React Native內建的命令行界面來生成一個新項目。讓我們創建一個名為"AwesomeProject"的新React Native項目：

```shell
npx react-native@latest init AwesomeProject
```

如果您正在將React Native集成到現有應用程式中，或者已在項目中安裝了[Expo](https://docs.expo.dev/bare/installing-expo-modules/)，或者正在為現有React Native項目添加iOS支持（請參閱[與現有應用程式集成](integration-with-existing-apps.md)），則不需要此步驟。您也可以使用第三方CLI（如[Ignite CLI](https://github.com/infinitered/ignite)）來初始化您的React Native應用程式。

:::info

如果您在iOS上遇到問題，請嘗試通過運行以下命令重新安裝依賴項：

1. `cd ios` 導航到iOS目錄
2. `bundle install` 安裝Bundler
   1. 如果需要：安裝[Ruby版本管理器](#ruby)並更新Ruby版本
3. `bundle exec pod install` 安裝iOS依賴項

:::

### [可選] 使用特定版本或模板

如果您想使用特定的React Native版本開始新項目，可以使用`--version`參數：

```shell
npx react-native@X.XX.X init AwesomeProject --version X.XX.X
```

您也可以使用`--template`參數以自定義的React Native模板開始項目。

> **注意** 如果上述命令失敗，可能是因為您的電腦上全局安裝了舊版本的`react-native`或`react-native-cli`。嘗試卸載CLI並使用`npx`運行CLI。

### [可選] 配置您的環境

從React Native 0.69版本開始，可以使用模板提供的`.xcode.env`文件來配置Xcode環境。

The `.xcode.env` file contains an environment variable to export the path to the `node` executable in the `NODE_BINARY` variable.
This is the **suggested approach** to decouple the build infrastructure from the system version of `node`. You should customize this variable with your own path or your own `node` version manager, if it differs from the default.

除此之外，還可以添加任何其他環境變量，並在構建腳本階段中引用`.xcode.env`文件。如果您需要運行需要特定環境的腳本，這是**建議的方法**：它允許將構建階段與特定環境解耦。

## 運行您的React Native應用程式

### 步驟1：啟動Metro

首先，您需要啟動 Metro——這是 React Native 內建的 JavaScript 打包工具。Metro「接收一個入口檔案和各種選項，並返回包含所有程式碼及其依賴項的單一 JavaScript 檔案。」——[Metro 文件](https://metrobundler.dev/docs/concepts)

要啟動 Metro，請在您的 React Native 專案資料夾中執行 `npx react-native start`：

```shell
npx react-native start
```

`react-native start` 會啟動 Metro 打包工具。

> 如果您使用 Yarn 套件管理器，在現有專案中執行 React Native 指令時，可以用 `yarn` 替代 `npx`。

> 如果您熟悉網頁開發，Metro 類似於 React Native 應用程式中的 webpack。與 Swift 或 Objective-C 不同，JavaScript 不需要編譯——React Native 也是如此。打包（bundling）不等同於編譯，但它能提升啟動效能，並將某些平台特定的 JavaScript 轉換為更廣泛支援的 JavaScript。

### 步驟 2：啟動您的應用程式

讓 Metro 打包工具在它自己的終端機中運行。在您的 React Native 專案資料夾中開啟一個新的終端機，然後執行以下指令：

```shell
npx react-native run-ios
```

您應該很快就能看到新應用程式在 iOS 模擬器中運行。

![iOS 上的 AwesomeProject](/docs/assets/GettingStartediOSSuccess.png)

`npx react-native run-ios` 是運行應用程式的一種方式。您也可以直接在 Xcode 中運行它。

> 如果無法順利運行，請參閱[疑難排解](troubleshooting.md)頁面。

### 在實體裝置上運行

上述指令預設會自動在 iOS 模擬器中運行您的應用程式。如果想在實際的 iOS 裝置上運行，請按照[這裡](running-on-device.md)的說明操作。

### 修改您的應用程式

現在您已成功運行應用程式，讓我們來修改它。

- 在您選擇的文字編輯器中開啟 `App.tsx` 並編輯部分程式碼。
- 在 iOS 模擬器中按下 `⌘R` 重新載入應用程式，即可看到變更！

### 完成了！

恭喜！您已成功運行並修改了第一個 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

## 接下來呢？

- 如果想將這段新的 React Native 程式碼整合到現有應用程式中，請參閱[整合指南](integration-with-existing-apps.md)。

如果想深入瞭解 React Native，請查閱 [React Native 簡介](getting-started)。