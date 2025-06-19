import RemoveGlobalCLI from './\_remove-global-cli.md';

## 安裝依賴套件

您需要安裝 Node、React Native 命令列介面、JDK 以及 Android Studio。

雖然您可以使用任何偏好的編輯器開發應用程式，但必須安裝 Android Studio 才能設置建置 React Native Android 應用程式所需的工具。

<h3>Node</h3>

請依照 [Linux 發行版的安裝說明](https://nodejs.org/en/download/package-manager/) 安裝 Node 14 或更新版本。

<h3>Java Development Kit</h3>

React Native currently recommends version 11 of the Java SE Development Kit (JDK). You may encounter problems using higher JDK versions. You may download and install [OpenJDK](http://openjdk.java.net) from [AdoptOpenJDK](https://adoptopenjdk.net/) or your system packager.

<h3>Android development environment</h3>

如果您是 Android 開發的新手，設置開發環境可能會有些繁瑣。如果您已經熟悉 Android 開發，則可能需要進行一些配置。無論哪種情況，請務必仔細遵循接下來的幾個步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在 Android Studio 安裝精靈中，請確保勾選以下所有項目的方塊：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`

然後，點擊「Next」安裝所有這些元件。

> 如果複選框呈現灰色，您稍後仍有機會安裝這些元件。

安裝完成並顯示歡迎畫面後，請繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新的 Android SDK。然而，使用原生程式碼建置 React Native 應用程式需要特定的 `Android 12 (S)` SDK。您可以透過 Android Studio 的 SDK 管理器安裝其他 Android SDK。

為此，請開啟 Android Studio，點擊「Configure」按鈕並選擇「SDK Manager」。

> SDK 管理器也可以在 Android Studio 的「Settings」對話方塊中找到，位於 **Languages & Frameworks** → **Android SDK**。

在 SDK 管理器中選擇「SDK Platforms」標籤，然後勾選右下角的「Show Package Details」。尋找並展開 `Android 12 (S)` 項目，確保勾選以下項目：

- `Android SDK Platform 31`
- `Intel x86 Atom_64 System Image` 或 `Google APIs Intel x86 Atom System Image`

接著，選擇「SDK Tools」標籤，同樣勾選「Show Package Details」。尋找並展開「Android SDK Build-Tools」項目，確保選中 `31.0.0` 版本。

最後，點擊「Apply」下載並安裝 Android SDK 及相關建置工具。

<h4>3. Configure the ANDROID_SDK_ROOT environment variable</h4>

React Native 工具需要設置一些環境變數才能使用原生程式碼建置應用程式。

將以下內容加入您的 `$HOME/.bash_profile` 或 `$HOME/.bashrc` 設定檔（如果您使用 `zsh`，則為 `~/.zprofile` 或 `~/.zshrc`）：

```shell
export ANDROID_SDK_ROOT=$HOME/Library/Android/Sdk
export PATH=$PATH:$ANDROID_SDK_ROOT/emulator
export PATH=$PATH:$ANDROID_SDK_ROOT/platform-tools
```

> `.bash_profile` 是 `bash` 專用的。如果您使用其他 shell，則需要編輯相應的 shell 專用設定檔。

Type `source $HOME/.bash_profile` for `bash` or `source $HOME/.zprofile` to load the config into your current shell. Verify that ANDROID_SDK_ROOT has been set by running `echo $ANDROID_SDK_ROOT` and the appropriate directories have been added to your path by running `echo $PATH`.

> 請確保使用正確的 Android SDK 路徑。您可以在 Android Studio 的「設定」對話框中找到 SDK 的實際位置，路徑為 **Languages & Frameworks** → **Android SDK**。

<h3>Watchman</h3>

請遵循 [Watchman 安裝指南](https://facebook.github.io/watchman/docs/install#buildinstall) 從原始碼編譯並安裝 Watchman。

> [Watchman](https://facebook.github.io/watchman/docs/install) 是 Facebook 開發的檔案系統變更監控工具。強烈建議安裝以提升效能並增加特定邊緣案例的相容性（翻譯：您或許可以不安裝此工具，但效果可能因情況而異；現在安裝可能避免日後的麻煩）。

<h3>React Native Command Line Interface</h3>

React Native 內建命令列介面。與其全域安裝並管理特定版本的 CLI，我們建議您使用 Node.js 自帶的 `npx` 在執行時存取當前版本。透過 `npx react-native <command>`，系統會在執行命令時下載並執行當前穩定版的 CLI。

<h2>Creating a new application</h2>

<RemoveGlobalCLI />

React Native 內建命令列介面，可用於生成新專案。您無需全域安裝任何套件，直接使用 Node.js 自帶的 `npx` 即可存取此功能。讓我們建立一個名為「AwesomeProject」的新 React Native 專案：

```shell
npx react-native init AwesomeProject
```

如果您是將 React Native 整合至現有應用程式、或在專案中安裝了 [Expo](https://docs.expo.dev/bare/installing-expo-modules/)、或為現有 React Native 專案新增 Android 支援（參見 [與現有應用程式整合](integration-with-existing-apps.md)），則無需此步驟。您也可以使用第三方 CLI（如 [Ignite CLI](https://github.com/infinitered/ignite)）來初始化 React Native 應用程式。

<h3>[Optional] Using a specific version or template</h3>

若想以特定 React Native 版本建立新專案，可使用 `--version` 參數：

```shell
npx react-native init AwesomeProject --version X.XX.X
```

您也可以使用 `--template` 參數，以自訂範本（如 TypeScript）建立專案：

```shell
npx react-native init AwesomeTSProject --template react-native-template-typescript
```

<h2>Preparing the Android device</h2>

您需要一台 Android 裝置來執行 React Native Android 應用程式。這可以是實體 Android 裝置，或更常見的是使用 Android 虛擬裝置（AVD），讓您在電腦上模擬 Android 裝置。

無論哪種方式，您都需要準備裝置以執行開發用的 Android 應用程式。

<h3>Using a physical device</h3>

若有實體 Android 裝置，可透過 USB 線連接電腦後，依照[此處說明](running-on-device.md)將其用於開發，以取代 AVD。

<h3>Using a virtual device</h3>

若使用 Android Studio 開啟 `./AwesomeProject/android`，可透過 Android Studio 內的「AVD Manager」查看可用 Android 虛擬裝置（AVD）清單。尋找如下圖示的圖標：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

若您最近才安裝 Android Studio，可能需要[建立新的 AVD](https://developer.android.com/studio/run/managing-avds.html)。點選「Create Virtual Device...」，從清單中選擇任一手機型號後點擊「Next」，接著選擇 **S** API Level 31 的系統映像。

> 建議為系統配置[虛擬機加速](https://developer.android.com/studio/run/emulator-acceleration.html#vm-linux)以提升效能。完成設定後，請返回 AVD 管理員介面。

點擊「Next」後選擇「Finish」即可建立 AVD。此時您應能點擊 AVD 旁的綠色三角形按鈕啟動模擬器，並繼續後續步驟。

<h2>Running your React Native application</h2>

<h3>Step 1: Start Metro</h3>

First, you will need to start Metro, the JavaScript bundler that ships with React Native. Metro "takes in an entry file and various options, and returns a single JavaScript file that includes all your code and its dependencies."—[Metro Docs](https://metrobundler.dev/docs/concepts)

To start Metro, run `npx react-native start` inside your React Native project folder:

```shell
npx react-native start
```

`react-native start` 指令會啟動 Metro 打包工具。

> 若使用 Yarn 套件管理器，在既有專案中執行 React Native 指令時可用 `yarn` 替代 `npx`。

> 對網頁開發者而言，Metro 類似於 React Native 的 webpack。需注意 JavaScript 不同於 Kotlin 或 Java，它不需編譯——React Native 也是。打包雖不等同於編譯，但能改善啟動效能，並將某些平台特定的 JavaScript 轉換為相容性更高的程式碼。

<h3>Step 2: Start your application</h3>

讓 Metro 打包工具在獨立終端機中運行。另開新終端機進入專案目錄，執行：

```shell
npx react-native run-android
```

若所有設定正確，您將很快在 Android 模擬器中看到新應用程式運行。

`npx react-native run-android` 只是執行方式之一，您亦可直接透過 Android Studio 啟動應用。

> 若遇到問題，請參閱[疑難排解指南](troubleshooting.md)。

<h3>Modifying your app</h3>

成功運行應用後，現在開始修改它：

- Open `App.js` in your text editor of choice and edit some lines.
- Press the `R` key twice or select `Reload` from the Developer Menu (`Ctrl + M`) to see your changes!

<h3>That's it!</h3>

恭喜！您已成功運行並修改了第一個 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- If you want to add this new React Native code to an existing application, check out the [Integration guide](integration-with-existing-apps.md).

If you're curious to learn more about React Native, check out the [Introduction to React Native](getting-started).