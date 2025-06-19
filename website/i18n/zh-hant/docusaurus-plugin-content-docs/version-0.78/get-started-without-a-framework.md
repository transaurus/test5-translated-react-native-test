---
id: getting-started-without-a-framework
title: Get Started Without a Framework
hide_table_of_contents: true
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import constants from '@site/core/TabsConstants';
import PlatformSupport from '@site/src/theme/PlatformSupport';

import RemoveGlobalCLI from './\_remove-global-cli.md';

<PlatformSupport platforms={['android', 'ios', 'macOS', 'tv', 'watchOS', 'web', 'windows', 'visionOS']} />

如果現有的[框架](/architecture/glossary#react-native-framework)無法滿足您的需求，或者您偏好自行開發框架，您可以在不使用框架的情況下建立 React Native 應用程式。

為此，您首先需要[設定開發環境](set-up-your-environment)。完成環境設定後，請按照以下步驟建立應用程式並開始開發。

### 步驟 1：建立新應用程式

<RemoveGlobalCLI />

您可以使用 [React Native Community CLI](https://github.com/react-native-community/cli) 來生成新專案。讓我們建立一個名為「AwesomeProject」的新 React Native 專案：

```shell
npx @react-native-community/cli@latest init AwesomeProject
```

如果您是將 React Native 整合到現有應用程式中，或已在專案中安裝 [Expo](https://docs.expo.dev/bare/installing-expo-modules/)，或是為現有 React Native 專案新增 Android 支援（請參閱[與現有應用程式整合](integration-with-existing-apps.md)），則不需要此步驟。您也可以使用第三方 CLI（如 [Ignite CLI](https://github.com/infinitered/ignite)）來設定 React Native 應用程式。

:::info

If you are having trouble with iOS, try to reinstall the dependencies by running:

1. `cd ios` to navigate to the `ios` folder.
2. `bundle install` to install [Bundler](https://bundler.io/)
3. `bundle exec pod install` to install the iOS dependencies managed by CocoaPods.

:::

#### [選用] 使用特定版本或模板

若要使用特定 React Native 版本建立新專案，可使用 `--version` 參數：

```shell
npx @react-native-community/cli@X.XX.X init AwesomeProject --version X.XX.X
```

您也可以透過 `--template` 參數使用自訂的 React Native 模板建立專案，詳情請參閱[此處](https://github.com/react-native-community/cli/blob/main/docs/init.md#initializing-project-with-custom-template)。

### 步驟 2：啟動 Metro

[**Metro**](https://metrobundler.dev/) 是 React Native 的 JavaScript 建置工具。若要啟動 Metro 開發伺服器，請在專案資料夾中執行以下指令：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm start
```

</TabItem>
<TabItem value="yarn">

```shell
yarn start
```

</TabItem>
</Tabs>

:::note
若您熟悉網頁開發，Metro 類似於 Vite 或 webpack 等打包工具，但專為 React Native 設計。例如，Metro 使用 [Babel](https://babel.dev/) 將 JSX 等語法轉換為可執行的 JavaScript。
:::

### 步驟 3：啟動應用程式

讓 Metro Bundler 在獨立終端中運行。在 React Native 專案資料夾中開啟新終端機，執行以下指令：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run android
```

</TabItem>
<TabItem value="yarn">

```shell
yarn android
```

</TabItem>
</Tabs>

若一切設定正確，您應該很快就能在 Android 模擬器中看到新應用程式運行。

這是執行應用程式的方式之一，您也可以直接透過 Android Studio 執行。

> 若無法順利執行，請參閱[疑難排解](troubleshooting.md)頁面。

### 步驟 4：修改應用程式

現在您已成功執行應用程式，讓我們進行修改。

- Open `App.tsx` in your text editor of choice and edit some lines.
- Press the <kbd>R</kbd> key twice or select `Reload` from the Dev Menu (<kbd>Ctrl</kbd> + <kbd>M</kbd>) to see your changes!

### 完成了！

恭喜！您已成功執行並修改了第一個基礎 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

### 接下來呢？

- 若您想將此 React Native 程式碼整合至現有應用程式，請參閱[整合指南](integration-with-existing-apps.md)。
- 若您想進一步了解 React Native，請參閱[React Native 簡介](getting-started)。