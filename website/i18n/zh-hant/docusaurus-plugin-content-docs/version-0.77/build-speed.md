---
id: build-speed
title: Speeding up your Build phase
---

建構 React Native 應用程式可能**相當耗時**，會耗費開發者數分鐘的時間。
隨著專案規模增長，特別是在擁有多名 React Native 開發者的大型組織中，這個問題會更加明顯。

為緩解此效能問題，本頁面將分享如何**提升建構速度**的建議。

## 開發期間僅建構單一 ABI (僅限 Android)

本地建構 Android 應用程式時，預設會建構所有 4 種[應用程式二進位介面 (ABI)](https://developer.android.com/ndk/guides/abis)：`armeabi-v7a`、`arm64-v8a`、`x86` 和 `x86_64`。

但若您僅需在本地建構並於模擬器或實體裝置測試，可能無需建構所有 ABI。

此舉可將**原生建構時間**縮短約 75%。

If you're using the React Native CLI, you can add the `--active-arch-only` flag to the `run-android` command. This flag will make sure the correct ABI is picked up from either the running emulator or the plugged in phone. To confirm that this approach is working fine, you'll see a message like `info Detected architectures arm64-v8a` on console.

```
$ yarn react-native run-android --active-arch-only

[ ... ]
info Running jetifier to migrate libraries to AndroidX. You can disable it using "--no-jetifier" flag.
Jetifier found 1037 file(s) to forward-jetify. Using 32 workers...
info JS server already running.
info Detected architectures arm64-v8a
info Installing the app...
```

此機制依賴於 `reactNativeArchitectures` Gradle 屬性。

因此，若直接透過命令列使用 Gradle 建構（不使用 CLI），可透過以下方式指定要建構的 ABI：

```
$ ./gradlew :app:assembleDebug -PreactNativeArchitectures=x86,x86_64
```

這在 CI 環境建構 Android 應用程式時特別有用，可透過矩陣平行化不同架構的建構流程。

If you wish, you can also override this value locally, using the `gradle.properties` file you have in the [top-level folder](https://github.com/facebook/react-native/blob/19cf70266eb8ca151aa0cc46ac4c09cb987b2ceb/template/android/gradle.properties#L30-L33) of your project:

```
# Use this property to specify which architecture you want to build.
# You can also override it from the CLI using
# ./gradlew <task> -PreactNativeArchitectures=x86_64
reactNativeArchitectures=armeabi-v7a,arm64-v8a,x86,x86_64
```

建構**正式版**應用程式時，請務必移除這些旗標，以確保產生的 apk/app bundle 支援所有 ABI，而非僅限日常開發使用的架構。

## 使用編譯器快取

若頻繁執行原生建構（C++ 或 Objective-C），使用**編譯器快取**可帶來顯著效益。

具體而言，您可使用兩類快取：本地編譯器快取與分散式編譯器快取。

### 本地快取

:::info
以下指令適用於 **Android 與 iOS**。
若僅建構 Android 應用程式，直接遵循即可。
若需建構 iOS 應用程式，請參閱下方 [XCode 專屬設定](#xcode-specific-setup)章節的說明。
:::

建議使用 [**ccache**](https://ccache.dev/) 來快取原生建構的編譯結果。
Ccache 透過封裝 C++ 編譯器運作，儲存編譯結果並在檢測到相同的中間編譯結果時跳過重新編譯。

Ccache 可透過多數作業系統的套件管理器安裝。在 macOS 上，可執行 `brew install ccache`。
或參閱[官方安裝指南](https://github.com/ccache/ccache/blob/master/doc/INSTALL.md)從原始碼編譯安裝。

You can then do two clean builds (e.g. on Android you can first run `yarn react-native run-android`, delete the `android/app/build` folder and run the first command once more). You will notice that the second build was way faster than the first one (it should take seconds rather than minutes).
While building, you can verify that `ccache` works correctly and check the cache hits/miss rate `ccache -s`

```
$ ccache -s
Summary:
  Hits:             196 /  3068 (6.39 %)
    Direct:           0 /  3068 (0.00 %)
    Preprocessed:   196 /  3068 (6.39 %)
  Misses:          2872
    Direct:        3068
    Preprocessed:  2872
  Uncacheable:        1
Primary storage:
  Hits:             196 /  6136 (3.19 %)
  Misses:          5940
  Cache size (GB): 0.60 / 20.00 (3.00 %)
```

請注意，`ccache` 會累積所有建置的統計數據。您可以使用 `ccache --zero-stats` 在建置前重置這些數據，以驗證快取命中率。

Should you need to wipe your cache, you can do so with `ccache --clear`

#### XCode 專屬設定

為確保 `ccache` 在 iOS 和 XCode 中正常運作，您需要在 `ios/Podfile` 中啟用 React Native 對 ccache 的支援。

在編輯器中開啟 `ios/Podfile` 並取消註解 `ccache_enabled` 這一行。

```ruby
  post_install do |installer|
    # https://github.com/facebook/react-native/blob/main/packages/react-native/scripts/react_native_pods.rb#L197-L202
    react_native_post_install(
      installer,
      config[:reactNativePath],
      :mac_catalyst_enabled => false,
      # TODO: Uncomment the line below
      :ccache_enabled => true
    )
  end
```

#### 在 CI 上使用此方法

Ccache 在 macOS 上使用 `/Users/$USER/Library/Caches/ccache` 資料夾來儲存快取。
因此，您也可以在 CI 上保存和恢復對應的資料夾以加速建置。

然而，有幾點需要注意：

1. 在 CI 上，建議進行完整的乾淨建置，以避免快取污染問題。如果您遵循前一段提到的方法，應該能夠在 4 種不同的 ABI 上平行化原生建置，這樣在 CI 上很可能就不需要 `ccache`。

2. `ccache` 依賴時間戳記來計算快取命中。這在 CI 上效果不佳，因為每次 CI 運行時都會重新下載檔案。為解決此問題，您需要使用 `compiler_check content` 選項，該選項改為依賴[檔案內容的雜湊值](https://ccache.dev/manual/4.3.html)。

### 分散式快取

與本地快取類似，您可能會考慮對原生建置使用分散式快取。
這對於進行頻繁原生建置的大型組織特別有用。

我們建議使用 [sccache](https://github.com/mozilla/sccache) 來實現這一點。
有關如何設定和使用此工具的說明，請參考 sccache 的[分散式編譯快速入門](https://github.com/mozilla/sccache/blob/main/docs/DistributedQuickstart.md)。