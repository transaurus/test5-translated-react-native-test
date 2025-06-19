---
title: Bots Reference
---

## pull-bot

此拉取請求檢查機器人會在每次創建拉取請求時執行基本檢查。如果它無法在描述中找到測試計劃或變更日誌，或者發現拉取請求並非針對 `main` 分支開啟，它可能會在拉取請求上留下評論。此機器人使用 [Danger](https://danger.systems)，其配置可在核心倉庫的 [`dangerfile.js`](https://github.com/facebook/react-native/blob/main/packages/react-native-bots/dangerfile.js) 中找到。

## analysis-bot

代碼分析機器人會在每次提交添加到拉取請求時，收集來自 Prettier、eslint 和 Flow 等工具的反饋。如果這些工具發現代碼問題，機器人會將其作為內聯審查評論添加到拉取請求中。其配置可在核心倉庫的 [`analyze_code.sh`](https://github.com/facebook/react-native/blob/main/scripts/circleci/analyze_code.sh) 文件中找到。

## label-actions

一個基於標籤對問題或拉取請求執行操作的機器人。配置於 [`.github/workflows/on-issue-labeled.yml`](https://github.com/facebook/react-native/blob/main/.github/workflows/on-issue-labeled.yml)。

## github-actions

一個執行 GitHub 工作流程中定義操作的機器人。工作流程配置於 [`.github/workflows`](https://github.com/facebook/react-native/tree/main/.github/workflows)。

## facebook-github-bot

Facebook GitHub 機器人用於 Meta 的多個開源項目。在 React Native 中，您最有可能在拉取請求成功導入 Facebook 內部源代碼控制後，看到它將合併提交推送到 `main` 分支時遇到它。它還會通知作者是否缺少貢獻者許可協議。

## react-native-bot

React Native 機器人是一個幫助我們自動化本 wiki 中描述的幾個流程的工具。配置於 [`hramos/react-native-bot`](https://github.com/hramos/react-native-bot)。