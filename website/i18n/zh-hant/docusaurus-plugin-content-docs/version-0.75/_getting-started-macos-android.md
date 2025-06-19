## 安裝依賴套件

您需要安裝 Node、Watchman、React Native 命令行工具、JDK 以及 Android Studio。

雖然您可以使用任何喜歡的編輯器來開發應用程式，但必須安裝 Android Studio 才能設置必要的工具來為 Android 構建 React Native 應用程式。

<h3>Node &amp; Watchman</h3>

我們建議使用 [Homebrew](https://brew.sh/) 來安裝 Node 和 Watchman。安裝 Homebrew 後，在終端機中執行以下命令：

```shell
brew install node
brew install watchman
```

如果您的系統已安裝 Node，請確保版本為 18.18 或更新。

[Watchman](https://facebook.github.io/watchman) 是 Facebook 開發的檔案系統監控工具。強烈建議安裝以獲得更好的效能。

<h3>Java Development Kit</h3>

We recommend installing the OpenJDK distribution called Azul **Zulu** using [Homebrew](https://brew.sh/). Run the following commands in a Terminal after installing Homebrew:

```shell
brew install --cask zulu@17

# Get path to where cask was installed to double-click installer
brew info --cask zulu@17
```

After the JDK installation, add or update your `JAVA_HOME` environment variable in `~/.zshrc` (or in `~/.bash_profile`) .

若按照上述步驟操作，JDK 通常會安裝在以下路徑：`/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home`：

```shell
export JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home
```

Zulu OpenJDK 發行版提供 **Intel 和 M1 Mac 專用** 的 JDK。這能確保在 M1 Mac 上的構建速度比使用 Intel 架構的 JDK 更快。

如果您的系統已安裝 JDK，建議使用 JDK 17 版本。使用更高版本可能會遇到問題。

<h3>Android development environment</h3>

如果您是 Android 開發新手，設置開發環境可能會有些繁瑣。即使您已有經驗，也可能需要進行一些配置。無論如何，請仔細遵循接下來的步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在安裝精靈中，請確保勾選以下所有項目：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`

接著點擊「Next」安裝這些元件。

> 如果選項呈灰色不可選，稍後仍有機會安裝這些元件。

安裝完成並顯示歡迎畫面後，請繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新版 Android SDK。但使用原生碼構建 React Native 應用程式需要特定的 `Android 14 (UpsideDownCake)` SDK。可透過 Android Studio 的 SDK 管理器安裝其他版本。

開啟 Android Studio 後，點擊「More Actions」按鈕並選擇「SDK Manager」。

![Android Studio 歡迎畫面](/docs/assets/GettingStartedAndroidStudioWelcomeMacOS.png)

> 也可在 Android Studio 的「Settings」對話框中找到 SDK 管理器，路徑為：**Languages & Frameworks** → **Android SDK**。

在 SDK 管理器中選擇「SDK Platforms」標籤，然後勾選右下角的「Show Package Details」。找到並展開 `Android 14 (UpsideDownCake)` 項目，確保以下內容已勾選：

- `Android SDK Platform 34`
- `Intel x86 Atom_64 System Image` or `Google APIs Intel x86 Atom System Image` or (for Apple M1 Silicon) `Google APIs ARM 64 v8a System Image`

接著，選擇「SDK Tools」標籤頁，同樣勾選「顯示套件詳細資訊」。找到並展開「Android SDK Build-Tools」項目，確保選中 `34.0.0` 版本。

最後點擊「Apply」按鈕下載並安裝 Android SDK 及相關建置工具。

<h4>3. Configure the ANDROID_HOME environment variable</h4>

React Native 工具需要設定某些環境變數才能建置含原生代碼的應用程式。

請將以下內容加入您的 `~/.zprofile` 或 `~/.zshrc` 設定檔（若使用 `bash` 則為 `~/.bash_profile` 或 `~/.bashrc`）：

```shell
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

Run `source ~/.zprofile` (or `source ~/.bash_profile` for `bash`) to load the config into your current shell. Verify that ANDROID_HOME has been set by running `echo $ANDROID_HOME` and the appropriate directories have been added to your path by running `echo $PATH`.

> 請務必使用正確的 Android SDK 路徑。您可以在 Android Studio 的「Settings」對話框中，於 **Languages & Frameworks** → **Android SDK** 找到 SDK 的實際位置。

<h2>Preparing the Android device</h2>

您需要一台 Android 裝置來執行 React Native Android 應用程式。這可以是實體 Android 裝置，或更常見的是使用 Android 虛擬裝置 (AVD) 在電腦上模擬 Android 裝置。

無論哪種方式，您都需要準備裝置以執行開發用的 Android 應用程式。

<h3>Using a physical device</h3>

若有實體 Android 裝置，可透過 USB 連接電腦後，依照[此處說明](running-on-device.md)設定為開發用途，以替代 AVD。

<h3>Using a virtual device</h3>

若用 Android Studio 開啟 `./AwesomeProject/android`，可透過 Android Studio 內的「AVD Manager」查看可用虛擬裝置清單。尋找如下圖示：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

若剛安裝 Android Studio，可能需要[建立新 AVD](https://developer.android.com/studio/run/managing-avds.html)。選擇「Create Virtual Device...」，任選一款手機後點擊「Next」，接著選擇 **UpsideDownCake** API Level 34 映像檔。

點擊「Next」後按「Finish」建立 AVD。此時應可點擊 AVD 旁的綠色三角形按鈕啟動模擬器。

<h3>That's it!</h3>

恭喜！您已成功設定開發環境。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- 若要將此 React Native 代碼整合至現有應用程式，請參閱[整合指南](integration-with-existing-apps.md)。
- 若想深入瞭解 React Native，請參閱[React Native 簡介](getting-started)。