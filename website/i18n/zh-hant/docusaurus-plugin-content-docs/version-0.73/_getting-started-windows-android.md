import RemoveGlobalCLI from './\_remove-global-cli.md';
import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<h2>Installing dependencies</h2>

您需要安裝 Node、React Native 命令行界面、JDK 和 Android Studio。

雖然您可以使用任何喜歡的編輯器開發應用程式，但必須安裝 Android Studio 才能設置必要的工具來為 Android 構建 React Native 應用程式。

<h3 id="jdk">Node, JDK</h3>

我們建議透過 [Chocolatey](https://chocolatey.org/install)（Windows 上流行的套件管理工具）安裝 Node。

建議使用 Node 的 LTS 版本。若需切換不同版本，可透過 [nvm-windows](https://github.com/coreybutler/nvm-windows)（Windows 的 Node 版本管理工具）安裝 Node。

React Native 還需要 [Java SE 開發套件 (JDK)](https://openjdk.java.net/projects/jdk/17/)，同樣可透過 Chocolatey 安裝。

以管理員身分開啟命令提示字元（右鍵點擊命令提示字元並選擇「以系統管理員身分執行」），然後執行以下命令：

```powershell
choco install -y nodejs-lts microsoft-openjdk17
```

若系統已安裝 Node，請確認版本為 18 或更新。若已安裝 JDK，建議使用 JDK17。使用更高版本的 JDK 可能會遇到問題。

> 您可以在 [Node 下載頁面](https://nodejs.org/en/download/) 找到其他安裝選項。

> 若使用最新版 Java 開發套件，需變更專案的 Gradle 版本以識別 JDK。方法是前往 `{project root folder}\android\gradle\wrapper\gradle-wrapper.properties` 並修改 `distributionUrl` 值來升級 Gradle 版本。可參考 [Gradle 最新發布版本](https://gradle.org/releases/)。

<h3>Android development environment</h3>

若您是 Android 開發新手，設置開發環境可能較為繁瑣。若已熟悉 Android 開發，則只需配置部分項目。無論哪種情況，請仔細遵循接下來的步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在安裝精靈中，請確保勾選以下所有項目：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`
- 若未使用 Hyper-V：`Performance (Intel ® HAXM)`（[AMD 或 Hyper-V 用戶請參閱此處](https://android-developers.googleblog.com/2018/07/android-emulator-amd-processor-hyper-v.html)）

接著點擊「Next」安裝所有元件。

> 若複選框呈灰色，後續仍可安裝這些元件。

安裝完成並顯示歡迎畫面後，繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新版 Android SDK。但使用原生代碼構建 React Native 應用程式需要特定的 `Android 14 (UpsideDownCake)` SDK。可透過 Android Studio 的 SDK 管理器安裝其他 Android SDK。

開啟 Android Studio，點擊「More Actions」按鈕並選擇「SDK Manager」。

![Android Studio 歡迎畫面](/docs/assets/GettingStartedAndroidStudioWelcomeWindows.png)

> SDK 管理器也可在 Android Studio「設定」對話框的 **Languages & Frameworks** → **Android SDK** 中找到。

在 SDK 管理器中選擇「SDK Platforms」標籤頁，然後勾選右下角的「顯示套件詳細資訊」。找到並展開 `Android 14 (UpsideDownCake)` 項目，確保以下項目已被勾選：

- `Android SDK Platform 34`
- `Intel x86 Atom_64 System Image` or `Google APIs Intel x86 Atom System Image`

接著，選擇「SDK Tools」標籤頁，同樣勾選「顯示套件詳細資訊」。找到並展開 `Android SDK Build-Tools` 項目，確保已選取 `34.0.0` 版本。

最後，點擊「Apply」以下載並安裝 Android SDK 及相關建置工具。

<h4>3. Configure the ANDROID_HOME environment variable</h4>

React Native 工具需要設定某些環境變數才能建置包含原生代碼的應用程式。

1. Open the **Windows Control Panel.**
2. Click on **User Accounts,** then click **User Accounts** again
3. Click on **Change my environment variables**
4. Click on **New...** to create a new `ANDROID_HOME` user variable that points to the path to your Android SDK:

![ANDROID_HOME 環境變數](/docs/assets/GettingStartedAndroidEnvironmentVariableANDROID_HOME.png)

SDK 預設安裝於以下位置：

```powershell
%LOCALAPPDATA%\Android\Sdk
```

您可以在 Android Studio 的「設定」對話框中找到 SDK 的實際位置，路徑為 **Languages & Frameworks** → **Android SDK**。

開啟新的命令提示字元視窗，以確保新環境變數已載入，再繼續下一步。

1. 開啟 powershell
2. 複製並貼上 **Get-ChildItem -Path Env:\\** 至 powershell
3. 確認 `ANDROID_HOME` 已成功新增

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

React Native 內建命令列介面。與其全域安裝並管理特定版本的 CLI，我們建議您透過 Node.js 自帶的 `npx` 在執行時存取當前版本。使用 `npx react-native <command>` 時，系統會自動下載並執行當前穩定版的 CLI。

<h2>Creating a new application</h2>

<RemoveGlobalCLI />

React Native 內建命令列介面，可用於生成新專案。您無需全域安裝任何套件，直接使用 Node.js 自帶的 `npx` 即可存取。讓我們建立一個名為「AwesomeProject」的新 React Native 專案：

```shell
npx react-native@latest init AwesomeProject
```

若您要將 React Native 整合至現有應用程式、或已在專案中安裝 [Expo](https://docs.expo.dev/bare/installing-expo-modules/)、或要為現有 React Native 專案新增 Android 支援（參見[與現有應用程式整合](integration-with-existing-apps.md)），則無需執行此步驟。您也可以使用第三方 CLI（如 [Ignite CLI](https://github.com/infinitered/ignite)）來初始化 React Native 應用程式。

<h3>[Optional] Using a specific version or template</h3>

若要以特定 React Native 版本建立新專案，可使用 `--version` 參數：

```shell
npx react-native@X.XX.X init AwesomeProject --version X.XX.X
```

您也可以使用 `--template` 參數來以自訂的 React Native 模板啟動專案。

<h2>Preparing the Android device</h2>

您需要一個 Android 裝置來執行您的 React Native Android 應用程式。這可以是實體的 Android 裝置，或者更常見的是使用 Android 虛擬裝置 (AVD)，讓您在電腦上模擬 Android 裝置。

無論哪種方式，您都需要準備裝置以執行開發用的 Android 應用程式。

<h3>Using a physical device</h3>

如果您有實體的 Android 裝置，可以透過 USB 線將其連接到電腦，並按照[這裡](running-on-device.md)的指示來取代 AVD 進行開發。

<h3>Using a virtual device</h3>

如果您使用 Android Studio 開啟 `./AwesomeProject/android`，可以透過 Android Studio 中的「AVD Manager」查看可用的 Android 虛擬裝置 (AVDs) 清單。尋找類似這樣的圖示：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

如果您最近安裝了 Android Studio，可能需要[建立新的 AVD](https://developer.android.com/studio/run/managing-avds.html)。選擇「Create Virtual Device...」，然後從清單中選擇任何手機並點擊「Next」，接著選擇 **UpsideDownCake** API Level 34 映像檔。

> 如果沒有安裝 HAXM，請點擊「Install HAXM」或按照[這些指示](https://github.com/intel/haxm/wiki/Installation-Instructions-on-Windows)進行設定，然後返回 AVD Manager。

點擊「Next」然後「Finish」以建立您的 AVD。此時您應該可以點擊 AVD 旁邊的綠色三角形按鈕來啟動它，接著進行下一步。

<h2>Running your React Native application</h2>

<h3>Step 1: Start Metro</h3>

[**Metro**](https://facebook.github.io/metro/) 是 React Native 的 JavaScript 建置工具。要啟動 Metro 開發伺服器，請在您的專案資料夾中執行以下指令：

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
如果您熟悉網頁開發，Metro 類似於 Vite 和 webpack 等打包工具，但專為 React Native 設計。例如，Metro 使用 [Babel](https://babel.dev/) 將 JSX 等語法轉換為可執行的 JavaScript。
:::

<h3>Step 2: Start your application</h3>

讓 Metro Bundler 在其專用的終端機中執行。在您的 React Native 專案資料夾中開啟新的終端機，執行以下指令：

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

如果一切設定正確，您應該很快就能在 Android 模擬器中看到您的新應用程式執行。

![AwesomeProject on Android](/docs/assets/GettingStartedAndroidSuccessWindows.png)

這是執行應用程式的一種方式——您也可以直接從 Android Studio 中執行它。

> 如果無法正常執行，請參閱[疑難排解](troubleshooting.md)頁面。

<h3>Modifying your app</h3>

現在您已成功執行了應用程式，讓我們來修改它。

- Open `App.tsx` in your text editor of choice and edit some lines.
- Press the <kbd>R</kbd> key twice or select `Reload` from the Dev Menu (<kbd>Ctrl</kbd> + <kbd>M</kbd>) to see your changes!

<h3>That's it!</h3>

恭喜！您已成功執行並修改了您的第一個 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- 如果想將這段新的 React Native 程式碼整合到現有應用程式中，請參閱[整合指南](integration-with-existing-apps.md)。

如果想進一步了解 React Native，請查看[React Native 簡介](getting-started)。