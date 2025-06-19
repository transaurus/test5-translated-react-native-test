---
id: build-speed
title: Speeding up your Build phase
---

建構 React Native 應用程式可能**相當耗時**，會佔用開發者數分鐘的時間。隨著專案規模增長，特別是在擁有眾多 React Native 開發者的大型組織中，這個問題會更加明顯。

為緩解此效能問題，本頁面提供了一些關於如何**提升建構速度**的建議。

:::info

請注意，這些建議屬於進階功能，需要對原生建構工具運作方式有一定程度的理解。

:::

## 開發期間僅建構單一 ABI (僅限 Android)

當你在本地建構 Android 應用程式時，預設會建構所有 4 種[應用程式二進位介面 (ABI)](https://developer.android.com/ndk/guides/abis)：`armeabi-v7a`、`arm64-v8a`、`x86` 和 `x86_64`。

但如果你只是在本地建構並於模擬器或實體裝置上測試，很可能不需要建構所有這些 ABI。

這應該能將你的**原生建構時間**減少約 75%。

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

因此，如果你直接透過 Gradle 從指令列建構且不使用 CLI，可以如下指定要建構的 ABI：

```
$ ./gradlew :app:assembleDebug -PreactNativeArchitectures=x86,x86_64
```

這在你想於 CI 上建構 Android 應用程式並使用矩陣平行化不同架構的建構時特別有用。

If you wish, you can also override this value locally, using the `gradle.properties` file you have in the [top-level folder](https://github.com/facebook/react-native/blob/19cf70266eb8ca151aa0cc46ac4c09cb987b2ceb/template/android/gradle.properties#L30-L33) of your project:

```
# Use this property to specify which architecture you want to build.
# You can also override it from the CLI using
# ./gradlew <task> -PreactNativeArchitectures=x86_64
reactNativeArchitectures=armeabi-v7a,arm64-v8a,x86,x86_64
```

當你建構應用程式的**正式版本**時，請記得移除這些旗標，因為你需要建構支援所有 ABI 的 apk/app bundle，而不僅是日常開發工作流程中使用的單一 ABI。

## 啟用設定快取 (僅限 Android)

自 React Native 0.79 起，你也可以啟用 Gradle 設定快取。

當你執行 `yarn android` 進行 Android 建構時，會執行一個由兩個階段組成的 Gradle 建構（[來源](https://docs.gradle.org/current/userguide/build_lifecycle.html)）：

- 設定階段：評估所有 `.gradle` 檔案。
- 執行階段：實際執行任務，編譯 Java/Kotlin 程式碼等。

現在你可以啟用設定快取，這將讓你在後續建構中跳過設定階段。

這對於頻繁變更原生程式碼的情況特別有益，因為它能提升建構速度。

例如，這裡你可以看到在變更原生程式碼後，重新建構 RN-Tester 的速度提升：

![gradle 設定快取](/docs/assets/gradle-config-caching.gif)

你可以透過在 `android/gradle.properties` 檔案中加入以下行來啟用 Gradle 設定快取：

```
org.gradle.configuration-cache=true
```

更多關於設定快取的資源，請參考[官方 Gradle 文件](https://docs.gradle.org/current/userguide/configuration_cache.html)。

## 使用編譯器快取

如果您頻繁執行原生建置（無論是 C++ 或 Objective-C），使用**編譯器快取**可能會帶來顯著效益。

具體而言，您可以選擇兩種快取類型：本地編譯器快取與分散式編譯器快取。

### 本地快取

:::info
以下說明適用於**Android 與 iOS**。
若您僅建置 Android 應用程式，可直接遵循本節指引。
若需同時建置 iOS 應用程式，請參閱後續的[XCode 專屬設定](#xcode-specific-setup)章節。
:::

我們建議使用 [**ccache**](https://ccache.dev/) 來快取原生建置的編譯過程。
Ccache 透過封裝 C++ 編譯器運作，儲存編譯結果，並在偵測到相同的中間編譯結果時直接跳過重新編譯。

Ccache 可透過多數作業系統的套件管理工具取得。在 macOS 上，可透過 `brew install ccache` 安裝。
或參照[官方安裝指南](https://github.com/ccache/ccache/blob/master/doc/INSTALL.md)從原始碼編譯安裝。

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

請注意 `ccache` 會累積所有建置階段的統計數據。可使用 `ccache --zero-stats` 在建置前重置數據以準確驗證快取命中率。

若需清除快取，請執行 `ccache --clear`

#### XCode 專屬設定

要確保 `ccache` 在 iOS 與 XCode 環境正常運作，需於 `ios/Podfile` 中啟用 React Native 對 ccache 的支援。

開啟 `ios/Podfile` 檔案，取消註解 `ccache_enabled` 參數。

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

#### 在 CI 環境使用此方案

在 macOS 系統中，Ccache 預設將快取儲存於 `/Users/$USER/Library/Caches/ccache` 目錄。
因此您可在 CI 環境中保存與還原此目錄以加速建置流程。

但需注意以下事項：

1. 在 CI 環境建議執行完整乾淨建置，避免快取污染問題。若採用前文提到的多 ABI 平行建置方案，通常無需在 CI 使用 `ccache`。

2. `ccache` 依賴時間戳記計算快取命中，此機制在 CI 環境可能失效（因檔案會在每次 CI 執行時重新下載）。解決方案是啟用 `compiler_check content` 選項，改為[基於檔案內容雜湊值](https://ccache.dev/manual/4.3.html)進行判斷。

### 分散式快取

類似本地快取概念，對於頻繁執行原生建置的大型組織，可考慮採用分散式快取方案。

我們推薦使用 [sccache](https://github.com/mozilla/sccache) 實現此需求。
具體設定方式請參閱 sccache 的[分散式編譯快速入門指南](https://github.com/mozilla/sccache/blob/main/docs/DistributedQuickstart.md)。