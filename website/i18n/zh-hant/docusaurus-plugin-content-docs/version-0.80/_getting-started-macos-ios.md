## 安裝依賴套件

您需要安裝 Node、Watchman、React Native 命令行工具、Xcode 和 CocoaPods。

雖然您可以使用任何編輯器來開發應用程式，但必須安裝 Xcode 才能設置必要的工具來為 iOS 構建 React Native 應用程式。

### Node 與 Watchman

我們建議使用 [Homebrew](https://brew.sh/) 安裝 Node 和 Watchman。安裝 Homebrew 後，在終端機中執行以下命令：

```shell
brew install node
brew install watchman
```

如果您的系統已安裝 Node，請確保版本為 18.18 或更新。

[Watchman](https://facebook.github.io/watchman) 是 Facebook 開發的檔案系統變更監控工具。強烈建議安裝以提升效能。

### Xcode

請使用 **最新版本** 的 Xcode。

最簡單的安裝方式是透過 [Mac App Store](https://itunes.apple.com/us/app/xcode/id497799835?mt=12)。安裝 Xcode 會同時安裝 iOS 模擬器與構建 iOS 應用程式所需的所有工具。

#### 命令行工具

您還需要安裝 Xcode 命令行工具。打開 Xcode，從選單中選擇 **Settings... (或 Preferences...)**，進入 Locations 面板，從 Command Line Tools 下拉選單中選擇最新版本進行安裝。

![Xcode Command Line Tools](/docs/assets/GettingStartedXcodeCommandLineTools.png)

#### 在 Xcode 中安裝 iOS 模擬器

要安裝模擬器，打開 **Xcode > Settings... (或 Preferences...)** 並選擇 **Platforms (或 Components)** 標籤頁。選擇您想使用的 iOS 版本對應的模擬器。

如果您使用 Xcode 14.0 或更高版本安裝模擬器，請打開 **Xcode > Settings > Platforms** 標籤頁，點擊 "+" 圖標並選擇 **iOS…** 選項。

#### CocoaPods

[CocoaPods](https://cocoapods.org/) 是 iOS 的依賴管理工具之一。CocoaPods 是一個 Ruby [gem](https://en.wikipedia.org/wiki/RubyGems)。您可以使用 macOS 最新版本內建的 Ruby 安裝 CocoaPods。

更多資訊請參閱 [CocoaPods 入門指南](https://guides.cocoapods.org/using/getting-started.html)。

### [選項] 環境設定

從 React Native 0.69 版開始，可以使用模板提供的 `.xcode.env` 檔案來設定 Xcode 環境。

The `.xcode.env` file contains an environment variable to export the path to the `node` executable in the `NODE_BINARY` variable.
This is the **suggested approach** to decouple the build infrastructure from the system version of `node`. You should customize this variable with your own path or your own `node` version manager, if it differs from the default.

此外，您可以在 `.xcode.env` 檔案中添加任何其他環境變數，並在構建腳本階段中引入該檔案。如果您需要執行某些需要特定環境的腳本，這是 **建議的做法**：它可以將構建階段與特定環境解耦。

:::info
If you are already using [NVM](https://nvm.sh/) (a command which helps you install and switch between versions of Node.js) and [zsh](https://ohmyz.sh/), you might want to move the code that initialize NVM from your `~/.zshrc` into a `~/.zshenv` file to help Xcode find your Node executable:

```zsh
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
```

You might also want to ensure that all "shell script build phase" of your Xcode project, is using `/bin/zsh` as its shell.
:::

<h3>That's it!</h3>

恭喜！您已成功設定開發環境。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- 若需將此 React Native 程式碼整合至現有應用程式，請參閱[整合指南](integration-with-existing-apps.md)。
- 若想深入瞭解 React Native，請參閱[React Native 簡介](getting-started)。