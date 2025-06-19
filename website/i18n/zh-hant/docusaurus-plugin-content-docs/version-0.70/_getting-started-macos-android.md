import RemoveGlobalCLI from './\_remove-global-cli.md';

## 安裝依賴套件

您需要安裝 Node、Watchman、React Native 命令行工具、JDK 以及 Android Studio。

雖然您可以選擇任何編輯器來開發應用程式，但必須安裝 Android Studio 才能設置建置 React Native Android 應用所需的工具鏈。

<h3>Node &amp; Watchman</h3>

建議透過 [Homebrew](http://brew.sh/) 安裝 Node 和 Watchman。安裝 Homebrew 後，在終端機執行以下命令：

```shell
brew install node
brew install watchman
```

若系統已安裝 Node，請確認版本為 14 或更新。

[Watchman](https://facebook.github.io/watchman) 是 Facebook 開發的檔案系統監控工具，強烈建議安裝以提升效能。

<h3>Java Development Kit</h3>

We recommend installing the OpenJDK distribution called Azul **Zulu** using [Homebrew](http://brew.sh/). Run the following commands in a Terminal after installing Homebrew:

```shell
brew install --cask zulu@11

# Get path to where cask was installed to double-click installer
brew info --cask zulu@11
```

安裝 JDK 後，請更新 `JAVA_HOME` 環境變數。若依上述步驟安裝，JDK 路徑通常為 `/Library/Java/JavaVirtualMachines/zulu-11.jdk/Contents/Home`

Zulu OpenJDK 發行版提供 **Intel 和 M1 晶片 Mac 專用**的 JDK，可確保在 M1 Mac 上獲得比 Intel 版 JDK 更快的建置速度。

若系統已安裝 JDK，建議使用 JDK 11 版本。使用更高版本可能會遇到相容性問題。

<h3>Android development environment</h3>

對於 Android 開發新手來說，設置開發環境可能較為繁瑣。若您已有 Android 開發經驗，則只需進行部分配置。無論如何，請仔細遵循後續步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在安裝精靈中，請確保勾選以下所有項目：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`

接著點擊「Next」安裝所有元件。

> 若選項呈灰色不可勾選，後續仍可安裝這些元件。

完成安裝並顯示歡迎畫面後，請繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新版 Android SDK。但使用原生碼建置 React Native 應用需特定安裝 `Android 12 (S)` SDK，可透過 Android Studio 的 SDK 管理員安裝其他版本。

開啟 Android Studio 後，點擊「More Actions」按鈕並選擇「SDK Manager」。

![Android Studio 歡迎畫面](/docs/assets/GettingStartedAndroidStudioWelcomeMacOS.png)

> 也可在 Android Studio 的「Settings」對話框中找到 SDK Manager，路徑為 **Languages & Frameworks** → **Android SDK**。

在 SDK Manager 中選擇「SDK Platforms」標籤頁，勾選右下角的「Show Package Details」。找到並展開 `Android 12 (S)` 項目，確認以下元件已勾選：

- `Android SDK Platform 31`
- `Intel x86 Atom_64 System Image` or `Google APIs Intel x86 Atom System Image` or (for Apple M1 Silicon) `Google APIs ARM 64 v8a System Image`

接著，切換至「SDK Tools」標籤頁，同樣勾選「顯示套件詳細資訊」。找到並展開「Android SDK Build-Tools」項目，確保已選取 `31.0.0` 版本。

最後點擊「Apply」按鈕以下載並安裝 Android SDK 及相關建置工具。

<h4>3. Configure the ANDROID_SDK_ROOT environment variable</h4>

React Native 工具需要設定某些環境變數才能建置包含原生代碼的應用程式。

請將以下內容加入您的 `$HOME/.bash_profile` 或 `$HOME/.bashrc` 設定檔（若使用 `zsh` 則為 `~/.zprofile` 或 `~/.zshrc`）：

```shell
export ANDROID_SDK_ROOT=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_SDK_ROOT/emulator
export PATH=$PATH:$ANDROID_SDK_ROOT/platform-tools
```

> `.bash_profile` 是 `bash` 專用的設定檔。若使用其他 shell，需編輯對應的 shell 設定檔。

Type `source $HOME/.bash_profile` for `bash` or `source $HOME/.zprofile` to load the config into your current shell. Verify that ANDROID_SDK_ROOT has been set by running `echo $ANDROID_SDK_ROOT` and the appropriate directories have been added to your path by running `echo $PATH`.

> 請務必使用正確的 Android SDK 路徑。您可以在 Android Studio 的「Settings」對話框中，透過 **Languages & Frameworks** → **Android SDK** 找到 SDK 的實際位置。

<h3>React Native Command Line Interface</h3>

React Native 內建命令列介面。我們建議透過 Node.js 自帶的 `npx` 工具在執行時存取當前版本，而非全域安裝特定 CLI 版本。使用 `npx react-native <command>` 時，系統會自動下載並執行最新穩定版的 CLI。

<h2>Creating a new application</h2>

<RemoveGlobalCLI />

React Native 內建命令列介面可用於生成新專案。您無需全域安裝任何套件，直接透過 Node.js 自帶的 `npx` 工具即可使用。以下示範建立名為「AwesomeProject」的新專案：

```shell
npx react-native init AwesomeProject
```

若您要將 React Native 整合至現有應用程式、專案中已安裝 [Expo](https://docs.expo.dev/bare/installing-expo-modules/)，或為現有 React Native 專案添加 Android 支援（參見[與現有應用程式整合](integration-with-existing-apps.md)），則無需此步驟。您也可使用第三方 CLI（如 [Ignite CLI](https://github.com/infinitered/ignite)）初始化專案。

<h3>[Optional] Using a specific version or template</h3>

若要使用特定 React Native 版本建立專案，可加入 `--version` 參數：

```shell
npx react-native init AwesomeProject --version X.XX.X
```

您也可透過 `--template` 參數使用自訂模板（如 TypeScript）建立專案：

```shell
npx react-native init AwesomeTSProject --template react-native-template-typescript
```

<h2>Preparing the Android device</h2>

執行 React Native Android 應用程式需準備 Android 裝置，可以是實體裝置或透過 Android 虛擬裝置 (AVD) 在電腦上模擬。

無論採用何種方式，均需將裝置設定為開發模式以執行 Android 應用程式。

<h3>Using a physical device</h3>

如果您擁有實體 Android 裝置，可以透過 USB 線連接電腦後，依照[此處說明](running-on-device.md)將其用作開發環境來取代 AVD。

<h3>Using a virtual device</h3>

若使用 Android Studio 開啟 `./AwesomeProject/android` 專案，可透過點選 Android Studio 內的「AVD Manager」圖示（如下所示）查看可用虛擬裝置清單：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

若剛安裝 Android Studio，您可能需要[建立新 AVD](https://developer.android.com/studio/run/managing-avds.html)。選擇「Create Virtual Device...」後，從清單任選手機型號點擊「Next」，接著選取 **S** API Level 31 系統映像檔。

點擊「Next」後按「Finish」完成 AVD 建立。此時點擊 AVD 旁的綠色三角形按鈕即可啟動模擬器，並繼續後續步驟。

<h2>Running your React Native application</h2>

<h3>Step 1: Start Metro</h3>

First, you will need to start Metro, the JavaScript bundler that ships with React Native. Metro "takes in an entry file and various options, and returns a single JavaScript file that includes all your code and its dependencies."—[Metro Docs](https://metrobundler.dev/docs/concepts)

To start Metro, run `npx react-native start` inside your React Native project folder:

```shell
npx react-native start
```

`react-native start` 指令會啟動 Metro 打包工具。

> 若使用 Yarn 套件管理器，在既有專案中執行 React Native 指令時可用 `yarn` 取代 `npx`。

> 熟悉網頁開發者可以將 Metro 視為 React Native 的 webpack。不同於 Kotlin 或 Java，JavaScript 與 React Native 皆不需編譯。打包雖不等同於編譯，但能提升啟動效能並將部分平台專用 JavaScript 轉換為通用格式。

<h3>Step 2: Start your application</h3>

讓 Metro 打包工具在獨立終端機運行，另開新終端機進入專案目錄後執行：

```shell
npx react-native run-android
```

若設定正確，稍後即可在 Android 模擬器中看見新應用程式運行。

![Android 上的 AwesomeProject](/docs/assets/GettingStartedAndroidSuccessMacOS.png)

`npx react-native run-android` 是執行應用程式的方式之一，您亦可直接透過 Android Studio 運行。

> 若執行失敗，請參閱[疑難排解頁面](troubleshooting.md)。

<h3>Modifying your app</h3>

成功運行應用程式後，現在開始修改：

- Open `App.js` in your text editor of choice and edit some lines.
- Press the `R` key twice or select `Reload` from the Developer Menu (`⌘M`) to see your changes!

<h3>That's it!</h3>

恭喜！您已成功運行並修改第一個 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- If you want to add this new React Native code to an existing application, check out the [Integration guide](integration-with-existing-apps.md).

若想深入學習 React Native，請查看[React Native 簡介](getting-started)。