## 安裝依賴套件

您需要安裝 Node、React Native 命令行界面、JDK 以及 Android Studio。

雖然您可以使用任何喜歡的編輯器來開發應用程式，但必須安裝 Android Studio 才能設置必要的工具來為 Android 構建 React Native 應用程式。

<h3>Node</h3>

請依照 [Linux 發行版的安裝說明](https://nodejs.org/en/download/package-manager/) 安裝 Node 18.18 或更新版本。

<h3>Java Development Kit</h3>

React Native 目前推薦使用 Java SE 開發套件 (JDK) 第 17 版。使用更高版本的 JDK 可能會遇到問題。您可以從 [OpenJDK](https://openjdk.java.net) 或 [AdoptOpenJDK](https://adoptopenjdk.net/) 下載並安裝，或透過系統套件管理器安裝。

<h3>Android development environment</h3>

如果您是 Android 開發的新手，設置開發環境可能會有些繁瑣。如果您已經熟悉 Android 開發，則可能需要進行一些配置。無論哪種情況，請務必仔細遵循接下來的幾個步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在 Android Studio 安裝精靈中，請確保勾選以下所有項目的方框：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`

然後，點擊「Next」安裝所有這些組件。

> 如果複選框呈灰色，您稍後仍有機會安裝這些組件。

安裝完成並顯示歡迎畫面後，請繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新的 Android SDK。然而，使用原生代碼構建 React Native 應用程式需要特定的 `Android 15 (VanillaIceCream)` SDK。可以透過 Android Studio 中的 SDK 管理器安裝其他 Android SDK。

為此，請打開 Android Studio，點擊「Configure」按鈕並選擇「SDK Manager」。

> 也可以在 Android Studio 的「Settings」對話框中找到 SDK 管理器，位於 **Languages & Frameworks** → **Android SDK**。

在 SDK 管理器中選擇「SDK Platforms」標籤，然後勾選右下角的「Show Package Details」。找到並展開 `Android 15 (VanillaIceCream)` 條目，確保勾選以下項目：

- `Android SDK Platform 35`
- `Intel x86 Atom_64 System Image` 或 `Google APIs Intel x86 Atom System Image`

接著，選擇「SDK Tools」標籤並同樣勾選「Show Package Details」。找到並展開「Android SDK Build-Tools」條目，確保選擇 `35.0.0`。

最後，點擊「Apply」下載並安裝 Android SDK 及相關構建工具。

<h4>3. Configure the ANDROID_HOME environment variable</h4>

React Native 工具需要設置一些環境變數才能使用原生代碼構建應用程式。

將以下行添加到您的 `$HOME/.bash_profile` 或 `$HOME/.bashrc`（如果您使用 `zsh`，則為 `~/.zprofile` 或 `~/.zshrc`）配置文件中：

```shell
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

> `.bash_profile` 是 `bash` 專用的。如果您使用其他 shell，則需要編輯相應的 shell 專用配置文件。

Type `source $HOME/.bash_profile` for `bash` or `source $HOME/.zprofile` to load the config into your current shell. Verify that ANDROID_HOME has been set by running `echo $ANDROID_HOME` and the appropriate directories have been added to your path by running `echo $PATH`.

> 請務必使用正確的 Android SDK 路徑。您可以在 Android Studio 的「Settings」對話框中找到 SDK 實際位置，路徑為 **Languages & Frameworks** → **Android SDK**。

<h3>Watchman</h3>

請遵循 [Watchman 安裝指南](https://facebook.github.io/watchman/docs/install#buildinstall) 從原始碼編譯並安裝 Watchman。

> [Watchman](https://facebook.github.io/watchman/docs/install) 是 Facebook 開發的檔案系統監控工具。強烈建議安裝以提升效能並增強特定邊緣案例的相容性（註：不安裝可能仍可運作，但效果因環境而異；預先安裝可避免後續潛在問題）。

<h2>Preparing the Android device</h2>

您需要一台 Android 裝置來執行 React Native Android 應用程式。可使用實體 Android 裝置，或更常見的做法是使用 Android 虛擬裝置 (AVD) 在電腦上模擬 Android 環境。

無論採用何種方式，均需將裝置設定為開發模式以執行 Android 應用程式。

<h3>Using a physical device</h3>

若擁有實體 Android 裝置，可透過 USB 連接電腦後，依照[此處說明](running-on-device.md) 設定以替代 AVD 進行開發。

<h3>Using a virtual device</h3>

若使用 Android Studio 開啟 `./AwesomeProject/android`，可透過點選 Android Studio 內的「AVD Manager」圖示（外觀如下）查看可用虛擬裝置清單：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

若剛安裝 Android Studio，可能需要[建立新 AVD](https://developer.android.com/studio/run/managing-avds.html)。選擇「Create Virtual Device...」後，從清單任選手機型號點擊「Next」，接著選擇 **VanillaIceCream** API Level 35 映像檔。

> 建議設定 [VM 加速](https://developer.android.com/studio/run/emulator-acceleration.html#vm-linux) 以提升效能。完成設定後請返回 AVD Manager。

點擊「Next」後選擇「Finish」建立 AVD。此時點擊 AVD 旁的綠色三角形按鈕即可啟動模擬器。

<h3>That's it!</h3>

恭喜！您已成功設定開發環境。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- 若要將 React Native 程式碼整合至現有應用程式，請參閱[整合指南](integration-with-existing-apps.md)。
- 若想深入瞭解 React Native，請查看[React Native 簡介](getting-started)。