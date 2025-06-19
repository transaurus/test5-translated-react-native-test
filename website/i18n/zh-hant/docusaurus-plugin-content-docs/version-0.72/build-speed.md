---
id: build-speed
title: Speeding up your Build phase
---

建置 React Native 應用程式可能**相當耗時**，會佔用開發者數分鐘的時間。
隨著專案規模擴大，特別是在擁有多位 React Native 開發者的大型組織中，這個問題會更加明顯。

為緩解效能瓶頸，本頁面提供數項**加速建置流程**的實用建議。

## 開發階段僅建置單一 ABI（僅限 Android）

本地建置 Android 應用時，預設會同時建置所有 4 種[應用程式二進位介面 (ABI)](https://developer.android.com/ndk/guides/abis)：`armeabi-v7a`、`arm64-v8a`、`x86` 與 `x86_64`。

但若僅需在模擬器或實體裝置測試，通常無需建置全部架構。

此調整可降低**原生建置時間**達約 75%。

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

此機制依賴 `reactNativeArchitectures` Gradle 屬性實現。

因此若直接透過 Gradle 指令建置（不使用 CLI），可透過以下方式指定目標 ABI：

```
$ ./gradlew :app:assembleDebug -PreactNativeArchitectures=x86,x86_64
```

此方式特別適用於 CI 環境，可透過建置矩陣平行處理不同架構。

If you wish, you can also override this value locally, using the `gradle.properties` file you have in the [top-level folder](https://github.com/facebook/react-native/blob/19cf70266eb8ca151aa0cc46ac4c09cb987b2ceb/template/android/gradle.properties#L30-L33) of your project:

```
# Use this property to specify which architecture you want to build.
# You can also override it from the CLI using
# ./gradlew <task> -PreactNativeArchitectures=x86_64
reactNativeArchitectures=armeabi-v7a,arm64-v8a,x86,x86_64
```

請注意：發佈**正式版本**時，務必移除這些參數，以確保 APK/app bundle 支援全架構，而非僅限開發環境使用的單一 ABI。

## 使用編譯器快取

若需頻繁執行原生建置（C++ 或 Objective-C），採用**編譯器快取**可顯著提升效率。

具體可分為兩類：本地編譯器快取與分散式編譯器快取。

### 本地快取

:::info
以下教學適用於**Android 與 iOS** 專案。
若僅建置 Android 應用，直接依步驟操作即可。
若需建置 iOS 應用，請額外參照下方 [XCode 專屬設定](#xcode-specific-setup)章節。
:::

建議使用 [**ccache**](https://ccache.dev/) 快取原生建置編譯結果。其原理是透過包裝 C++ 編譯器，儲存編譯產出物，當偵測到相同的中間編譯結果時直接跳過重複編譯。

安裝方式請參照[官方文件](https://github.com/ccache/ccache/blob/master/doc/INSTALL.md)。

macOS 環境可透過 `brew install ccache` 安裝。完成後請依以下步驟設定 NDK 編譯快取：

```
ln -s $(which ccache) /usr/local/bin/gcc
ln -s $(which ccache) /usr/local/bin/g++
ln -s $(which ccache) /usr/local/bin/cc
ln -s $(which ccache) /usr/local/bin/c++
ln -s $(which ccache) /usr/local/bin/clang
ln -s $(which ccache) /usr/local/bin/clang++
```

This will create symbolic links to `ccache` inside the `/usr/local/bin/` which are called `gcc`, `g++`, and so on.

This works as long as `/usr/local/bin/` comes first than `/usr/bin/` inside your `$PATH` variable, which is the default.

你可以使用 `which` 指令來驗證是否生效：

```
$ which gcc
/usr/local/bin/gcc
```

若結果顯示為 `/usr/local/bin/gcc`，則代表你實際呼叫的是 `ccache`，它會包裹 `gcc` 的呼叫。

:::caution
請注意，此 `ccache` 設定會影響你電腦上所有的編譯行為，不僅限於 React Native 相關的編譯。請自行承擔使用風險。若你發現其他軟體無法安裝或編譯，可能是此設定導致。若發生此情況，你可以移除先前建立的符號連結來恢復原始狀態：

```
unlink /usr/local/bin/gcc
unlink /usr/local/bin/g++
unlink /usr/local/bin/cc
unlink /usr/local/bin/c++
unlink /usr/local/bin/clang
unlink /usr/local/bin/clang++
```

這些指令會移除符號連結，讓系統恢復使用預設的編譯器。
:::

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

注意 `ccache` 會累積所有建置的統計數據。你可以使用 `ccache --zero-stats` 在建置前重置數據，以便準確驗證快取命中率。

若需清除快取，可執行 `ccache --clear`

#### XCode 專屬設定

要讓 `ccache` 在 iOS 和 XCode 環境正常運作，需額外執行以下步驟：

1. You must alter the way Xcode and `xcodebuild` call for the compiler command. By default they use _fully specified paths_ to the compiler binaries, so the symbolic links installed in `/usr/local/bin` will not be used. You may configure Xcode to use _relative_ names for the compilers using either of these two options:

- environment variables prefixed on the command line if you use a direct command line: `CLANG=clang CLANGPLUSPLUS=clang++ LD=clang LDPLUSPLUS=clang++ xcodebuild <rest of xcodebuild command line>`
- A `post_install` section in your `ios/Podfile` that alters the compiler in your Xcode workspace during the `pod install` step:

```ruby
  post_install do |installer|
    react_native_post_install(installer)

    # ...possibly other post_install items here

    installer.pods_project.targets.each do |target|
      target.build_configurations.each do |config|
        # Using the un-qualified names means you can swap in different implementations, for example ccache
        config.build_settings["CC"] = "clang"
        config.build_settings["LD"] = "clang"
        config.build_settings["CXX"] = "clang++"
        config.build_settings["LDPLUSPLUS"] = "clang++"
      end
    end

    __apply_Xcode_12_5_M1_post_install_workaround(installer)
  end
```

2. 需要設定 ccache 配置，允許一定程度的寬鬆行為和快取機制，使 ccache 能在 Xcode 編譯時正確記錄快取命中。若透過環境變數設定，以下為不同於標準設定的 ccache 配置參數：

```bash
export CCACHE_SLOPPINESS=clang_index_store,file_stat_matches,include_file_ctime,include_file_mtime,ivfsoverlay,pch_defines,modules,system_headers,time_macros
export CCACHE_FILECLONE=true
export CCACHE_DEPEND=true
export CCACHE_INODECACHE=true
```

相同設定也可透過 `ccache.conf` 檔案或其他 ccache 支援的方式配置。更多細節請參考 [ccache 官方手冊](https://ccache.dev/manual/4.3.html)。

#### 在 CI 上使用此方法

Ccache 在 macOS 上會使用 `/Users/$USER/Library/Caches/ccache` 資料夾儲存快取。
因此你也可以在 CI 環境儲存與恢復此資料夾來加速建置。

但需注意以下事項：

1. 在 CI 環境建議執行完整乾淨建置，避免快取污染問題。若採用前文提到的平行建置方法，你應該能將原生建置分散到 4 種不同 ABI 上執行，此時在 CI 上可能就不需要 `ccache`。

2. `ccache` 依賴時間戳記來計算快取命中，這在 CI 環境效果不佳，因為每次執行時檔案都會重新下載。為解決此問題，需使用 `compiler_check content` 選項，改為[對檔案內容進行雜湊計算](https://ccache.dev/manual/4.3.html)。

### 分散式快取

與本地緩存類似，您可能會考慮為原生建構使用分散式緩存。
這對於頻繁進行原生建構的大型組織尤其有用。

我們建議使用 [sccache](https://github.com/mozilla/sccache) 來實現此目的。
有關如何設置和使用此工具的說明，請參考 sccache 的 [分散式編譯快速入門](https://github.com/mozilla/sccache/blob/main/docs/DistributedQuickstart.md)。