---
id: upgrading
title: Upgrading to new versions
---

升級至新版本的 React Native 將讓您獲得更多 API、視圖元件、開發者工具和其他優化功能。升級過程需要少量操作，但我們會盡量使其直觀簡便。

## Expo 專案

Upgrading your Expo project to a new version of React Native requires updating the `react-native`, `react`, and `expo` package versions in your `package.json` file. Expo provides an `upgrade` command to handle upgrading these and any other known dependencies for you. See the [Upgrading Expo SDK Walkthrough](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/) for up-to-date information about upgrading your project.

## React Native 專案

由於標準 React Native 專案本質上由 Android 專案、iOS 專案和 JavaScript 專案組成，升級過程較為複雜。目前有兩種升級方式：使用 [React Native CLI](https://github.com/react-native-community/cli) 或透過 [Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) 手動操作。

### React Native CLI

The [React Native CLI](https://github.com/react-native-community/cli) comes with `upgrade` command that provides a one-step operation to upgrade the source files with a minimum of conflicts, it internally uses [rn-diff-purge](https://github.com/react-native-community/rn-diff-purge) project to find out which files need to be created, removed or modified.

#### 1. 執行 `upgrade` 指令

> The `upgrade` command works on top of Git by using `git apply` with 3-way merge, therefore it's required to use Git in order for this to work, if you don't use Git but still want to use this solution then you can check out how to do it in the [Troubleshooting](#i-want-to-upgrade-with-react-native-cli-but-i-dont-use-git) section.

執行以下指令開始升級至最新版本：

```shell
npx react-native upgrade
```

可透過參數指定 React Native 版本，例如升級至 `0.61.0-rc.0`：

```shell
npx react-native upgrade 0.61.0-rc.0
```

專案升級過程使用 `git apply` 三方合併，完成後可能需要解決部分衝突。

#### 2. 解決衝突

衝突檔案會包含明確標示變更來源的分隔符號，例如：

```
13B07F951A680F5B00A75B9A /* Release */ = {
  isa = XCBuildConfiguration;
  buildSettings = {
    ASSETCATALOG_COMPILER_APPICON_NAME = AppIcon;
<<<<<<< ours
    CODE_SIGN_IDENTITY = "iPhone Developer";
    FRAMEWORK_SEARCH_PATHS = (
      "$(inherited)",
      "$(PROJECT_DIR)/HockeySDK.embeddedframework",
      "$(PROJECT_DIR)/HockeySDK-iOS/HockeySDK.embeddedframework",
    );
=======
    CURRENT_PROJECT_VERSION = 1;
>>>>>>> theirs
    HEADER_SEARCH_PATHS = (
      "$(inherited)",
      /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include,
      "$(SRCROOT)/../node_modules/react-native/React/**",
      "$(SRCROOT)/../node_modules/react-native-code-push/ios/CodePush/**",
    );
```

可將 "ours" 視為「您的團隊版本」，"theirs" 視為「React Native 開發團隊版本」。

### Upgrade Helper

The [Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) is a web tool to help you out when upgrading your apps by providing the full set of changes happening between any two versions. It also shows comments on specific files to help understanding why that change is needed.

#### 1. 選擇版本

首先需選擇原始版本與目標版本（預設為最新主要版本），點擊「Show me how to upgrade」按鈕後顯示升級指引。

💡 主要版本升級時，頂部會顯示「實用內容」區塊，包含升級輔助連結。

#### 2. 升級依賴項

首個顯示的檔案是 `package.json`，建議先更新其中標示的依賴項。例如若顯示需變更 `react-native` 和 `react`，可執行 `yarn add` 安裝：

```shell
# {{VERSION}} and {{REACT_VERSION}} are the release versions showing in the diff
yarn add react-native@{{VERSION}}
yarn add react@{{REACT_VERSION}}
```

#### 3. 升級專案檔案

新版本可能包含執行 `npx react-native init` 時生成的其他檔案更新，這些檔案會列在 Upgrade Helper 頁面的 `package.json` 之後。如果沒有其他變更，您只需重新建置專案即可繼續開發。

若有變更，您可以手動從頁面中的變更複製貼上更新，或透過執行以下 React Native CLI 升級指令自動處理：

```shell
npx react-native upgrade
```

此指令會將您的檔案與最新模板比對，並執行以下操作：

- 若模板中有新檔案，則建立該檔案。
- 若模板中的檔案與您的檔案完全相同，則跳過。
- 若您的專案檔案與模板不同，系統會提示您選擇保留原檔案或覆寫為模板版本。

> 部分升級無法透過 React Native CLI 自動完成（例如從 `0.28` 升級至 `0.29` 或 `0.56` 升級至 `0.57`），需手動處理。升級時請務必查閱[版本發布說明](https://github.com/facebook/react-native/releases)，以確認專案所需的特殊調整。

### 疑難排解

#### 我想使用 React Native CLI 升級，但未使用 Git

即使您的專案未使用 Git 版本控制系統（可能使用 Mercurial、SVN 或未使用任何系統），仍需[安裝 Git](https://git-scm.com/downloads) 才能執行 `npx react-native upgrade`，且 Git 必須位於 `PATH` 環境變數中。若專案未初始化 Git 儲存庫，請先執行以下指令：

```shell
git init # Initialize a Git repository
git add . # Stage all the current files
git commit -m "Upgrade react-native" # Save the current files in a commit
```

升級完成後可刪除 `.git` 目錄。

#### 已完成所有變更，但應用程式仍使用舊版本

此類問題通常與快取有關，建議安裝 [react-native-clean-project](https://github.com/pmadruga/react-native-clean-project) 清除專案所有快取後重新執行。