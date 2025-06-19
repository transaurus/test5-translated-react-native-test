# 為你的模組建立函式庫

React Native 擁有豐富的函式庫生態系統來解決常見問題。我們將 React Native 函式庫收集在 [reactnative.directory](https://reactnative.directory) 網站上，這是每位 React Native 開發者都值得收藏的寶貴資源。

有時，你可能會開發一個值得提取為獨立函式庫以供重複使用的模組。這可能是你想在所有應用中重用的函式庫、想作為開源元件分發給生態系統的函式庫，甚至是你想販售的函式庫。

在本指南中，你將學習：

- 如何將模組提取為函式庫
- 如何使用 NPM 分發函式庫

## 將模組提取為函式庫

你可以使用 [`create-react-native-library`](https://callstack.github.io/react-native-builder-bob/create) 工具來建立新函式庫。此工具會設置一個包含所有必要樣板程式碼的新函式庫：所有配置檔案和各平台所需的檔案。它還提供了一個互動式選單來引導你完成函式庫的建立過程。

要將模組提取為獨立函式庫，你可以按照以下步驟操作：

1. 建立新函式庫
2. 將程式碼從應用移動到函式庫
3. 更新程式碼以反映新結構
4. 發佈它。

### 1. 建立函式庫

1. 執行以下命令開始建立過程：

```sh
npx create-react-native-library@latest <Name of Your Library>
```

2. 為你的模組添加名稱。它必須是有效的 npm 名稱，因此應全部使用小寫字母。你可以使用 `-` 來分隔單詞。
3. 為套件添加描述。
4. 繼續填寫表單，直到你看到問題 _"你想開發什麼類型的函式庫？"_
   ![函式庫類型](/docs/assets/what-library.png)
5. 為了本指南的緣故，選擇 _Turbo 模組_ 選項。請注意，你可以為新架構和舊架構建立函式庫。
6. 然後，你可以選擇是否要一個訪問平台（Kotlin 和 Objective-C）的函式庫，或是一個共享的 C++ 函式庫（適用於 Android 和 iOS 的 C++）。
7. 最後，選擇 `Test App` 作為最後一個選項。此選項會在函式庫資料夾內建立一個已配置好的獨立應用。

一旦互動式提示完成，該工具會建立一個在 Visual Studio Code 中看起來像這樣的資料夾結構：

<img class="half-size" alt="Folder structure after initializing a new library." src="/docs/assets/turbo-native-modules/c++visualstudiocode.webp" />

請隨意探索為你建立的程式碼。然而，最重要的部分包括：

- The `android` folder: this is where the Android code lives
- The `cpp` folder: this is where the c++ code lives
- The `ios` folder: this is where the iOS code lives
- The `src` folder: this is where the JS code lives.

The `package.json` is already configured with all the information that we provided to the `create-react-native-library` tool, including the name and the description of the package. Notice that the `package.json` is also already configured to run Codegen.

```json
  "codegenConfig": {
    "name": "RN<your module name>Spec",
    "type": "all",
    "jsSrcsDir": "src",
    "outputDir": {
      "ios": "ios/generated",
      "android": "android/generated"
    },
    "android": {
      "javaPackageName": "com.<name-of-the-module>"
    }
  },
```

最後，函式庫已經包含了所有讓函式庫能與 iOS 和 Android 連結的基礎設施。

### 2. 從你的應用複製程式碼

本指南的其餘部分假設你在應用中有一個本地 Turbo 原生模組，該模組是根據網站上其他指南的指導建立的：平台特定的 Turbo 原生模組，或[跨平台 Turbo 原生模組](./pure-cxx-modules)。但它也適用於元件和舊架構的模組與元件。你需要根據需要調整要複製和更新的檔案。

<!-- TODO: add links for Turbo Native Modules -->

1. **[Not required for legacy architecture modules and components]** Move the code you have in the `specs` folder in your app into the `src` folder created by the `create-react-native-library` folder.
2. Update the `index.ts` file to properly export the Turbo Native Module spec so that it is accessible from the library. For example:

```ts
import NativeSampleModule from './NativeSampleModule';

export default NativeSampleModule;
```

3. Copy the native module over:

   - Replace the code in the `android/src/main/java/com/<name-of-the-module>` with the code you wrote in the app for your native module, if any.
   - Replace the code in the `ios` folder with the code you wrote in your app for your native module, if any.
   - Replace the code in the `cpp` folder with the code you wrote in your app for your native module, if any.

4. **[Not required for legacy architecture modules and components]** Update all the references from the previous spec name to the new spec name, the one that is defined in the `codegenConfig` field of the library's `package.json`. For example, if in the app `package.json` you set `AppSpecs` as `codegenConfig.name` and in the library it is called `RNNativeSampleModuleSpec`, you have to replace every occurrence of `AppSpecs` with `RNNativeSampleModuleSpec`.

完成！您已將所有必要程式碼從應用移出至獨立程式庫。

## 測試程式庫

The `create-react-native-library` comes with a useful example application that is already configured to work properly with the library. This is a great way to test it!

查看 `example` 資料夾，其結構與透過 [`react-native-community/template`](https://github.com/react-native-community/template) 建立的 React Native 應用完全相同。

測試步驟：

1. Navigate to the `example` folder.
2. Run `yarn install` to install all the dependencies.
3. For iOS only, you need to install CocoaPods: `cd ios && pod install`.
4. Build and run Android with `yarn android` from the `example` folder.
5. Build and run iOS with `yarn ios` from the `example` folder.

## 將程式庫作為本地模組使用

某些情境下，您可能希望直接將程式庫作為本地模組重用，無需發佈至 NPM。

此時您的專案結構可能會呈現應用與程式庫並列存放：

```shell
Development
├── App
└── Library
```

透過以下步驟使用 `create-react-native-library` 建立的程式庫：

1. add you library to your app by navigating into the `App` folder and running `yarn add ../Library`.
2. For iOS only, navigate in the `App/ios` folder and run `bundle exec pod install` to install your dependencies.
3. Update the `App.tsx` code to import the code in your library. For example:

```tsx
import NativeSampleModule from '@site/versioned_docs/version-0.80/Library/src/index';
```

If you run your app right now, Metro would not find the JS files that it needs to serve to the app. That's because metro will be running starting from the `App` folder and it would not have access to the JS files located in the `Library` folder. To fix this, let's update the `metro.config.js` file as it follows

```diff
const {getDefaultConfig, mergeConfig} = require('@react-native/metro-config');

/**
 * Metro configuration
 * https://reactnative.dev/docs/metro
 *
 * @type {import('metro-config').MetroConfig}
 */
+ const path = require('path');

- const config = {}
+ const config = {
+  // Make Metro able to resolve required external dependencies
+  watchFolders: [
+    path.resolve(__dirname, '../Library'),
+  ],
+  resolver: {
+    extraNodeModules: {
+      'react-native': path.resolve(__dirname, 'node_modules/react-native'),
+    },
+  },
+};

module.exports = mergeConfig(getDefaultConfig(__dirname), config);
```

The `watchFolders` configs tells Metro to watch for files and changes in some additional paths, in this case to the `../Library` path, which contains the `src/index` file you need.
The `resolver`property is required to feed to the library the React Native code used by the app. The library might refer and import code from React Native: without the additional resolver, the imports in the library will fail.

完成後即可正常建置與執行應用：

- Build and run Android with `yarn android` from the `example` folder.
- Build and run iOS with `yarn ios` from the `example` folder.

## 將函式庫發佈至 NPM

由於 `create-react-native-library` 已預先配置好發佈流程，因此發佈至 NPM 的設定已準備就緒。

1. 在模組中安裝相依套件：執行 `yarn install`。
2. 建置函式庫：執行 `yarn prepare`。
3. 發佈函式庫：執行 `yarn release`。

稍候片刻，您的函式庫就會出現在 NPM 上。若要確認，請執行：

```bash
npm view <package.name>
```

where `package.name` is the `name` you set up in the `package.json` file during the initialization of the library.

現在，您可以透過以下指令將函式庫安裝至應用程式中：

```bash
yarn add <package.name>
```

:::note
僅限 iOS：每當安裝含有原生程式碼的新模組時，您必須重新安裝 CocoaPods，建議執行 `bundle exec pod install`（推薦使用 Ruby 的 Bundler）或直接執行 `pod install`（不推薦）。
:::

恭喜！您已成功發佈第一個 React Native 函式庫。