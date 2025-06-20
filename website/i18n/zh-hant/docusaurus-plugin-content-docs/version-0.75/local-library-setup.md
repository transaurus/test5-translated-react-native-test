---
id: local-library-setup
title: Local libraries setup
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

本地函式庫是指包含視圖或模組且僅限於應用程式內部使用、未發佈至註冊表的程式庫。與傳統視圖和模組設置不同，本地函式庫與應用程式的原生代碼是解耦的。

本地函式庫創建於 `android/` 和 `ios/` 目錄之外，並利用自動連結（autolinking）機制與應用程式整合。包含本地函式庫的專案結構可能如下：

```plaintext
MyApp
├── node_modules
├── modules <-- folder for your local libraries
│ └── awesome-module <-- your local library
├── android
├── ios
├── src
├── index.js
└── package.json
```

由於本地函式庫的代碼存在於 `android/` 和 `ios/` 目錄之外，未來升級 React Native 版本、複製到其他專案等操作會更加便捷。

我們將使用 [create-react-native-library](https://callstack.github.io/react-native-builder-bob/create) 工具創建本地函式庫，該工具包含所有必要的模板。

### 開始使用

在 React Native 應用程式的根目錄下執行以下命令：

```shell
npx create-react-native-library@latest awesome-module
```

其中 `awesome-module` 是新模組的名稱。完成提示操作後，專案根目錄會生成一個名為 `modules` 的新資料夾，其中包含該模組。

### 連結

預設情況下，生成的函式庫會根據包管理器自動連結：使用 Yarn 時採用 `link:` 協議，使用 npm 時採用 `file:` 協議：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>

<TabItem value="npm">

```json
"dependencies": {
  "awesome-module": "file:./modules/awesome-module"
}
```

</TabItem>
<TabItem value="yarn">

```json
"dependencies": {
  "awesome-module": "link:./modules/awesome-module"
}
```

</TabItem>
</Tabs>

這會在 `node_modules` 下建立指向函式庫的符號連結，使自動連結機制生效。

### 安裝依賴項

連結模組前需安裝依賴項：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>

<TabItem value="npm">

```shell
npm install
```

</TabItem>
<TabItem value="yarn">

```shell
yarn install
```

</TabItem>
</Tabs>

### 在應用程式中使用模組

可通過模組名稱直接導入使用：

```js
import {multiply} from 'awesome-module';
```