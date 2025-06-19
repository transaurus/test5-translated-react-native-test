import RemoveGlobalCLI from './\_remove-global-cli.md';
import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 安裝依賴套件

您需要安裝 Node、React Native 命令行界面、JDK 和 Android Studio。

雖然您可以使用任何編輯器開發應用程式，但必須安裝 Android Studio 才能設置建置 React Native Android 應用所需的工具鏈。

<h3>Node</h3>

請根據[Linux 發行版的安裝指南](https://nodejs.org/en/download/package-manager/)安裝 Node 18 或更新版本。

<h3>Java Development Kit</h3>

React Native currently recommends version 17 of the Java SE Development Kit (JDK). You may encounter problems using higher JDK versions. You may download and install [OpenJDK](https://openjdk.java.net) from [AdoptOpenJDK](https://adoptopenjdk.net/) or your system packager.

<h3>Android development environment</h3>

若您是 Android 開發新手，設置開發環境可能較為繁瑣。若已有相關經驗，則需進行部分配置。無論如何，請仔細遵循後續步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在安裝精靈中，請確保勾選以下所有項目：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`

接著點擊「Next」安裝這些元件。

> 若複選框呈灰色，後續仍可安裝這些元件。

完成安裝並顯示歡迎畫面後，請繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新版 Android SDK。但使用原生代碼建置 React Native 應用時，需特定安裝 `Android 14 (UpsideDownCake)` SDK。可透過 Android Studio 的 SDK 管理器安裝其他版本。

開啟 Android Studio 後，點擊「Configure」按鈕並選擇「SDK Manager」。

> SDK 管理器也可在 Android Studio 的「Settings」對話框中找到，路徑為 **Languages & Frameworks** → **Android SDK**。

在 SDK 管理器內選擇「SDK Platforms」標籤頁，勾選右下角的「Show Package Details」。找到並展開 `Android 14 (UpsideDownCake)` 項目，確保勾選以下內容：

- `Android SDK Platform 34`
- `Intel x86 Atom_64 System Image` 或 `Google APIs Intel x86 Atom System Image`

接著切換至「SDK Tools」標籤頁，同樣勾選「Show Package Details」。找到並展開「Android SDK Build-Tools」項目，確保選中 `34.0.0` 版本。

最後點擊「Apply」下載並安裝 Android SDK 及相關建置工具。

<h4>3. Configure the ANDROID_HOME environment variable</h4>

React Native 工具鏈需要設置特定環境變數才能建置含原生代碼的應用。

將以下內容加入您的 `$HOME/.bash_profile` 或 `$HOME/.bashrc` 配置檔案（若使用 `zsh` 則為 `~/.zprofile` 或 `~/.zshrc`）：

```shell
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

> `.bash_profile` 專屬於 `bash`。若使用其他 shell，需編輯對應的 shell 配置檔案。

Type `source $HOME/.bash_profile` for `bash` or `source $HOME/.zprofile` to load the config into your current shell. Verify that ANDROID_HOME has been set by running `echo $ANDROID_HOME` and the appropriate directories have been added to your path by running `echo $PATH`.

> 請確保使用正確的 Android SDK 路徑。您可以在 Android Studio 的「設定」對話框中找到 SDK 的實際位置，路徑為 **Languages & Frameworks** → **Android SDK**。

<h3>Watchman</h3>

請遵循 [Watchman 安裝指南](https://facebook.github.io/watchman/docs/install#buildinstall) 從原始碼編譯並安裝 Watchman。

> [Watchman](https://facebook.github.io/watchman/docs/install) 是 Facebook 開發的檔案系統變更監控工具。強烈建議安裝以提升效能並增強特定邊緣案例的相容性（譯註：不安裝可能也能運作，但效果因人而異；現在安裝可避免後續潛在問題）。

<h3>React Native Command Line Interface</h3>

React Native 內建命令列介面。與其全域安裝並管理特定版本的 CLI，我們建議透過 Node.js 內建的 `npx` 在執行時存取當前版本。使用 `npx react-native <command>` 時，系統會自動下載並執行最新穩定版的 CLI。

<h2>Creating a new application</h2>

<RemoveGlobalCLI />

React Native 內建命令列介面，可用於生成新專案。您無需全域安裝任何套件，直接使用 Node.js 內建的 `npx` 即可存取。以下示範建立名為「AwesomeProject」的新 React Native 專案：

```shell
npx react-native@latest init AwesomeProject
```

若您要將 React Native 整合至現有應用程式、專案中已安裝 [Expo](https://docs.expo.dev/bare/installing-expo-modules/)，或為現有 React Native 專案新增 Android 支援（參見 [與現有應用程式整合](integration-with-existing-apps.md)），則無需此步驟。您也可使用第三方 CLI（如 [Ignite CLI](https://github.com/infinitered/ignite)）初始化 React Native 應用程式。

<h3>[Optional] Using a specific version or template</h3>

若要使用特定 React Native 版本建立新專案，可加入 `--version` 參數：

```shell
npx react-native@X.XX.X init AwesomeProject --version X.XX.X
```

您也可透過 `--template` 參數使用自訂的 React Native 模板建立專案。

<h2>Preparing the Android device</h2>

執行 React Native Android 應用程式需要 Android 裝置，可以是實體裝置，或更常見的 Android 虛擬裝置（AVD），後者能在電腦上模擬 Android 裝置。

無論採用何種方式，均需準備裝置以執行開發用的 Android 應用程式。

<h3>Using a physical device</h3>

若有實體 Android 裝置，可透過 USB 連接電腦後，依照[此處說明](running-on-device.md)設定為開發環境，無需使用 AVD。

<h3>Using a virtual device</h3>

若使用 Android Studio 開啟 `./AwesomeProject/android`，可透過 Android Studio 內的「AVD Manager」查看可用虛擬裝置清單。尋找如下圖示：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

若您剛安裝 Android Studio，可能需要[建立新 AVD](https://developer.android.com/studio/run/managing-avds.html)。選擇「Create Virtual Device...」，從清單中任選手機型號後點擊「Next」，接著選擇 **UpsideDownCake** API Level 34 映像檔。

> 我們建議在您的系統上配置 [VM 加速](https://developer.android.com/studio/run/emulator-acceleration.html#vm-linux) 以提升效能。完成這些指示後，請返回 AVD 管理員。

點擊「下一步」然後「完成」以創建您的 AVD。此時您應該可以點擊 AVD 旁邊的綠色三角形按鈕來啟動它，接著進行下一步。

<h2>Running your React Native application</h2>

<h3>Step 1: Start Metro</h3>

[**Metro**](https://facebook.github.io/metro/) 是 React Native 的 JavaScript 建置工具。要啟動 Metro 開發伺服器，請在您的專案資料夾中執行以下命令：

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
如果您熟悉網頁開發，Metro 類似於 Vite 和 webpack 等打包工具，但專為 React Native 端到端設計。例如，Metro 使用 [Babel](https://babel.dev/) 將 JSX 等語法轉換為可執行的 JavaScript。
:::

<h3>Step 2: Start your application</h3>

讓 Metro Bundler 在它自己的終端機中運行。在您的 React Native 專案資料夾中開啟一個新的終端機。執行以下命令：

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

如果一切設置正確，您應該很快就能看到您的新應用程式在 Android 模擬器中運行。

這是運行應用程式的一種方式——您也可以直接從 Android Studio 內部運行它。

> 如果無法正常運作，請參閱 [疑難排解](troubleshooting.md) 頁面。

<h3>Modifying your app</h3>

現在您已成功運行應用程式，讓我們來修改它。

- Open `App.tsx` in your text editor of choice and edit some lines.
- Press the <kbd>R</kbd> key twice or select `Reload` from the Dev Menu (<kbd>Ctrl</kbd> + <kbd>M</kbd>) to see your changes!

<h3>That's it!</h3>

恭喜！您已成功運行並修改了您的第一個 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- 如果您想將這段新的 React Native 代碼添加到現有應用程式中，請查看 [整合指南](integration-with-existing-apps.md)。

如果您想了解更多關於 React Native 的資訊，請查看 [React Native 簡介](getting-started)。