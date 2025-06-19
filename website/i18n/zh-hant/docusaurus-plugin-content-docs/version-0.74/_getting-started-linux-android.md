## 安裝依賴套件

您需要安裝 Node、React Native 命令行界面、JDK 以及 Android Studio。

雖然您可以使用任何編輯器來開發應用程式，但必須安裝 Android Studio 才能設置建置 React Native Android 應用程式所需的工具。

<h3>Node</h3>

請依照 [Linux 發行版的安裝說明](https://nodejs.org/en/download/package-manager/) 安裝 Node 18 或更新版本。

<h3>Java Development Kit</h3>

React Native currently recommends version 17 of the Java SE Development Kit (JDK). You may encounter problems using higher JDK versions. You may download and install [OpenJDK](https://openjdk.java.net) from [AdoptOpenJDK](https://adoptopenjdk.net/) or your system packager.

<h3>Android development environment</h3>

如果您是 Android 開發的新手，設置開發環境可能會有些繁瑣。如果您已經熟悉 Android 開發，則可能需要進行一些配置。無論哪種情況，請務必仔細遵循接下來的幾個步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在 Android Studio 安裝精靈中，請確保勾選以下所有項目的方框：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`

然後，點擊「Next」安裝所有這些元件。

> 如果複選框呈灰色，您稍後仍有機會安裝這些元件。

安裝完成並顯示歡迎畫面後，請繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新的 Android SDK。然而，使用原生程式碼建置 React Native 應用程式需要特定的 `Android 14 (UpsideDownCake)` SDK。您可以透過 Android Studio 的 SDK 管理器安裝其他 Android SDK。

為此，請開啟 Android Studio，點擊「Configure」按鈕並選擇「SDK Manager」。

> SDK 管理器也可以在 Android Studio 的「Settings」對話框中找到，位於 **Languages & Frameworks** → **Android SDK**。

在 SDK 管理器中選擇「SDK Platforms」標籤，然後勾選右下角的「Show Package Details」。找到並展開 `Android 14 (UpsideDownCake)` 條目，確保以下項目已勾選：

- `Android SDK Platform 34`
- `Intel x86 Atom_64 System Image` 或 `Google APIs Intel x86 Atom System Image`

接著，選擇「SDK Tools」標籤，同樣勾選「Show Package Details」。找到並展開「Android SDK Build-Tools」條目，確保已選擇 `34.0.0`。

最後，點擊「Apply」以下載並安裝 Android SDK 及相關建置工具。

<h4>3. Configure the ANDROID_HOME environment variable</h4>

React Native 工具需要設置一些環境變數才能使用原生程式碼建置應用程式。

將以下行添加到您的 `$HOME/.bash_profile` 或 `$HOME/.bashrc` 配置文件中（如果您使用 `zsh`，則為 `~/.zprofile` 或 `~/.zshrc`）：

```shell
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

> `.bash_profile` 是 `bash` 專用的。如果您使用其他 shell，則需要編輯相應的 shell 專用配置文件。

Type `source $HOME/.bash_profile` for `bash` or `source $HOME/.zprofile` to load the config into your current shell. Verify that ANDROID_HOME has been set by running `echo $ANDROID_HOME` and the appropriate directories have been added to your path by running `echo $PATH`.

> 請務必使用正確的 Android SDK 路徑。您可以在 Android Studio 的「設定」對話框中找到 SDK 的實際位置，路徑為：**Languages & Frameworks** → **Android SDK**。

<h3>Watchman</h3>

請遵循 [Watchman 安裝指南](https://facebook.github.io/watchman/docs/install#buildinstall) 從原始碼編譯並安裝 Watchman。

> [Watchman](https://facebook.github.io/watchman/docs/install) 是 Facebook 開發的檔案系統監控工具。強烈建議安裝以提升效能，並避免某些邊緣案例的相容性問題（註：雖然不安裝仍可運作，但可能導致後續開發困擾）。

<h2>Preparing the Android device</h2>

執行 React Native Android 應用程式需要 Android 裝置。您可以使用實體 Android 裝置，或更常見的做法是透過 Android 虛擬裝置 (AVD) 在電腦上模擬 Android 環境。

無論採用何種方式，均需預先設定裝置以支援開發用途的 Android 應用程式執行。

<h3>Using a physical device</h3>

若擁有實體 Android 裝置，可透過 USB 連接電腦後，依照[此處說明](running-on-device.md)進行設定，即可替代 AVD 進行開發。

<h3>Using a virtual device</h3>

若透過 Android Studio 開啟 `./AwesomeProject/android` 專案，可點選 AVD Manager 圖示（圖示樣式如下）查看可用虛擬裝置清單：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

若剛安裝 Android Studio，需先[建立新 AVD](https://developer.android.com/studio/run/managing-avds.html)。選擇「Create Virtual Device...」後，任選手機型號點擊「Next」，接著選擇 **UpsideDownCake** API Level 34 系統映像檔。

> 建議設定 [VM 加速](https://developer.android.com/studio/run/emulator-acceleration.html#vm-linux)以提升效能。完成設定後，請返回 AVD Manager 繼續操作。

點擊「Next」後選擇「Finish」完成 AVD 建立。此時點擊 AVD 旁的綠色三角形按鈕即可啟動模擬器。

<h3>That's it!</h3>

恭喜！您已成功完成開發環境設定。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- If you want to add this new React Native code to an existing application, check out the [Integration guide](integration-with-existing-apps.md).
- If you're curious to learn more about React Native, check out the [Introduction to React Native](getting-started).