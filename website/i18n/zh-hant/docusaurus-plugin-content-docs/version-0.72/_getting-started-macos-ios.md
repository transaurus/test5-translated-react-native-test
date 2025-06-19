import RemoveGlobalCLI from './\_remove-global-cli.md';
import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 安裝依賴套件

您需要安裝 Node、Watchman、React Native 命令行工具、Xcode 和 CocoaPods。

雖然您可以使用任何編輯器開發應用程式，但必須安裝 Xcode 才能設置必要的工具來為 iOS 構建 React Native 應用程式。

### Node 與 Watchman

建議使用 [Homebrew](http://brew.sh/) 安裝 Node 和 Watchman。安裝 Homebrew 後，在終端機中執行以下命令：

```shell
brew install node
brew install watchman
```

如果系統已安裝 Node，請確保版本為 16 或更新。

[Watchman](https://facebook.github.io/watchman) 是 Facebook 開發的檔案系統監控工具。強烈建議安裝以提升效能。

### Xcode

請使用 **最新版本** 的 Xcode。

最簡便的安裝方式是透過 [Mac App Store](https://itunes.apple.com/us/app/xcode/id497799835?mt=12)。安裝 Xcode 會同時安裝 iOS 模擬器與構建 iOS 應用程式所需的工具。

#### 命令行工具

您還需要安裝 Xcode 命令行工具。開啟 Xcode，從選單中選擇 **Settings... (或 Preferences...)**，進入 Locations 面板，從 Command Line Tools 下拉選單中選擇最新版本進行安裝。

![Xcode 命令行工具](/docs/assets/GettingStartedXcodeCommandLineTools.png)

#### 在 Xcode 中安裝 iOS 模擬器

要安裝模擬器，開啟 **Xcode > Settings... (或 Preferences...)** 並選擇 **Platforms (或 Components)** 標籤頁。選擇與您想使用的 iOS 版本對應的模擬器。

#### CocoaPods

[CocoaPods](https://cocoapods.org/) 是 iOS 的依賴管理工具之一。CocoaPods 是一個 Ruby [gem](https://en.wikipedia.org/wiki/RubyGems)。您可以使用 macOS 最新版本內建的 Ruby 安裝 CocoaPods。

更多資訊請參閱 [CocoaPods 入門指南](https://guides.cocoapods.org/using/getting-started.html)。

### React Native 命令行工具

React Native 內建命令行工具。我們建議使用 Node.js 自帶的 `npx` 在運行時存取當前版本，而非全域安裝特定版本的 CLI。透過 `npx react-native <command>` 執行命令時，會自動下載並運行當前穩定版的 CLI。

## 建立新應用程式

<RemoveGlobalCLI />

您可以使用 React Native 內建命令行工具生成新專案。以下指令將建立名為 "AwesomeProject" 的新專案：

```shell
npx react-native@latest init AwesomeProject
```

若您是要將 React Native 整合至現有應用程式、專案中已安裝 [Expo](https://docs.expo.dev/bare/installing-expo-modules/)，或是為現有 React Native 專案添加 iOS 支援（參見 [與現有應用程式整合](integration-with-existing-apps.md)），則無需執行此步驟。您也可以使用第三方 CLI（如 [Ignite CLI](https://github.com/infinitered/ignite)）初始化 React Native 專案。

:::info

If you are having trouble with iOS, try to reinstall the dependencies by running:

1. `cd ios` to navigate to the `ios` folder.
2. `bundle install` to install [Bundler](https://bundler.io/)
3. `bundle exec pod install` to install the iOS dependencies managed by CocoaPods.

:::

### [選用] 使用特定版本或模板

若想以特定 React Native 版本建立新專案，可使用 `--version` 參數：

```shell
npx react-native@X.XX.X init AwesomeProject --version X.XX.X
```

亦可透過 `--template` 參數使用自訂 React Native 模板啟動專案。

> **注意** 若上述指令執行失敗，可能是電腦中安裝了舊版全域 `react-native` 或 `react-native-cli`。請嘗試解除安裝 CLI 後改用 `npx` 執行。

### [選用] 環境設定

自 React Native 0.69 版起，可透過模板提供的 `.xcode.env` 檔案設定 Xcode 環境。

The `.xcode.env` file contains an environment variable to export the path to the `node` executable in the `NODE_BINARY` variable.
This is the **suggested approach** to decouple the build infrastructure from the system version of `node`. You should customize this variable with your own path or your own `node` version manager, if it differs from the default.

此外，還可添加其他環境變數，並在建置指令階段引用 `.xcode.env` 檔案。若需執行依賴特定環境的指令，此為**建議做法**：能讓建置階段與特定環境解耦。

:::info
If you are already using [NVM](http://nvm.sh/) (a command which helps you install and switch between versions of Node.js) and [zsh](https://ohmyz.sh/), you might want to move the code that initialize NVM from your `~/.zshrc` into a `~/.zshenv` file to help Xcode find your Node executable:

```zsh
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
```

You might also want to ensure that all "shell script build phase" of your Xcode project, is using `/bin/zsh` as its shell.
:::

## 執行 React Native 應用程式

### 步驟 1：啟動 Metro

首先需啟動 Metro——React Native 內建的 JavaScript 打包工具。Metro「接收入口檔案與各項參數，回傳包含所有程式碼與依賴項的單一 JavaScript 檔案」——[Metro 文件](https://metrobundler.dev/docs/concepts)

請在 React Native 專案資料夾內執行以下指令啟動 Metro：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm start
```

</TabItem>
<TabItem value="yarn">

```shell
yarn start
```

</TabItem>
</Tabs>

> 若熟悉網頁開發，Metro 類似 React Native 應用中的 webpack。與 Swift 或 Objective-C 不同，JavaScript 不需編譯——React Native 亦然。打包雖不等同編譯，但能提升啟動效能，並將部分平台專用 JavaScript 轉換為更通用的 JavaScript。

### 步驟 2：啟動應用程式

讓 Metro Bundler 在獨立終端機中運行。另開新終端機進入專案資料夾，執行：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run ios
```

</TabItem>
<TabItem value="yarn">

```shell
yarn ios
```

</TabItem>
</Tabs>

稍後即可在 iOS 模擬器中看見新應用程式運行。

![iOS 上的 AwesomeProject](/docs/assets/GettingStartediOSSuccess.png)

此為執行應用程式的方式之一，亦可直接透過 Xcode 運行。

> 若無法正常執行，請參閱[疑難排解](troubleshooting.md)頁面。

### 在實體裝置上執行

上述指令預設會在 iOS 模擬器執行應用程式。若需在實體 iOS 裝置運行，請參照[此處](running-on-device.md)說明操作。

### 修改應用程式

現在你已成功運行應用程式，接下來讓我們對其進行修改。

- 在文字編輯器中開啟 `App.tsx` 並編輯部分內容。
- 在 iOS 模擬器中按下 <kbd>Cmd ⌘</kbd> + <kbd>R</kbd> 重新載入應用程式，即可看到變更效果！

### 完成了！

恭喜！你已成功運行並修改了第一個 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

## 接下來呢？

- 若想將此 React Native 程式碼整合至現有應用程式，請參閱[整合指南](integration-with-existing-apps.md)。

如果想進一步了解 React Native，請查看 [React Native 簡介](getting-started)。