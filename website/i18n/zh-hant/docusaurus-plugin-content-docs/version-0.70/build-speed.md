---
id: build-speed
title: Speeding up your Build phase
---

建構 React Native 應用程式可能**相當耗時**，會佔用開發者數分鐘的時間。隨著專案規模擴大，特別是在擁有多位 React Native 開發者的大型組織中，這個問題會更加明顯。

隨著[新 React Native 架構](the-new-architecture/landing-page.md)的推出，此問題變得更加關鍵，因為除了 iOS 和 Android 平台原本需要的原生程式碼外，您可能還需要使用 Android NDK 編譯專案中的部分 C++ 程式碼。

為減輕效能影響，本頁面將分享一些**提升建構速度**的建議。

## 開發期間僅建構單一 ABI（僅限 Android）

在本地建構 Android 應用程式時，預設會建構所有 4 種[應用程式二進位介面 (ABI)](https://developer.android.com/ndk/guides/abis)：`armeabi-v7a`、`arm64-v8a`、`x86` 和 `x86_64`。

但若您僅在本地建構並於模擬器或實體裝置上測試，可能無需建構所有 ABI。

此舉可將建構時間**縮短約 75%**。

If you're using the React Native CLI, you can use the `--active-arch-only` flag together with the `run-android` command.
This flag will make sure the correct ABI is picked up from either the running emulator or the plugged in phone.
To confirm that this approach is working fine, you'll see a message like `info Detected architectures arm64-v8a` on console.

```
$ yarn react-native run-android --active-arch-only

[ ... ]
info Running jetifier to migrate libraries to AndroidX. You can disable it using "--no-jetifier" flag.
Jetifier found 1037 file(s) to forward-jetify. Using 32 workers...
info JS server already running.
info Detected architectures arm64-v8a
info Installing the app...
```

此機制依賴於 Gradle 屬性 `reactNativeArchitectures`。

因此，若直接透過命令列使用 Gradle 建構（不透過 CLI），可透過以下方式指定要建構的 ABI：

```
$ ./gradlew :app:assembleDebug -PreactNativeArchitectures=x86,x86_64
```

此方法適用於在 CI 環境建構 Android 應用時，透過矩陣平行化處理不同架構的建構作業。

If you wish, you can also override this value locally, using the `gradle.properties` file you have in the [top-level folder](https://github.com/facebook/react-native/blob/19cf70266eb8ca151aa0cc46ac4c09cb987b2ceb/template/android/gradle.properties#L30-L33) of your project:

```
# Use this property to specify which architecture you want to build.
# You can also override it from the CLI using
# ./gradlew <task> -PreactNativeArchitectures=x86_64
reactNativeArchitectures=armeabi-v7a,arm64-v8a,x86,x86_64
```

請注意，建構**正式版**應用時務必移除這些參數，以確保產出的 apk/app bundle 能支援所有 ABI，而非僅限日常開發使用的單一架構。

## 使用編譯器快取

若頻繁執行原生建構（C++ 或 Objective-C），使用**編譯器快取**可顯著提升效率。

具體可分為兩類：本地編譯器快取與分散式編譯器快取。

### 本地快取

:::info
以下說明適用於**Android 與 iOS**。若僅建構 Android 應用，直接依指示操作即可；若需建構 iOS 應用，請參閱下方[XCode 特定設定](#xcode-specific-setup)章節。
:::

建議使用 [**ccache**](https://ccache.dev/) 快取原生建構的編譯結果。其原理是透過封裝 C++ 編譯器，儲存編譯結果，當偵測到相同的中間編譯結果時直接跳過重複編譯。

安裝方式請參閱[官方安裝指南](https://github.com/ccache/ccache/blob/master/doc/INSTALL.md)。

在 macOS 上可透過 `brew install ccache` 安裝。安裝完成後，可透過以下設定快取 NDK 編譯結果：

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

你可以使用 `which` 命令驗證是否生效：

```
$ which gcc
/usr/local/bin/gcc
```

若結果顯示 `/usr/local/bin/gcc`，則表示你實際調用的是 `ccache`，它會包裹 `gcc` 的調用。

:::caution
請注意，此 `ccache` 設定會影響你機器上所有的編譯行為，不僅限於 React Native 相關編譯。使用時需自行承擔風險。若其他軟體安裝/編譯失敗，可能是此原因所致。若發生此情況，可通過以下命令移除創建的符號連結以恢復原始狀態：

```
unlink /usr/local/bin/gcc
unlink /usr/local/bin/g++
unlink /usr/local/bin/cc
unlink /usr/local/bin/c++
unlink /usr/local/bin/clang
unlink /usr/local/bin/clang++
```

即可恢復使用默認編譯器。
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

注意 `ccache` 會累積所有建置的統計數據。可使用 `ccache --zero-stats` 在建置前重置統計以驗證緩存命中率。

Should you need to wipe your cache, you can do so with `ccache --clear`

#### XCode 專屬設定

要確保 `ccache` 在 iOS 和 XCode 環境正常運作，需額外執行以下步驟：

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

2. 需要配置 ccache 允許一定程度的寬鬆和緩存行為，使其能在 Xcode 編譯時記錄緩存命中。若通過環境變量配置，以下 ccache 配置參數與標準值不同：

```bash
export CCACHE_SLOPPINESS=clang_index_store,file_stat_matches,include_file_ctime,include_file_mtime,ivfsoverlay,pch_defines,modules,system_headers,time_macros
export CCACHE_FILECLONE=true
export CCACHE_DEPEND=true
export CCACHE_INODECACHE=true
```

相同配置也可寫入 `ccache.conf` 文件或通過 ccache 提供的其他機制實現。詳情參見 [官方 ccache 手冊](https://ccache.dev/manual/4.3.html)。

#### 在 CI 上使用此方法

Ccache 在 macOS 上使用 `/Users/$USER/Library/Caches/ccache` 目錄存儲緩存。因此你也可以在 CI 上保存和恢復該目錄以加速建置。

但需注意以下事項：

1. 在 CI 環境中，我們建議進行完整的乾淨建置，以避免快取污染問題。若您遵循前文提及的方法，應該能夠將原生建置平行化到 4 種不同的 ABI 架構上，如此一來很可能就不需要在 CI 上使用 `ccache`。

2. `ccache` 依賴時間戳記來計算快取命中率。這在 CI 環境中效果不佳，因為每次 CI 執行時檔案都會重新下載。為解決此問題，您需要使用 `compiler_check content` 選項，該選項改為[透過檔案內容的雜湊值](https://ccache.dev/manual/4.3.html)來判斷。

### 分散式快取

與本地快取類似，您可能會想考慮為原生建置使用分散式快取。
這對於經常進行原生建置的大型組織特別有用。

我們推薦使用 [sccache](https://github.com/mozilla/sccache) 來實現此目的。
關於如何設定和使用此工具，請參閱 sccache 的[分散式編譯快速入門指南](https://github.com/mozilla/sccache/blob/main/docs/DistributedQuickstart.md)。