import RemoveGlobalCLI from './\_remove-global-cli.md';
import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 安裝依賴套件

您需要安裝 Node、Watchman、React Native 命令行工具、JDK 以及 Android Studio。

雖然您可以使用任何喜歡的編輯器開發應用程式，但必須安裝 Android Studio 才能設置建置 React Native Android 應用所需的工具鏈。

<h3>Node &amp; Watchman</h3>

建議使用 [Homebrew](https://brew.sh/) 安裝 Node 和 Watchman。安裝 Homebrew 後，在終端機執行以下命令：

```shell
brew install node
brew install watchman
```

若系統已安裝 Node，請確認版本為 18 或更新。

[Watchman](https://facebook.github.io/watchman) 是 Facebook 開發的檔案系統監控工具。強烈建議安裝以提升效能。

<h3>Java Development Kit</h3>

We recommend installing the OpenJDK distribution called Azul **Zulu** using [Homebrew](https://brew.sh/). Run the following commands in a Terminal after installing Homebrew:

```shell
brew install --cask zulu@17

# Get path to where cask was installed to double-click installer
brew info --cask zulu@17
```

安裝 JDK 後，請更新 `JAVA_HOME` 環境變數。若依上述步驟安裝，JDK 路徑通常為 `/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home`

Zulu OpenJDK 發行版提供 **Intel 和 M1 晶片 Mac 專用** 的 JDK。這能確保在 M1 Mac 上的建置速度比使用 Intel 版 JDK 更快。

若系統已安裝 JDK，建議使用 JDK 17 版本。使用更高版本可能會遇到問題。

<h3>Android development environment</h3>

若您是 Android 開發新手，設置開發環境可能較為繁瑣。若已有經驗，則只需調整部分設定。無論如何，請仔細遵循後續步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在安裝精靈中，請勾選以下所有項目：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`

接著點擊「Next」安裝這些元件。

> 若選項呈灰色不可選，後續仍可安裝這些元件。

安裝完成並顯示歡迎畫面後，繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新版 Android SDK。但使用原生碼建置 React Native 應用需特定安裝 `Android 14 (UpsideDownCake)` SDK。可透過 Android Studio 的 SDK 管理員安裝其他版本。

開啟 Android Studio，點擊「More Actions」按鈕並選擇「SDK Manager」。

![Android Studio 歡迎畫面](/docs/assets/GettingStartedAndroidStudioWelcomeMacOS.png)

> 也可在 Android Studio 的「Settings」對話框中找到 SDK Manager，路徑為 **Languages & Frameworks** → **Android SDK**。

在 SDK Manager 中選擇「SDK Platforms」標籤頁，勾選右下角的「Show Package Details」。找到並展開 `Android 14 (UpsideDownCake)` 項目，確認以下元件已勾選：

- `Android SDK Platform 34`
- `Intel x86 Atom_64 System Image` or `Google APIs Intel x86 Atom System Image` or (for Apple M1 Silicon) `Google APIs ARM 64 v8a System Image`

接著，選擇「SDK Tools」標籤，同樣勾選右下角的「顯示套件詳細資訊」。找到並展開「Android SDK Build-Tools」項目，確保已選取 `34.0.0` 版本。

最後點擊「Apply」按鈕以下載並安裝 Android SDK 及相關建置工具。

<h4>3. Configure the ANDROID_HOME environment variable</h4>

React Native 工具需要設定某些環境變數才能建置包含原生代碼的應用程式。

請將以下內容加入您的 `~/.zprofile` 或 `~/.zshrc` 設定檔（若使用 `bash` 則為 `~/.bash_profile` 或 `~/.bashrc`）：

```shell
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

執行 `source ~/.zprofile`（或 `source ~/.bash_profile` 適用於 `bash`）以載入設定至當前 shell。透過執行 `echo $ANDROID_HOME` 確認 ANDROID_HOME 已設定，並執行 `echo $PATH` 檢查正確目錄是否已加入路徑。

> Please make sure you use the correct Android SDK path. You can find the actual location of the SDK in the Android Studio "Settings" dialog, under **Languages & Frameworks** → **Android SDK**.

<h3>React Native Command Line Interface</h3>

React Native 內建命令列介面。與其全域安裝並管理特定版本的 CLI，我們建議透過 Node.js 自帶的 `npx` 在執行時存取當前版本。使用 `npx react-native <command>` 時，系統會自動下載並執行最新穩定版的 CLI。

<h2>Creating a new application</h2>

<RemoveGlobalCLI />

React Native 內建命令列介面可用於生成新專案。您無需全域安裝任何套件，直接使用 Node.js 自帶的 `npx` 即可存取。以下示範建立名為「AwesomeProject」的新專案：

```shell
npx react-native@latest init AwesomeProject
```

若您要將 React Native 整合至現有應用程式、專案中已安裝 [Expo](https://docs.expo.dev/bare/installing-expo-modules/)，或是為現有 React Native 專案添加 Android 支援（參見 [與現有應用程式整合](integration-with-existing-apps.md)），則無需此步驟。您也可使用第三方 CLI（如 [Ignite CLI](https://github.com/infinitered/ignite)）初始化專案。

<h3>[Optional] Using a specific version or template</h3>

若要使用特定 React Native 版本建立專案，可加入 `--version` 參數：

```shell
npx react-native@X.XX.X init AwesomeProject --version X.XX.X
```

您也可透過 `--template` 參數使用自訂的 React Native 模板建立專案。

<h2>Preparing the Android device</h2>

執行 React Native Android 應用程式需要 Android 裝置，可以是實體裝置或更常見的 Android 虛擬裝置（AVD），後者能在電腦上模擬 Android 裝置。

無論採用何種方式，均需準備裝置以執行開發版 Android 應用程式。

<h3>Using a physical device</h3>

若有實體 Android 裝置，可透過 USB 連接電腦後，依照[此處說明](running-on-device.md)設定以取代 AVD 進行開發。

<h3>Using a virtual device</h3>

如果你使用 Android Studio 開啟 `./AwesomeProject/android` 專案，可以透過 Android Studio 內的「AVD 管理器」查看可用的 Android 虛擬裝置 (AVD) 清單。尋找如下圖所示的圖示：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

若你剛安裝 Android Studio，可能需要[建立新的 AVD](https://developer.android.com/studio/run/managing-avds.html)。點選「Create Virtual Device...」，從清單中選擇任一手機型號後點擊「Next」，接著選擇 **UpsideDownCake** API Level 34 系統映像。

點擊「Next」後按「Finish」建立 AVD。此時你應能點擊 AVD 旁的綠色三角形按鈕啟動模擬器，並繼續下一步。

<h2>Running your React Native application</h2>

<h3>Step 1: Start Metro</h3>

[**Metro**](https://facebook.github.io/metro/) 是 React Native 的 JavaScript 建置工具。要啟動 Metro 開發伺服器，請在專案目錄下執行：

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

:::note
若熟悉網頁開發，Metro 類似 Vite 或 webpack 等打包工具，但專為 React Native 設計。例如 Metro 使用 [Babel](https://babel.dev/) 將 JSX 等語法轉換為可執行的 JavaScript。
:::

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

若所有設定正確，你應該很快會在 Android 模擬器中看到新應用程式運行。

![Android 上的 AwesomeProject](/docs/assets/GettingStartedAndroidSuccessMacOS.png)

這是執行應用程式的方式之一，你也可以直接透過 Android Studio 運行。

> 若無法正常運作，請參閱[疑難排解](troubleshooting.md)頁面。

<h3>Modifying your app</h3>

現在你已成功運行應用程式，讓我們來修改它。

- Open `App.tsx` in your text editor of choice and edit some lines.
- Press the <kbd>R</kbd> key twice or select `Reload` from the Dev Menu (<kbd>Cmd ⌘</kbd> + <kbd>M</kbd>) to see your changes!

<h3>That's it!</h3>

恭喜！你已成功運行並修改了第一個 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- 若要將此 React Native 程式碼整合至現有應用程式，請參閱[整合指南](integration-with-existing-apps.md)。

若想深入學習 React Native，請查看 [React Native 簡介](getting-started)。