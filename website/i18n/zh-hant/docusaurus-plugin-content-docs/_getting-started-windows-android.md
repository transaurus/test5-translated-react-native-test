<h2>Installing dependencies</h2>

您需要安裝 Node、React Native 命令行工具、JDK 和 Android Studio。

雖然您可以選擇任何編輯器來開發應用程式，但必須安裝 Android Studio 才能設置建置 React Native Android 應用所需的工具鏈。

<h3 id="jdk">Node, JDK</h3>

我們建議透過 [Chocolatey](https://chocolatey.org/install)（Windows 上流行的套件管理工具）來安裝 Node。

建議使用 Node 的 LTS 版本。若需切換不同版本，可透過 [nvm-windows](https://github.com/coreybutler/nvm-windows)（Windows 的 Node 版本管理工具）安裝 Node。

React Native 還需要 [Java SE 開發套件 (JDK)](https://openjdk.java.net/projects/jdk/17/)，同樣可透過 Chocolatey 安裝。

以系統管理員身分開啟命令提示字元（對命令提示字元按右鍵選擇「以系統管理員身分執行」），然後執行以下指令：

```powershell
choco install -y nodejs-lts microsoft-openjdk17
```

若系統已安裝 Node，請確認版本為 18 或更新。若已安裝 JDK，建議使用 JDK17。使用更高版本的 JDK 可能會遇到問題。

> 您可以在 [Node 下載頁面](https://nodejs.org/en/download/) 找到其他安裝選項。

> If you're using the latest version of Java Development Kit, you'll need to change the Gradle version of your project so it can recognize the JDK. You can do that by going to `{project root folder}\android\gradle\wrapper\gradle-wrapper.properties` and changing the `distributionUrl` value to upgrade the Gradle version. You can check out [here the latest releases of Gradle](https://gradle.org/releases/).

<h3>Android development environment</h3>

若您是 Android 開發新手，設置開發環境可能較為繁瑣。若已熟悉 Android 開發，則只需配置部分項目。無論如何，請仔細遵循後續步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在安裝精靈中，請確保勾選以下所有項目：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`
- 若未使用 Hyper-V：`Performance (Intel ® HAXM)`（[AMD 或 Hyper-V 用戶請參閱此處](https://android-developers.googleblog.com/2018/07/android-emulator-amd-processor-hyper-v.html)）

接著點擊「下一步」安裝所有元件。

> 若選項呈灰色不可選，後續仍可安裝這些元件。

完成安裝並顯示歡迎畫面後，請繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新版 Android SDK。但使用原生碼建置 React Native 應用需特定版本 `Android 15 (VanillaIceCream)` SDK。可透過 Android Studio 的 SDK 管理器安裝其他版本。

開啟 Android Studio，點擊「More Actions」按鈕並選擇「SDK Manager」。

![Android Studio 歡迎畫面](/docs/assets/GettingStartedAndroidStudioWelcomeWindows.png)

> SDK 管理器也可在 Android Studio 的「Settings」對話框中找到，路徑為 **Languages & Frameworks** → **Android SDK**。

在 SDK 管理器中選擇「SDK Platforms」分頁，然後勾選右下角的「Show Package Details」。找到並展開 `Android 15 (VanillaIceCream)` 項目，確保以下項目已勾選：

- `Android SDK Platform 35`
- `Intel x86 Atom_64 System Image` 或 `Google APIs Intel x86 Atom System Image`

接著選擇「SDK Tools」分頁，同樣勾選「Show Package Details」。找到並展開 `Android SDK Build-Tools` 項目，確保已選取 `35.0.0` 版本。

最後點擊「Apply」以下載並安裝 Android SDK 及相關建置工具。

<h4>3. Configure the ANDROID_HOME environment variable</h4>

React Native 工具需要設定一些環境變數才能建置包含原生代碼的應用程式。

1. Open the **Windows Control Panel.**
2. Click on **User Accounts,** then click **User Accounts** again
3. Click on **Change my environment variables**
4. Click on **New...** to create a new `ANDROID_HOME` user variable that points to the path to your Android SDK:

![ANDROID_HOME 環境變數](/docs/assets/GettingStartedAndroidEnvironmentVariableANDROID_HOME.png)

SDK 預設安裝在以下位置：

```powershell
%LOCALAPPDATA%\Android\Sdk
```

你可以在 Android Studio 的「設定」對話框中找到 SDK 的實際位置，路徑為 **Languages & Frameworks** → **Android SDK**。

開啟一個新的命令提示字元視窗，以確保新的環境變數已載入，再繼續下一步。

1. 開啟 powershell
2. 複製並貼上 **Get-ChildItem -Path Env:\\** 到 powershell
3. 確認 `ANDROID_HOME` 已新增

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

<h2>Preparing the Android device</h2>

你需要一個 Android 裝置來執行你的 React Native Android 應用程式。這可以是實體的 Android 裝置，或者更常見的是使用 Android 虛擬裝置（AVD），它允許你在電腦上模擬 Android 裝置。

無論哪種方式，你都需要準備裝置以執行開發用的 Android 應用程式。

<h3>Using a physical device</h3>

如果你有實體的 Android 裝置，你可以透過 USB 線將其連接到電腦，並按照[這裡](running-on-device.md)的指示來取代 AVD 進行開發。

<h3>Using a virtual device</h3>

如果你使用 Android Studio 開啟 `./AwesomeProject/android`，可以透過 Android Studio 內的「AVD Manager」查看可用的 Android 虛擬裝置（AVDs）清單。尋找類似以下的圖示：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

如果你最近安裝了 Android Studio，可能需要[建立一個新的 AVD](https://developer.android.com/studio/run/managing-avds.html)。選擇「Create Virtual Device...」，然後從清單中選擇任一手機，點擊「Next」，接著選擇 **VanillaIceCream** API Level 35 映像檔。

> 若尚未安裝 HAXM，請點擊「安裝 HAXM」或依照[這些指示](https://github.com/intel/haxm/wiki/Installation-Instructions-on-Windows)進行設定，完成後返回 AVD 管理員。

點擊「下一步」後選擇「完成」以建立您的 AVD。此時您應能點擊 AVD 旁的綠色三角形按鈕來啟動模擬器。

<h3>That's it!</h3>

恭喜！您已成功設定開發環境。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- 若需將此 React Native 程式碼整合至現有應用程式，請參閱[整合指南](integration-with-existing-apps.md)。
- 若想深入瞭解 React Native，請參閱[React Native 簡介](getting-started)。