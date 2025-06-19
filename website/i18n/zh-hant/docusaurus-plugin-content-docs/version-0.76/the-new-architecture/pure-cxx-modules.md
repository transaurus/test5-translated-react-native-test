# 跨平台原生模組 (C++)

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

使用 C++ 編寫模組是共享 Android 和 iOS 之間平台無關程式碼的最佳方式。透過純 C++ 模組，您只需編寫一次邏輯，即可立即在所有平台上重複使用，無需編寫平台特定程式碼。

本指南將逐步介紹如何建立純 C++ Turbo 原生模組：

1. 建立 JS 規格文件
2. 配置 Codegen 以生成框架程式碼
3. 實作原生邏輯
4. 在 Android 和 iOS 應用中註冊模組
5. 在 JS 中測試您的變更

本指南的其餘部分假設您已透過以下命令建立應用程式：

```shell
npx @react-native-community/cli@latest init SampleApp --version 0.76.0
```

## 1. 建立 JS 規格文件

純 C++ Turbo 原生模組屬於 Turbo 原生模組。它們需要一個規格文件（也稱為 spec 文件），以便 Codegen 能為我們生成框架程式碼。規格文件也是我們在 JS 中存取 Turbo 原生模組的方式。

規格文件需要使用類型化的 JS 方言編寫。React Native 目前支援 Flow 或 TypeScript。

1. 在應用程式的根目錄中，建立一個名為 `specs` 的新資料夾。
2. 建立一個名為 `NativeSampleModule.ts` 的新文件，並包含以下程式碼：

:::warning
所有原生 Turbo 模組規格文件必須以 `Native` 為前綴，否則 Codegen 將忽略它們。
:::

<Tabs groupId="tnm-specs" queryString defaultValue={constants.defaultJavaScriptSpecLanguages} values={constants.javaScriptSpecLanguages}>
<TabItem value="flow">

```ts title="specs/NativeSampleModule.ts"
// @flow
import type {TurboModule} from 'react-native'
import { TurboModuleRegistry } from "react-native";

export interface Spec extends TurboModule {
  +reverseString: (input: string) => string;
}

export default (TurboModuleRegistry.getEnforcing<Spec>(
  "NativeSampleModule"
): Spec);
```

</TabItem>
<TabItem value="typescript">

```ts title="specs/NativeSampleModule.ts"
import {TurboModule, TurboModuleRegistry} from 'react-native';

export interface Spec extends TurboModule {
  readonly reverseString: (input: string) => string;
}

export default TurboModuleRegistry.getEnforcing<Spec>(
  'NativeSampleModule',
);
```

</TabItem>
</Tabs>

## 2. 配置 Codegen

The next step is to configure [Codegen](what-is-codegen.md) in your `package.json`. Update the file to include:

```json title="package.json"
     "start": "react-native start",
     "test": "jest"
   },
   // highlight-add-start
   "codegenConfig": {
     "name": "AppSpecs",
     "type": "modules",
     "jsSrcsDir": "specs",
     "android": {
       "javaPackageName": "com.sampleapp.specs"
     }
   },
   // highlight-add-end
   "dependencies": {
```

此配置告訴 Codegen 在 `specs` 資料夾中尋找規格文件。它還指示 Codegen 僅為 `modules` 生成程式碼，並將生成的程式碼命名空間設為 `AppSpecs`。

## 3. 編寫原生程式碼

編寫 C++ Turbo 原生模組可讓您在 Android 和 iOS 之間共享程式碼。因此，我們只需編寫一次程式碼，然後研究需要對平台進行哪些變更，以便 C++ 程式碼能被正確引用。

1. Create a folder named `shared` at the same level of the `android` and `ios` folders.
2. Inside the `shared` folder, create a new file called `NativeSampleModule.h`.

   ```cpp title="shared/NativeSampleModule.h"
   #pragma once

   #include <AppSpecsJSI.h>

   #include <memory>
   #include <string>

   namespace facebook::react {

   class NativeSampleModule : public NativeSampleModuleCxxSpec<NativeSampleModule> {
   public:
     NativeSampleModule(std::shared_ptr<CallInvoker> jsInvoker);

     std::string reverseString(jsi::Runtime& rt, std::string input);
   };

   } // namespace facebook::react

   ```

3. Inside the `shared` folder, create a new file called `NativeSampleModule.cpp`.

   ```cpp title="shared/NativeSampleModule.cpp"
   #include "NativeSampleModule.h"

   namespace facebook::react {

   NativeSampleModule::NativeSampleModule(std::shared_ptr<CallInvoker> jsInvoker)
       : NativeSampleModuleCxxSpec(std::move(jsInvoker)) {}

   std::string NativeSampleModule::reverseString(jsi::Runtime& rt, std::string input) {
     return std::string(input.rbegin(), input.rend());
   }

   } // namespace facebook::react
   ```

讓我們看看我們建立的這兩個文件：

- The `NativeSampleModule.h` file is the header file for a Pure C++ TurboModule. The `include` statements make sure that we include the specs that will be created by Codegen and that contains the interface and the base class we need to implement.
- The module lives in the `facebook::react` namespace to have access to all the types that live in that namespace.
- The class `NativeSampleModule` is the actual Turbo Native Module class and it extends the `NativeSampleModuleCxxSpec` class which contains some glue code and boilerplate code to let this class behave as a Turbo Native Module.
- Finally, we have the constructor, that accepts a pointer to the `CallInvoker`, to communicate with JS if needed and the function's prototype we have to implement.

The `NativeSampleModule.cpp` files is the actual implementation of our Turbo Native Module and implements the constructor and the method that we declared in the specs.

## 4. 在平台中註冊模組

接下來的步驟將讓我們在平台中註冊模組。這一步驟將原生程式碼暴露給 JS，以便 React Native 應用程式最終可以從 JS 層調用原生方法。

這是我們唯一需要編寫一些平台特定程式碼的時候。

### Android

為了確保 Android 應用程式能夠有效地構建 C++ Turbo Native Module，我們需要：

1. 創建一個 `CMakeLists.txt` 來存取我們的 C++ 程式碼。
2. 修改 `build.gradle` 以指向新創建的 `CMakeLists.txt` 檔案。
3. 在我們的 Android 應用程式中創建一個 `OnLoad.cpp` 檔案來註冊新的 Turbo Native Module。

#### 1. 創建 `CMakeLists.txt` 檔案

Android 使用 CMake 進行構建。CMake 需要存取我們在共享資料夾中定義的檔案，以便能夠構建它們。

1. 創建一個新資料夾 `SampleApp/android/app/src/main/jni`。`jni` 資料夾是 Android 的 C++ 部分所在的位置。
2. 創建一個 `CMakeLists.txt` 檔案並添加以下內容：

```shell title="CMakeLists.txt"
cmake_minimum_required(VERSION 3.13)

# Define the library name here.
project(appmodules)

# This file includes all the necessary to let you build your React Native application
include(${REACT_ANDROID_DIR}/cmake-utils/ReactNative-application.cmake)

# Define where the additional source code lives. We need to crawl back the jni, main, src, app, android folders
target_sources(${CMAKE_PROJECT_NAME} PRIVATE ../../../../../shared/NativeSampleModule.cpp)

# Define where CMake can find the additional header files. We need to crawl back the jni, main, src, app, android folders
target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC ../../../../../shared)
```

CMake 檔案執行以下操作：

- 定義 `appmodules` 庫，其中將包含所有應用程式的 C++ 程式碼。
- 載入基礎的 React Native 的 CMake 檔案。
- 使用 `target_sources` 指令添加我們需要構建的模組 C++ 原始碼。預設情況下，React Native 已經會將預設原始碼填充到 `appmodules` 庫中，這裡我們包含我們自訂的部分。你可以看到我們需要從 `jni` 資料夾回溯到 `shared` 資料夾，那裡有我們的 C++ Turbo Module。
- 指定 CMake 可以找到模組標頭檔的位置。同樣在這種情況下，我們需要從 `jni` 資料夾回溯。

#### 2. 修改 `build.gradle` 以包含自訂 C++ 程式碼

Gradle 是協調 Android 構建的工具。我們需要告訴它可以在哪裡找到構建 Turbo Native Module 的 `CMake` 檔案。

1. 打開 `SampleApp/android/app/build.gradle` 檔案。
2. 在現有的 `android` 區塊中添加以下內容：

```diff title="android/app/build.gradle"
    buildTypes {
        debug {
            signingConfig signingConfigs.debug
        }
        release {
            // Caution! In production, you need to generate your own keystore file.
            // see https://reactnative.dev/docs/signed-apk-android.
            signingConfig signingConfigs.debug
            minifyEnabled enableProguardInReleaseBuilds
            proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
        }
    }

+   externalNativeBuild {
+       cmake {
+           path "src/main/jni/CMakeLists.txt"
+       }
+   }
}
```

This block tells the Gradle file where to look for the CMake file. The path is relative to the folder where the `build.gradle` file lives, so we need to add the path to the `CMakeLists.txt` files in the `jni` folder.

#### 3. 註冊新的 Turbo Native Module

最後一步是在運行時註冊新的 C++ Turbo Native Module，以便當 JS 請求 C++ Turbo Native Module 時，應用程式知道在哪裡找到它並可以返回它。

1. 從 `SampleApp/android/app/src/main/jni` 資料夾中運行以下命令：

```sh
curl -O https://raw.githubusercontent.com/facebook/react-native/v0.76.0/packages/react-native/ReactAndroid/cmake-utils/default-app-setup/OnLoad.cpp
```

2. 接著，按照以下方式修改此檔案：

```diff title="android/app/src/main/jni/OnLoad.cpp"
#include <DefaultComponentsRegistry.h>
#include <DefaultTurboModuleManagerDelegate.h>
#include <autolinking.h>
#include <fbjni/fbjni.h>
#include <react/renderer/componentregistry/ComponentDescriptorProviderRegistry.h>
#include <rncore.h>

+ // Include the NativeSampleModule header
+ #include <NativeSampleModule.h>

//...

std::shared_ptr<TurboModule> cxxModuleProvider(
    const std::string& name,
    const std::shared_ptr<CallInvoker>& jsInvoker) {
  // Here you can provide your CXX Turbo Modules coming from
  // either your application or from external libraries. The approach to follow
  // is similar to the following (for a module called `NativeCxxModuleExample`):
  //
  // if (name == NativeCxxModuleExample::kModuleName) {
  //   return std::make_shared<NativeCxxModuleExample>(jsInvoker);
  // }

+  // This code register the module so that when the JS side asks for it, the app can return it
+  if (name == NativeSampleModule::kModuleName) {
+    return std::make_shared<NativeSampleModule>(jsInvoker);
+  }

  // And we fallback to the CXX module providers autolinked
  return autolinking_cxxModuleProvider(name, jsInvoker);
}

// leave the rest of the file
```

這些步驟會從 React Native 下載原始的 `OnLoad.cpp` 檔案，以便我們安全地覆寫它，將 C++ Turbo Native Module 載入應用程式中。

下載檔案後，我們可以透過以下方式進行修改：

- 引入指向我們模組的標頭檔
- 註冊 Turbo Native Module，以便當 JS 要求時，應用程式能夠回傳它。

現在，你可以從專案根目錄執行 `yarn android`，確認應用程式成功建置。

### iOS

為了確保 iOS 應用程式能正確建置 C++ Turbo Native Module，我們需要：

1. 安裝 pods 並執行 Codegen。
2. 將 `shared` 資料夾加入 iOS 專案。
3. 在應用程式中註冊 C++ Turbo Native Module。

#### 1. 安裝 Pods 並執行 Codegen

首先需執行每次準備 iOS 應用程式時的標準步驟。CocoaPods 是我們用來設定與安裝 React Native 相依套件的工具，在此過程中它也會為我們執行 Codegen。

```bash
cd ios
bundle install
bundle exec pod install
```

#### 2. 將 shared 資料夾加入 iOS 專案

此步驟將 `shared` 資料夾加入專案，使其對 Xcode 可見。

1. 開啟 CocoPods 產生的 Xcode Workspace。

```bash
cd ios
open SampleApp.xcworkspace
```

2. 點擊左側的 `SampleApp` 專案，選擇 `Add files to "Sample App"...`。

![Add Files to Sample App...](/docs/assets/AddFilesToXcode1.png)

3. 選取 `shared` 資料夾後點擊 `Add`。

![Add Files to Sample App...](/docs/assets/AddFilesToXcode2.png)

若一切操作正確，左側的專案結構應如下所示：

![Xcode Project](/docs/assets/CxxTMGuideXcodeProject.png)

#### 3. 在應用程式中註冊 Cxx Turbo Native Module

最後這個步驟，我們將告訴 iOS 應用程式從何處尋找純 C++ Turbo Native Module。

在 Xcode 中開啟 `AppDelegate.mm` 檔案並修改如下：

```diff title="SampleApp/AppDelegate.mm"
#import <React/RCTBundleURLProvider.h>
+ #import <RCTAppDelegate+Protected.h>
+ #import "NativeSampleModule.h"

// ...
  return [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
#endif
}

+- (std::shared_ptr<facebook::react::TurboModule>)getTurboModule:(const std::string &)name
+                                                      jsInvoker:(std::shared_ptr<facebook::react::CallInvoker>)jsInvoker
+{
+  if (name == "NativeSampleModule") {
+    return std::make_shared<facebook::react::NativeSampleModule>(jsInvoker);
+  }
+
+  return [super getTurboModule:name jsInvoker:jsInvoker];
+}

@end
```

這些修改主要實現以下功能：

1. 引入 `RCTAppDelegate+Protected` 標頭檔，使 AppDelegate 明確遵循 `RCTTurboModuleManagerDelegate` 協議。
2. 引入純 C++ Native Turbo Module 介面 `NativeSampleModule.h`
3. 覆寫 C++ 模組的 `getTurboModule` 方法，當 JS 端請求名為 `NativeSampleModule` 的模組時，應用程式知道該回傳哪個模組。

若現在從 Xcode 建置應用程式，應能成功完成建置。

## 5. 測試你的程式碼

現在是時候從 JS 端呼叫我們的 C++ Turbo Native Module 了。為此，我們需要修改 `App.tsx` 檔案來引入 Turbo Native Module 並在程式碼中呼叫它。

1. 開啟 `App.tsx` 檔案。
2. 將範本內容替換為以下程式碼：

```tsx title="App.tsx"
import React from 'react';
import {
  Button,
  SafeAreaView,
  StyleSheet,
  Text,
  TextInput,
  View,
} from 'react-native';
import SampleTurboModule from './specs/NativeSampleModule';

function App(): React.JSX.Element {
  const [value, setValue] = React.useState('');
  const [reversedValue, setReversedValue] = React.useState('');

  const onPress = () => {
    const revString = SampleTurboModule.reverseString(value);
    setReversedValue(revString);
  };

  return (
    <SafeAreaView style={styles.container}>
      <View>
        <Text style={styles.title}>
          Welcome to C++ Turbo Native Module Example
        </Text>
        <Text>Write down here he text you want to revert</Text>
        <TextInput
          style={styles.textInput}
          placeholder="Write your text here"
          onChangeText={setValue}
          value={value}
        />
        <Button title="Reverse" onPress={onPress} />
        <Text>Reversed text: {reversedValue}</Text>
      </View>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 18,
    marginBottom: 20,
  },
  textInput: {
    borderColor: 'black',
    borderWidth: 1,
    borderRadius: 5,
    padding: 10,
    marginTop: 10,
  },
});

export default App;
```

此應用程式中值得注意的程式碼行如下：

- `import SampleTurboModule from './specs/NativeSampleModule';`：這行程式碼將 Turbo Native Module 導入應用程式，
- `const revString = SampleTurboModule.reverseString(value);` 位於 `onPress` 回調函數中：這是您在應用程式中使用 Turbo Native Module 的方式。

:::warning
為了簡化範例並保持簡潔，我們直接在應用程式中導入規格文件。
最佳實踐是建立一個單獨的文件來封裝規格，並在應用程式中使用該文件。
這樣可以讓您準備規格的輸入參數，並在 JavaScript 中獲得更多控制權。
:::

恭喜，您已成功編寫了第一個 C++ Turbo Native Module！

| <center>Android</center>                                                                             | <center>iOS</center>                                                                          |
| ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| <center><img src="/docs/assets/CxxGuideAndroidVideo.gif" alt="Android Video" height="600"/></center> | <center><img src="/docs/assets/CxxGuideIOSVideo.gif" alt="iOS video" height="600" /></center> |