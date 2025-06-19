## 安裝依賴套件

您需要安裝 Node、Watchman、React Native 命令行工具、JDK 和 Android Studio。

雖然您可以使用任何喜歡的編輯器開發應用程式，但必須安裝 Android Studio 才能設置建置 React Native Android 應用程式所需的工具鏈。

<h3>Node &amp; Watchman</h3>

建議使用 [Homebrew](https://brew.sh/) 安裝 Node 和 Watchman。安裝 Homebrew 後，在終端機執行以下命令：

```shell
brew install node
brew install watchman
```

若系統已安裝 Node，請確認版本為 18 或更新。

[Watchman](https://facebook.github.io/watchman) 是 Facebook 開發的檔案系統監控工具。強烈建議安裝以獲得更好的效能表現。

<h3>Java Development Kit</h3>

We recommend installing the OpenJDK distribution called Azul **Zulu** using [Homebrew](https://brew.sh/). Run the following commands in a Terminal after installing Homebrew:

```shell
brew install --cask zulu@17

# Get path to where cask was installed to double-click installer
brew info --cask zulu@17
```

After the JDK installation, add or update your `JAVA_HOME` environment variable in `~/.zshrc` (or in `~/.bash_profile`) .

若依照上述步驟安裝，JDK 預設路徑通常為 `/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home`：

```shell
export JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home
```

Zulu OpenJDK 發行版提供 **同時支援 Intel 和 M1 晶片 Mac** 的 JDK。這能確保在 M1 Mac 上獲得比 Intel 版 JDK 更快的建置速度。

若系統已安裝 JDK，建議使用 JDK 17 版本。使用更高版本可能會遇到相容性問題。

<h3>Android development environment</h3>

對於 Android 開發新手來說，設置開發環境可能較為繁瑣。若您已有 Android 開發經驗，則只需調整部分設定。無論哪種情況，請務必仔細遵循後續步驟。

<h4 id="android-studio">1. Install Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在安裝精靈中，請確保勾選以下所有項目：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`

接著點擊「Next」安裝這些元件。

> 若選項呈灰色不可選狀態，後續仍可透過 Android Studio 安裝這些元件。

完成安裝並顯示歡迎畫面後，請繼續下一步。

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio 預設會安裝最新版 Android SDK。但使用原生碼建置 React Native 應用程式時，需特定安裝 `Android 14 (UpsideDownCake)` SDK。可透過 Android Studio 的 SDK 管理員安裝其他版本 SDK。

開啟 Android Studio 後，點擊「More Actions」按鈕並選擇「SDK Manager」。

![Android Studio 歡迎畫面](/docs/assets/GettingStartedAndroidStudioWelcomeMacOS.png)

> 也可在 Android Studio 的「Settings」對話框中，透過 **Languages & Frameworks** → **Android SDK** 路徑找到 SDK Manager。

在 SDK Manager 中選擇「SDK Platforms」標籤頁，然後勾選右下角的「Show Package Details」。找到並展開 `Android 14 (UpsideDownCake)` 項目，確保以下元件已被選中：

- `Android SDK Platform 34`
- `Intel x86 Atom_64 System Image` or `Google APIs Intel x86 Atom System Image` or (for Apple M1 Silicon) `Google APIs ARM 64 v8a System Image`

接著，選擇「SDK Tools」標籤頁，同樣勾選「顯示套件詳細資訊」。找到並展開「Android SDK Build-Tools」項目，確保已選取 `34.0.0` 版本。

最後，點擊「Apply」以下載並安裝 Android SDK 及相關建置工具。

<h4>3. Configure the ANDROID_HOME environment variable</h4>

React Native 工具需要設定某些環境變數才能建置包含原生代碼的應用程式。

將以下內容加入您的 `~/.zprofile` 或 `~/.zshrc` 設定檔（若使用 `bash`，則為 `~/.bash_profile` 或 `~/.bashrc`）：

```shell
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

Run `source ~/.zprofile` (or `source ~/.bash_profile` for `bash`) to load the config into your current shell. Verify that ANDROID_HOME has been set by running `echo $ANDROID_HOME` and the appropriate directories have been added to your path by running `echo $PATH`.

> 請確保使用正確的 Android SDK 路徑。您可以在 Android Studio 的「Settings」對話框中找到 SDK 的實際位置，路徑為 **Languages & Frameworks** → **Android SDK**。

<h2>Preparing the Android device</h2>

您需要一台 Android 裝置來執行 React Native Android 應用程式。這可以是實體 Android 裝置，或更常見的是使用 Android 虛擬裝置 (AVD)，讓您能在電腦上模擬 Android 裝置。

無論哪種方式，您都需要準備裝置以執行開發用的 Android 應用程式。

<h3>Using a physical device</h3>

若您擁有實體 Android 裝置，可透過 USB 連接線將其連接至電腦，並依照[此處說明](running-on-device.md)設定，即可用於開發以取代 AVD。

<h3>Using a virtual device</h3>

若使用 Android Studio 開啟 `./AwesomeProject/android`，可透過 Android Studio 內的「AVD Manager」查看可用虛擬裝置清單。尋找如下圖示：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

若您剛安裝 Android Studio，可能需要[建立新 AVD](https://developer.android.com/studio/run/managing-avds.html)。選擇「Create Virtual Device...」，從清單中任選一款手機後點擊「Next」，接著選取 **UpsideDownCake** API Level 34 映像檔。

點擊「Next」後選擇「Finish」以建立 AVD。此時您應可點擊 AVD 旁的綠色三角形按鈕來啟動它。

<h3>That's it!</h3>

恭喜！您已成功設定開發環境。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>Now what?</h2>

- 若想將此 React Native 代碼整合至現有應用程式，請參閱[整合指南](integration-with-existing-apps.md)。
- 若想深入瞭解 React Native，請查看[React Native 簡介](getting-started)。