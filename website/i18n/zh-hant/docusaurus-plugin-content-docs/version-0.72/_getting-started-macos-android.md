import RemoveGlobalCLI from './\_remove-global-cli.md';
import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 安裝依賴套件

您需要安裝 Node、Watchman、React Native 命令行界面、JDK 和 Android Studio。

雖然您可以使用任何喜歡的編輯器開發應用程式，但必須安裝 Android Studio 才能設置建置 React Native Android 應用程式所需的工具。

<h3>Node &amp; Watchman</h3>

建議使用 [Homebrew](http://brew.sh/) 安裝 Node 和 Watchman。安裝 Homebrew 後，在終端機執行以下指令：

```shell
brew install node
brew install watchman
```

若系統已安裝 Node，請確認版本為 16 或更新。

[Watchman](https://facebook.github.io/watchman) 是 Facebook 開發的檔案系統監控工具。強烈建議安裝以提升效能。

<h3>Java Development Kit</h3>

We recommend installing the OpenJDK distribution called Azul **Zulu** using [Homebrew](http://brew.sh/). Run the following commands in a Terminal after installing Homebrew:

```shell
brew install --cask zulu@11

# Get path to where cask was installed to double-click installer
brew info --cask zulu@11
```

安裝 JDK 後，請更新 `JAVA_HOME` 環境變數。若依上述步驟安裝，JDK 路徑通常為 `/Library/Java/JavaVirtualMachines/zulu-11.jdk/Contents/Home`

Zulu OpenJDK 發行版提供 **Intel 和 M1 晶片 Mac 專用**的 JDK。這能確保 M1 Mac 的建置速度比使用 Intel 版 JDK 更快。

若系統已安裝 JDK，建議使用 JDK 11 版本。使用更高版本可能會遇到問題。

<h3>Android development environment</h3>

若您是 Android 開發新手，設置開發環境可能較為繁瑣。若已有經驗，則只需調整部分設定。無論如何，請務必仔細遵循後續步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在安裝精靈中，請勾選以下所有項目：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`

接著點擊「Next」安裝這些元件。

> 若選項呈灰色不可選，後續仍可安裝這些元件。

安裝完成並顯示歡迎畫面後，請繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新版 Android SDK。但使用原生程式碼建置 React Native 應用程式需要特定的 `Android 13 (Tiramisu)` SDK。可透過 Android Studio 的 SDK 管理器安裝其他版本。

開啟 Android Studio 後，點擊「More Actions」按鈕並選擇「SDK Manager」。

![Android Studio 歡迎畫面](/docs/assets/GettingStartedAndroidStudioWelcomeMacOS.png)

> 也可在 Android Studio 的「Settings」對話框中找到 SDK 管理器，路徑為 **Languages & Frameworks** → **Android SDK**。

在 SDK 管理器中選擇「SDK Platforms」標籤，勾選右下角的「Show Package Details」。找到並展開 `Android 13 (Tiramisu)` 項目，確認以下內容已勾選：

- `Android SDK Platform 33`
- `Intel x86 Atom_64 System Image` or `Google APIs Intel x86 Atom System Image` or (for Apple M1 Silicon) `Google APIs ARM 64 v8a System Image`

接著，選擇「SDK Tools」標籤頁，同樣勾選「顯示套件詳細資訊」。找到並展開「Android SDK Build-Tools」項目，確保選中 `33.0.0` 版本。

最後，點擊「Apply」以下載並安裝 Android SDK 及相關建置工具。

<h4>3. Configure the ANDROID_HOME environment variable</h4>

React Native 工具需要設定某些環境變數才能建置包含原生代碼的應用程式。

將以下內容加入你的 `~/.zprofile` 或 `~/.zshrc` 設定檔（若使用 `bash`，則為 `~/.bash_profile` 或 `~/.bashrc`）：

```shell
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

執行 `source ~/.zprofile`（或 `source ~/.bash_profile` 適用於 `bash`）以載入設定至當前 shell。透過執行 `echo $ANDROID_HOME` 確認 ANDROID_HOME 已設定，並執行 `echo $PATH` 檢查正確目錄是否已加入路徑。

> 請確保使用正確的 Android SDK 路徑。你可以在 Android Studio 的「Settings」對話框中找到 SDK 的實際位置，路徑為 **Languages & Frameworks** → **Android SDK**。

<h3>React Native Command Line Interface</h3>

React Native 內建命令列介面。與其全域安裝並管理特定版本的 CLI，我們建議透過 Node.js 自帶的 `npx` 在執行時存取當前版本。使用 `npx react-native <command>` 時，將自動下載並執行當前穩定版的 CLI。

<h2>Creating a new application</h2>

<RemoveGlobalCLI />

React Native 內建命令列介面，可用於生成新專案。無需全域安裝任何套件，透過 Node.js 自帶的 `npx` 即可存取。以下建立一個名為「AwesomeProject」的新 React Native 專案：

```shell
npx react-native@latest init AwesomeProject
```

若你正在將 React Native 整合至現有應用程式、專案中已安裝 [Expo](https://docs.expo.dev/bare/installing-expo-modules/)，或為現有 React Native 專案添加 Android 支援（參見 [與現有應用程式整合](integration-with-existing-apps.md)），則無需此步驟。你也可以使用第三方 CLI（如 [Ignite CLI](https://github.com/infinitered/ignite)）初始化 React Native 應用程式。

<h3>[Optional] Using a specific version or template</h3>

若要使用特定 React Native 版本建立新專案，可加入 `--version` 參數：

```shell
npx react-native@X.XX.X init AwesomeProject --version X.XX.X
```

你也可以透過 `--template` 參數使用自訂 React Native 模板建立專案。

<h2>Preparing the Android device</h2>

執行 React Native Android 應用程式需要 Android 裝置，可以是實體裝置，或更常見的做法是使用 Android 虛擬裝置（AVD）在電腦上模擬 Android 裝置。

無論採用何種方式，均需準備裝置以執行開發用的 Android 應用程式。

<h3>Using a physical device</h3>

若有實體 Android 裝置，可透過 USB 連接電腦並遵循[此處說明](running-on-device.md)，即可用於開發以取代 AVD。

<h3>Using a virtual device</h3>

若使用 Android Studio 開啟 `./AwesomeProject/android` 專案，可透過點選 Android Studio 內的「AVD Manager」圖示（外觀如下）查看可用的 Android 虛擬裝置清單：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

若剛安裝 Android Studio，需先[建立新 AVD](https://developer.android.com/studio/run/managing-avds.html)。點選「Create Virtual Device...」後選擇任一手機型號，接著點擊「Next」，並選擇 **Tiramisu** API Level 33 系統映像。

點擊「Next」後按「Finish」完成 AVD 建立。此時可點擊 AVD 旁的綠色三角按鈕啟動模擬器，再繼續後續步驟。

<h2>Running your React Native application</h2>

<h3>Step 1: Start Metro</h3>

First, you will need to start Metro, the JavaScript bundler that ships with React Native. Metro "takes in an entry file and various options, and returns a single JavaScript file that includes all your code and its dependencies."—[Metro Docs](https://metrobundler.dev/docs/concepts)

在 React Native 專案目錄下執行以下指令啟動 Metro：

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

> 若熟悉網頁開發，Metro 之於 React Native 類似 webpack 之於網頁應用。與 Kotlin 或 Java 不同，JavaScript 不需編譯——React Native 亦然。打包（bundling）雖不等同編譯，但能提升啟動效能並將部分平台特定的 JavaScript 轉換為相容性更高的程式碼。

<h3>Step 2: Start your application</h3>

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run android
```

</TabItem>
<TabItem value="yarn">

```shell
yarn android
```

</TabItem>
</Tabs>

若所有設定正確，稍後應能在 Android 模擬器中看見新應用程式運行。

![Android 上的 AwesomeProject](/docs/assets/GettingStartedAndroidSuccessMacOS.png)

此為執行方式之一，亦可直接透過 Android Studio 運行應用程式。

> 若遇到問題，請參閱[疑難排解](troubleshooting.md)頁面。

<h3>Modifying your app</h3>

成功運行應用程式後，現在開始修改它。

- Open `App.tsx` in your text editor of choice and edit some lines.
- Press the <kbd>R</kbd> key twice or select `Reload` from the Dev Menu (<kbd>Cmd ⌘</kbd> + <kbd>M</kbd>) to see your changes!

<h3>That's it!</h3>

恭喜！您已成功運行並修改第一個 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- If you want to add this new React Native code to an existing application, check out the [Integration guide](integration-with-existing-apps.md).

想深入瞭解 React Native？請查看 [React Native 簡介](getting-started)。