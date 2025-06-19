## 安裝依賴套件

您需要安裝 Node、React Native 命令行界面、JDK 以及 Android Studio。

雖然您可以使用任何編輯器開發應用程式，但必須安裝 Android Studio 才能設置建置 React Native Android 應用程式所需的工具。

<h3>Node</h3>

請依照 [Linux 發行版的安裝說明](https://nodejs.org/en/download/package-manager/) 安裝 Node 18.18 或更新版本。

<h3>Java Development Kit</h3>

React Native currently recommends version 17 of the Java SE Development Kit (JDK). You may encounter problems using higher JDK versions. You may download and install [OpenJDK](https://openjdk.java.net) from [AdoptOpenJDK](https://adoptopenjdk.net/) or your system packager.

<h3>Android development environment</h3>

如果您是 Android 開發新手，設置開發環境可能會有些繁瑣。若您已熟悉 Android 開發，則只需進行部分配置。無論哪種情況，請務必仔細遵循接下來的步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在安裝精靈中，請確保勾選以下所有項目：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`

接著點擊「Next」安裝這些元件。

> 若複選框呈灰色不可選狀態，您稍後仍有機會安裝這些元件。

當安裝完成並顯示歡迎畫面後，請繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新版 Android SDK。但使用原生程式碼建置 React Native 應用程式時，需要特定的 `Android 14 (UpsideDownCake)` SDK。您可透過 Android Studio 的 SDK 管理器安裝其他 Android SDK。

開啟 Android Studio，點擊「Configure」按鈕並選擇「SDK Manager」。

> 您也可以在 Android Studio 的「Settings」對話框中找到 SDK 管理器，路徑為 **Languages & Frameworks** → **Android SDK**。

在 SDK 管理器中選擇「SDK Platforms」標籤頁，然後勾選右下角的「Show Package Details」。找到並展開 `Android 14 (UpsideDownCake)` 項目，確保勾選以下內容：

- `Android SDK Platform 34`
- `Intel x86 Atom_64 System Image` 或 `Google APIs Intel x86 Atom System Image`

接著選擇「SDK Tools」標籤頁，同樣勾選「Show Package Details」。找到並展開「Android SDK Build-Tools」項目，確保選中 `34.0.0` 版本。

最後點擊「Apply」下載並安裝 Android SDK 及相關建置工具。

<h4>3. Configure the ANDROID_HOME environment variable</h4>

React Native 工具需要設置部分環境變數才能建置含原生程式碼的應用程式。

將以下內容加入您的 `$HOME/.bash_profile` 或 `$HOME/.bashrc` 設定檔（若使用 `zsh` 則為 `~/.zprofile` 或 `~/.zshrc`）：

```shell
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

> `.bash_profile` 專屬於 `bash`。若使用其他 shell，您需要編輯對應的 shell 專用設定檔。

Type `source $HOME/.bash_profile` for `bash` or `source $HOME/.zprofile` to load the config into your current shell. Verify that ANDROID_HOME has been set by running `echo $ANDROID_HOME` and the appropriate directories have been added to your path by running `echo $PATH`.

> 請務必確認使用正確的 Android SDK 路徑。您可以在 Android Studio 的「Settings」對話框中找到 SDK 實際位置，路徑為 **Languages & Frameworks** → **Android SDK**。

<h3>Watchman</h3>

請遵循 [Watchman 安裝指南](https://facebook.github.io/watchman/docs/install#buildinstall) 從原始碼編譯並安裝 Watchman。

> [Watchman](https://facebook.github.io/watchman/docs/install) 是 Facebook 開發的檔案系統監控工具。強烈建議安裝以提升效能，並增強特定邊緣案例的相容性（註：不安裝可能仍可運作，但效果因環境而異；預先安裝可避免後續潛在問題）。

<h2>Preparing the Android device</h2>

執行 React Native Android 應用程式需準備 Android 裝置，可以是實體裝置，或更常見的 Android 虛擬裝置（AVD），後者能在電腦上模擬 Android 環境。

無論採用何種方式，均需將裝置設定為開發模式以執行 Android 應用程式。

<h3>Using a physical device</h3>

若擁有實體 Android 裝置，可透過 USB 連接電腦後，依照[此處指南](running-on-device.md)設定，即可替代 AVD 進行開發。

<h3>Using a virtual device</h3>

若透過 Android Studio 開啟 `./AwesomeProject/android` 專案，點選「AVD Manager」圖示（形如：）即可查看可用虛擬裝置清單。

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

若剛安裝 Android Studio，需[建立新 AVD](https://developer.android.com/studio/run/managing-avds.html)。選擇「Create Virtual Device...」後，任選手機型號點擊「Next」，接著選取 **UpsideDownCake** API Level 34 系統映像檔。

> 建議設定[虛擬機器加速](https://developer.android.com/studio/run/emulator-acceleration.html#vm-linux)以提升效能。完成設定後，返回 AVD Manager 繼續操作。

點擊「Next」後選擇「Finish」完成 AVD 建立。此時點擊 AVD 旁的綠色三角形按鈕即可啟動模擬器。

<h3>That's it!</h3>

恭喜！您已成功設定開發環境。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- 若需將 React Native 程式碼整合至現有應用程式，請參閱[整合指南](integration-with-existing-apps.md)。
- 若想深入瞭解 React Native，請查看[React Native 簡介](getting-started)。