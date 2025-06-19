---
title: How to Open a Pull Request
---

這些指示提供了逐步流程，幫助您設定機器以貢獻核心 React Native 儲存庫，並建立您的第一個拉取請求。

## 序章：準備工作

您需要一些工具和依賴項才能建置和開發 React Native。這些內容涵蓋在[環境設定](/docs/environment-setup)指南的「使用原生程式碼建置專案」部分中。

為了接受您的拉取請求，我們需要您提交[貢獻者授權協議 (CLA)](/contributing/contribution-license-agreement)。您只需執行一次此操作，即可參與 Meta 的任何開源專案。只需一分鐘即可完成，因此您可以在等待依賴項安裝時進行。

## 第一章：歡迎來到開源世界

### 1. 安裝 `git`

React Native 的原始碼託管在 GitHub 上。您可以透過 `git` 命令列程式與 git 版本控制互動。我們建議您遵循 [GitHub 的指示](https://help.github.com/articles/set-up-git/)在您的機器上設定 git。

### 2. 取得原始碼

雖然您可以在 [GitHub](https://github.com/facebook/react-native) 上瀏覽 React Native 的原始碼，但我們建議您在本地機器上設定一個分叉。

1. Go to https://github.com/facebook/react-native.
2. Click on "Fork" button on the upper right.
3. When asked, select your username as the host for this fork.

You will now have a fork of React Native on GitHub at https://github.com/your_username/react-native. Next, you will grab a copy of the source code for your local machine. Open a shell and type the following commands:

```bash
git clone https://github.com/facebook/react-native.git
cd react-native
git remote add fork https://github.com/your_username/react-native.git
```

:::note
如果上述內容對您來說是新知識，請不要害怕。您可以透過 macOS 和 Linux 上的 Terminal 應用程式或 Windows 上的 PowerShell 存取 shell。
:::

將建立一個新的 `react-native` 目錄，其中包含核心 React Native 儲存庫的內容。此目錄實際上是 React Native git 儲存庫的克隆。它設定了兩個遠端：

- `origin` 用於上游的 https://github.com/facebook/react-native 儲存庫
- `fork` 用於您自己 GitHub 帳戶上的 React Native 分叉。

### 3. 建立分支

我們建議在您的分叉中建立一個新分支以追蹤您的變更：

```bash
git checkout --branch my_feature_branch --track origin/main
```

## 第二章：實作您的變更

### 1. 安裝依賴項

React Native 是一個由 [Yarn Workspaces (Yarn Classic)](https://classic.yarnpkg.com/lang/en/docs/workspaces/) 管理的 JavaScript 單一儲存庫。

執行專案層級的安裝：

```sh
yarn
```

您還需要建置 `react-native-codegen` 套件一次：

```sh
yarn --cwd packages/react-native-codegen build
```

### 2. 修改程式碼

您現在可以使用您選擇的程式碼編輯器開啟專案。[Visual Studio Code](https://code.visualstudio.com/) 在 JavaScript 開發者中很受歡迎，如果您正在對 React Native 進行一般變更，我們推薦使用它。

IDE 專案設定：

- **VS Code**：開啟 `react-native.code-workspace` 檔案。這應該會開啟並推薦擴充功能，並正確設定 Flow 語言服務和其他編輯器設定。
- **Android Studio**：開啟儲存庫根目錄（包含 `.idea` 設定目錄）。
- **Xcode**：開啟 `packages/rn-tester/RNTesterPods.xcworkspace`。

### 3. 執行您的變更

套件 rn-tester 可用於執行並驗證您的變更。您可以在 [RNTester 說明文件](https://github.com/facebook/react-native/blob/main/packages/rn-tester/README.md) 中了解更多資訊。

### 4. 測試您的變更

請確保您的變更是正確的，且不會引入任何測試失敗。您可以在 [執行與撰寫測試](/contributing/how-to-run-and-write-tests) 中了解更多資訊。

### 5. 檢查程式碼風格

我們理解需要一些時間才能熟悉核心 React Native 儲存庫中每種語言所遵循的風格。開發者不應擔心細微的風格問題，因此我們盡可能使用工具來自動化重寫程式碼以符合慣例的流程。

例如，我們使用 [Prettier](https://prettier.io/) 來格式化 JavaScript 程式碼。這可以節省您的時間和精力，因為您可以透過其編輯器整合讓 Prettier 自動修復任何格式問題，或手動執行 `yarn run prettier`。

我們還使用 linter 來捕捉程式碼中可能存在的風格問題。您可以執行 `yarn run lint` 來檢查程式碼風格的狀態。

要了解更多關於編碼慣例的資訊，請參考 [編碼風格指南](/contributing/how-to-contribute-code#coding-style)。

### 6. 檢視您的變更

許多流行的編輯器以某種方式整合了版本控制。您也可以在命令列使用 `git status` 和 `git diff` 來追蹤已變更的內容。

## 第三章：提交您的變更

### 1. 提交您的變更

請確保使用 `git` 將您的變更加入版本控制：

```bash
git add <filename>
git commit -m <message>
```

您可以使用簡短的描述性句子作為提交訊息。

:::note
擔心寫不好 git 提交訊息？別擔心。當您的拉取請求被合併時，所有提交將被壓縮成單一提交。拉取請求的描述將用於填充此壓縮提交的訊息。
:::

本指南涵蓋了足夠的資訊來協助您完成首次貢獻。GitHub 提供了多種資源幫助您開始使用 git：

- [使用 Git](https://help.github.com/en/categories/using-git)
- [GitHub 流程](https://guides.github.com/introduction/flow/)

### 2. 將變更推送至 GitHub

一旦您的變更已提交至版本控制，您可以將其推送至 GitHub。

```bash
git push fork <my_feature_branch>
```

如果一切順利，您將看到一條鼓勵您開啟拉取請求的訊息：

```
remote:
remote: Create a pull request for 'your_feature_branch' on GitHub by visiting:
remote:      https://github.com/your_username/react-native/pull/new/your_feature_branch
remote:
```

訪問提供的 URL 以進行下一步。

### 3. 建立您的拉取請求

您快完成了！下一步是填寫拉取請求。使用一個描述性且不太長的標題。然後，確保填寫拉取請求範本提供的所有欄位：

- **摘要：** 使用此欄位說明您發送此拉取請求的動機。您要修復什麼？
- **[變更日誌](/contributing/changelogs-in-pull-requests)：** 透過提供簡短描述，幫助發布維護者在拉取請求被合併時撰寫發布說明。
- **測試計畫：** 讓審查者知道您如何測試變更。您是否考慮了任何邊緣案例？您遵循了哪些步驟來確保變更達到預期效果？請參閱 [什麼是測試計畫？](https://medium.com/@martinkonicek/what-is-a-test-plan-8bfc840ec171) 以了解更多資訊。

### 4. 審查並處理反饋

請密切關注 GitHub 上針對您的拉取請求所留下的任何評論與審核反饋。維護者們會盡力提供具建設性且可執行的反饋，以協助您的修改準備好被合併至 React Native 核心儲存庫中。