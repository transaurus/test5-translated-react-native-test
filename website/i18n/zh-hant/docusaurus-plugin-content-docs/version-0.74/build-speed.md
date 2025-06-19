---
id: build-speed
title: Speeding up your Build phase
---

建構 React Native 應用程式可能**相當耗時**，會佔用開發者數分鐘的時間。隨著專案規模增長，特別是在擁有多位 React Native 開發者的大型組織中，此問題會更加明顯。

為緩解此效能問題，本頁面提供數項**加速建構時間**的建議。

## 開發期間僅建構單一 ABI (僅限 Android)

本地建構 Android 應用程式時，預設會建構所有 4 種[應用程式二進位介面 (ABI)](https://developer.android.com/ndk/guides/abis)：`armeabi-v7a`、`arm64-v8a`、`x86` 和 `x86_64`。

但若僅在本地建構並於模擬器或實體裝置測試，實際上無需建構全部 ABI。

此做法可將**原生建構時間**縮短約 75%。

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

此方式適用於在 CI 上建構 Android 應用程式時，透過矩陣平行化處理不同架構的建構作業。

If you wish, you can also override this value locally, using the `gradle.properties` file you have in the [top-level folder](https://github.com/facebook/react-native/blob/19cf70266eb8ca151aa0cc46ac4c09cb987b2ceb/template/android/gradle.properties#L30-L33) of your project:

```
# Use this property to specify which architecture you want to build.
# You can also override it from the CLI using
# ./gradlew <task> -PreactNativeArchitectures=x86_64
reactNativeArchitectures=armeabi-v7a,arm64-v8a,x86,x86_64
```

建構**正式版**應用程式時，請務必移除這些旗標，以確保產出的 apk/app bundle 能支援所有 ABI，而非僅限日常開發使用的單一架構。

## 使用編譯器快取

若需頻繁執行原生建構（C++ 或 Objective-C），使用**編譯器快取**將帶來顯著效益。

具體而言，可採用兩類快取：本地編譯器快取與分散式編譯器快取。

### 本地快取

:::info
以下說明適用於**Android 與 iOS**。
若僅建構 Android 應用程式，直接遵循即可。
若需建構 iOS 應用程式，請參閱下方 [XCode 專屬設定](#xcode-specific-setup)章節的補充說明。
:::

建議使用 [**ccache**](https://ccache.dev/) 快取原生建構的編譯結果。Ccache 透過包裝 C++ 編譯器運作，儲存編譯結果並在偵測到相同的中間編譯結果時直接跳過編譯步驟。

多數作業系統的套件管理工具皆提供 Ccache。在 macOS 上可透過 `brew install ccache` 安裝，或參照[官方安裝指南](https://github.com/ccache/ccache/blob/master/doc/INSTALL.md)從原始碼編譯安裝。

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

請注意，`ccache` 會累積所有建置過程的統計數據。您可以使用 `ccache --zero-stats` 在建置前重置這些數據，以驗證快取命中率。

Should you need to wipe your cache, you can do so with `ccache --clear`

#### XCode 專屬設定

為確保 `ccache` 在 iOS 和 XCode 環境中正常運作，您需要在 `ios/Podfile` 中啟用 React Native 對 ccache 的支援。

使用編輯器開啟 `ios/Podfile` 並取消註解 `ccache_enabled` 這一行。

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

#### 在 CI 環境使用此方法

Ccache 在 macOS 上會將快取儲存於 `/Users/$USER/Library/Caches/ccache` 資料夾。
因此您也可以在 CI 環境中保存與還原對應的資料夾來加速建置。

但需注意以下幾點：

1. 在 CI 環境中，建議執行完整的乾淨建置，以避免快取污染問題。若您採用前文提及的方法，應能將原生建置平行分散到 4 種不同的 ABI 上執行，如此一來在 CI 環境中很可能就不需要 `ccache`。

2. `ccache` 依賴時間戳記來計算快取命中，這在 CI 環境中效果不佳，因為每次 CI 執行時檔案都會重新下載。為解決此問題，您需要使用 `compiler_check content` 選項，該選項改為[透過檔案內容的雜湊值](https://ccache.dev/manual/4.3.html)來判斷。

### 分散式快取

與本地快取類似，您可能會想考慮為原生建置使用分散式快取。
這對於需要頻繁執行原生建置的大型組織特別有用。

我們推薦使用 [sccache](https://github.com/mozilla/sccache) 來實現此需求。
關於如何設定與使用此工具，請參閱 sccache 的[分散式編譯快速入門指南](https://github.com/mozilla/sccache/blob/main/docs/DistributedQuickstart.md)。