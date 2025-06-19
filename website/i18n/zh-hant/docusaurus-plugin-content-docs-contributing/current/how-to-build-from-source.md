---
title: How to Build from Source
---

若您想開發新功能/修復錯誤、測試尚未發佈的最新功能，或維護包含無法合併至核心的修補程式的自訂分支，您需要從原始碼建構 React Native。

## Android

### 必要條件

要從原始碼建構，您需要安裝 Android SDK。若您已遵循[設定開發環境](/docs/environment-setup)指南，應該已完成設定。

無需安裝其他工具如特定版本的 NDK 或 CMake，因為 Android SDK 會**自動下載**建構原始碼所需的一切。

### 將專案指向 nightly 版本

要使用 React Native 的最新修復和功能，您可以透過以下方式將專案更新至 nightly 版本：

```
yarn add react-native@nightly
```

這會將您的專案更新為使用每晚發佈、包含最新變更的 React Native nightly 版本。

### 將專案更新為從原始碼建構

無論是穩定版本或 nightly 版本，您都會使用**預編譯**的成品。若您想改為從原始碼建構，以便直接測試對框架的變更，您需要編輯 `android/settings.gradle` 檔案如下：

```diff
  // ...
  include ':app'
  includeBuild('../node_modules/@react-native/gradle-plugin')
  
+ includeBuild('../node_modules/react-native') {
+     dependencySubstitution {
+         substitute(module("com.facebook.react:react-android")).using(project(":packages:react-native:ReactAndroid"))
+         substitute(module("com.facebook.react:react-native")).using(project(":packages:react-native:ReactAndroid"))
+         substitute(module("com.facebook.react:hermes-android")).using(project(":packages:react-native:ReactAndroid:hermes-engine"))
+         substitute(module("com.facebook.react:hermes-engine")).using(project(":packages:react-native:ReactAndroid:hermes-engine"))
+     }
+ }
```

### 補充說明

從原始碼建構可能耗時較長，尤其是首次建構時，因為需要下載約 200 MB 的成品並編譯原生程式碼。

每次從您的儲存庫更新 `react-native` 版本時，建構目錄可能會被刪除，所有檔案會重新下載。
為避免此情況，您可以編輯 `~/.gradle/init.gradle` 檔案來變更建構目錄路徑：

```groovy
gradle.projectsLoaded {
    rootProject.allprojects {
        buildDir = "/path/to/build/directory/${rootProject.name}/${project.name}"
    }
}
```

## 理由說明

建議的 React Native 使用方式是始終更新至最新版本。我們對舊版提供的支援[詳見支援政策](https://github.com/reactwg/react-native-releases/#releases-support-policy)。

從原始碼建構的方式應用於在提交 pull request 至 React Native 前進行端到端測試，我們不鼓勵長期使用此方式。特別是分叉 React Native 或將設定改為始終從原始碼建構，會導致專案更難更新，通常也會帶來較差的開發體驗。