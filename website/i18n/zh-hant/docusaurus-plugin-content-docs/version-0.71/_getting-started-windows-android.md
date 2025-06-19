import RemoveGlobalCLI from './\_remove-global-cli.md';

<h2>Installing dependencies</h2>

您需要安裝 Node、React Native 命令行工具、JDK 和 Android Studio。

雖然您可以使用任何編輯器開發應用程式，但必須安裝 Android Studio 才能設置必要的工具來為 Android 構建 React Native 應用程式。

<h3 id="jdk">Node, JDK</h3>

我們建議透過 [Chocolatey](https://chocolatey.org)（Windows 上流行的套件管理工具）安裝 Node。

建議使用 Node 的 LTS 版本。若需切換不同版本，可透過 [nvm-windows](https://github.com/coreybutler/nvm-windows)（Windows 的 Node 版本管理工具）安裝 Node。

React Native 還需要 [Java SE Development Kit (JDK)](https://openjdk.java.net/projects/jdk/11/)，同樣可透過 Chocolatey 安裝。

以管理員身分開啟命令提示字元（右鍵點擊命令提示字元並選擇「以系統管理員身分執行」），然後執行以下命令：

```powershell
choco install -y nodejs-lts microsoft-openjdk11
```

若系統已安裝 Node，請確認版本為 14 或更新。若已安裝 JDK，建議使用 JDK11。使用更高版本 JDK 可能會遇到問題。

> 更多安裝選項請參閱 [Node 下載頁面](https://nodejs.org/en/download/)。

> 若使用最新版 Java Development Kit，需變更專案的 Gradle 版本以識別 JDK。方法是前往 `{project root folder}\android\gradle\wrapper\gradle-wrapper.properties` 並修改 `distributionUrl` 值來升級 Gradle 版本。[此處可查閱 Gradle 最新版本](https://gradle.org/releases/)。

<h3>Android development environment</h3>

若您是 Android 開發新手，設置開發環境可能較為繁瑣。即使已有經驗，仍需配置部分設定。無論如何，請仔細遵循後續步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在安裝精靈中，請確保勾選以下所有項目：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`
- 若未使用 Hyper-V：`Performance (Intel ® HAXM)`（[AMD 或 Hyper-V 用戶請參閱此處](https://android-developers.googleblog.com/2018/07/android-emulator-amd-processor-hyper-v.html)）

接著點擊「Next」安裝所有元件。

> 若選項呈灰色不可勾選，後續仍可安裝這些元件。

安裝完成並顯示歡迎畫面後，請繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新版 Android SDK。但使用原生代碼構建 React Native 應用程式需要特定的 `Android 13 (Tiramisu)` SDK。可透過 Android Studio 的 SDK 管理器安裝其他 Android SDK。

開啟 Android Studio，點擊「More Actions」按鈕並選擇「SDK Manager」。

![Android Studio 歡迎畫面](/docs/assets/GettingStartedAndroidStudioWelcomeWindows.png)

> SDK 管理器也可在 Android Studio 的「Settings」對話框中找到，路徑為 **Languages & Frameworks** → **Android SDK**。

在 SDK 管理器中選擇「SDK Platforms」標籤頁，然後勾選右下角的「Show Package Details」。找到並展開 `Android 13 (Tiramisu)` 項目，確保以下項目已被勾選：

- `Android SDK Platform 33`
- `Intel x86 Atom_64 System Image` 或 `Google APIs Intel x86 Atom System Image`

接著選擇「SDK Tools」標籤頁，同樣勾選「Show Package Details」。找到並展開 `Android SDK Build-Tools` 項目，確保選中 `33.0.0` 版本。

最後點擊「Apply」以下載並安裝 Android SDK 及相關建置工具。

<h4>3. Configure the ANDROID_HOME environment variable</h4>

React Native 工具需要設置某些環境變數才能建置包含原生代碼的應用程式。

1. Open the **Windows Control Panel.**
2. Click on **User Accounts,** then click **User Accounts** again
3. Click on **Change my environment variables**
4. Click on **New...** to create a new `ANDROID_HOME` user variable that points to the path to your Android SDK:

![ANDROID_HOME 環境變數](/docs/assets/GettingStartedAndroidEnvironmentVariableANDROID_HOME.png)

SDK 預設安裝在以下位置：

```powershell
%LOCALAPPDATA%\Android\Sdk
```

您可以在 Android Studio 的「Settings」對話框中找到 SDK 的實際位置，路徑為 **Languages & Frameworks** → **Android SDK**。

開啟新的命令提示字元視窗以確保新環境變數已載入，再繼續下一步。

1. 開啟 powershell
2. 複製並貼上 **Get-ChildItem -Path Env:\\** 到 powershell
3. 確認 `ANDROID_HOME` 已添加

<h4>4. Add platform-tools to Path</h4>

1. Open the **Windows Control Panel.**
2. Click on **User Accounts,** then click **User Accounts** again
3. Click on **Change my environment variables**
4. Select the **Path** variable.
5. Click **Edit.**
6. Click **New** and add the path to platform-tools to the list.

此資料夾的預設位置為：

```powershell
%LOCALAPPDATA%\Android\Sdk\platform-tools
```

<h3>React Native Command Line Interface</h3>

React Native 內建命令列介面。與其全域安裝並管理特定版本的 CLI，我們建議您使用 Node.js 自帶的 `npx` 在運行時存取當前版本。透過 `npx react-native <command>` 執行命令時，將自動下載並運行當前穩定版的 CLI。

<h2>Creating a new application</h2>

<RemoveGlobalCLI />

React Native 內建命令列介面可用於生成新專案。您無需全域安裝任何套件，直接使用 Node.js 自帶的 `npx` 即可存取。讓我們創建一個名為「AwesomeProject」的新 React Native 專案：

```shell
npx react-native@latest init AwesomeProject
```

若您要將 React Native 整合至現有應用程式、專案中已安裝 [Expo](https://docs.expo.dev/bare/installing-expo-modules/)，或是為現有 React Native 專案添加 Android 支援（參見 [與現有應用程式整合](integration-with-existing-apps.md)），則無需此步驟。您也可以使用第三方 CLI（如 [Ignite CLI](https://github.com/infinitered/ignite)）來初始化 React Native 應用程式。

<h3>[Optional] Using a specific version or template</h3>

若想使用特定 React Native 版本創建新專案，可透過 `--version` 參數指定：

```shell
npx react-native@X.XX.X init AwesomeProject --version X.XX.X
```

你也可以使用 `--template` 參數來以自訂的 React Native 模板啟動專案。

<h2>Preparing the Android device</h2>

你需要一個 Android 裝置來運行你的 React Native Android 應用程式。這可以是實體的 Android 裝置，或者更常見的是使用 Android 虛擬裝置（AVD），它允許你在電腦上模擬 Android 裝置。

無論哪種方式，你都需要準備裝置以運行開發用的 Android 應用程式。

<h3>Using a physical device</h3>

如果你有實體的 Android 裝置，你可以透過 USB 線將其連接到電腦，並按照[這裡](running-on-device.md)的指示來代替 AVD 進行開發。

<h3>Using a virtual device</h3>

如果你使用 Android Studio 打開 `./AwesomeProject/android`，你可以透過 Android Studio 內的「AVD Manager」查看可用的 Android 虛擬裝置（AVDs）列表。尋找一個看起來像這樣的圖標：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

如果你最近安裝了 Android Studio，你可能需要[創建一個新的 AVD](https://developer.android.com/studio/run/managing-avds.html)。選擇「Create Virtual Device...」，然後從列表中選擇任何一款手機並點擊「Next」，接著選擇 **Tiramisu** API Level 33 的映像檔。

> 如果你沒有安裝 HAXM，點擊「Install HAXM」或按照[這些指示](https://github.com/intel/haxm/wiki/Installation-Instructions-on-Windows)來設置，然後回到 AVD Manager。

點擊「Next」然後「Finish」來創建你的 AVD。此時，你應該可以點擊 AVD 旁邊的綠色三角形按鈕來啟動它，然後繼續下一步。

<h2>Running your React Native application</h2>

<h3>Step 1: Start Metro</h3>

首先，你需要啟動 Metro，這是 React Native 內建的 JavaScript 打包工具。Metro「接收一個入口檔案和各種選項，並返回一個包含所有代碼及其依賴項的單一 JavaScript 檔案。」—[Metro 文檔](https://metrobundler.dev/docs/concepts)

要啟動 Metro，請在你的 React Native 專案資料夾內運行 `npx react-native start`：

```shell
npx react-native start
```

`react-native start` 會啟動 Metro Bundler。

> 如果你使用 Yarn 套件管理器，你可以在現有專案中運行 React Native 命令時使用 `yarn` 代替 `npx`。

> 如果你熟悉網頁開發，Metro 很像 webpack——但用於 React Native 應用程式。與 Kotlin 或 Java 不同，JavaScript 不是編譯的——React Native 也不是。打包（bundling）與編譯不同，但它可以幫助提高啟動性能，並將一些平台特定的 JavaScript 轉換為更廣泛支持的 JavaScript。

<h3>Step 2: Start your application</h3>

讓 Metro Bundler 在自己的終端中運行。在你的 React Native 專案資料夾內打開一個新的終端，然後運行以下命令：

```shell
npx react-native run-android
```

如果一切設置正確，你應該很快就能在你的 Android 模擬器中看到新運行的應用程式。

![AwesomeProject on Android](/docs/assets/GettingStartedAndroidSuccessWindows.png)

`npx react-native run-android` 是運行應用程式的一種方式——你也可以直接從 Android Studio 中運行它。

> 如果無法運行，請參閱[疑難排解](troubleshooting.md)頁面。

<h3>Modifying your app</h3>

現在您已成功運行應用程式，接下來讓我們進行修改。

- Open `App.tsx` in your text editor of choice and edit some lines.
- Press the `R` key twice or select `Reload` from the Developer Menu (`Ctrl + M`) to see your changes!

<h3>That's it!</h3>

恭喜！您已成功運行並修改了第一個 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- 如果想將這段新的 React Native 程式碼加入現有應用程式，請參閱 [整合指南](integration-with-existing-apps.md)。

若想進一步了解 React Native，請查看 [React Native 簡介](getting-started)。