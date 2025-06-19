# 跨平台原生模組 (C++)

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

使用 C++ 編寫模組是實現 Android 和 iOS 平台無關程式碼共享的最佳方式。透過純 C++ 模組，您只需編寫一次邏輯，即可在所有平台上直接重複使用，無需編寫平台特定程式碼。

本指南將逐步說明如何建立純 C++ Turbo 原生模組：

1. 建立 JS 規格文件
2. 配置 Codegen 生成框架程式碼
3. 實作原生邏輯
4. 在 Android 和 iOS 應用中註冊模組
5. 在 JS 中測試變更

本指南假設您已透過以下指令建立應用程式：

```shell
npx @react-native-community/cli@latest init SampleApp --version 0.76.0
```

## 1. 建立 JS 規格文件

純 C++ Turbo 原生模組屬於 Turbo 原生模組，需要規格文件（也稱為 spec 文件）讓 Codegen 能為我們生成框架程式碼。該規格文件也是我們在 JS 中存取 Turbo 原生模組的介面。

規格文件需使用類型化 JS 方言編寫。React Native 目前支援 Flow 或 TypeScript。

1. 在應用程式根目錄下建立名為 `specs` 的新資料夾
2. 建立名為 `NativeSampleModule.ts` 的新檔案，內容如下：

:::warning
所有 Turbo 原生模組規格文件必須以 `Native` 為前綴，否則 Codegen 會忽略它們。
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

此配置告知 Codegen 在 `specs` 資料夾中尋找規格文件，並指示 Codegen 僅為 `modules` 生成程式碼，同時將生成的程式碼命名空間設為 `AppSpecs`。

## 3. 編寫原生程式碼

編寫 C++ Turbo 原生模組可讓您在 Android 和 iOS 之間共享程式碼。因此我們只需編寫一次程式碼，然後研究需要對平台進行哪些調整才能使 C++ 程式碼生效。

1. Create a folder named `shared` at the same level as the `android` and `ios` folders.
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

讓我們來看看這兩個建立的檔案：

- The `NativeSampleModule.h` file is the header file for a Pure C++ TurboModule. The `include` statements make sure that we include the specs that will be created by Codegen and that contains the interface and the base class we need to implement.
- The module lives in the `facebook::react` namespace to have access to all the types that live in that namespace.
- The class `NativeSampleModule` is the actual Turbo Native Module class and it extends the `NativeSampleModuleCxxSpec` class which contains some glue code and boilerplate code to let this class behave as a Turbo Native Module.
- Finally, we have the constructor, that accepts a pointer to the `CallInvoker`, to communicate with JS if needed and the function's prototype we have to implement.

The `NativeSampleModule.cpp` file is the actual implementation of our Turbo Native Module and implements the constructor and the method that we declared in the specs.

## 4. 在平台中註冊模組

接下來的步驟將讓我們在平台中註冊模組。這一步驟將原生程式碼暴露給 JS，以便 React Native 應用程式最終可以從 JS 層調用原生方法。

這是我們唯一需要編寫一些平台特定程式碼的時候。

### Android

為了確保 Android 應用程式能夠有效地構建 C++ Turbo Native Module，我們需要：

1. 創建一個 `CMakeLists.txt` 來存取我們的 C++ 程式碼。
2. 修改 `build.gradle` 以指向新創建的 `CMakeLists.txt` 檔案。
3. 在我們的 Android 應用程式中創建一個 `OnLoad.cpp` 檔案來註冊新的 Turbo Native Module。

#### 1. 創建 `CMakeLists.txt` 檔案

Android 使用 CMake 進行構建。CMake 需要存取我們在共享資料夾中定義的檔案才能構建它們。

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
- 加載基礎 React Native 的 CMake 檔案。
- 使用 `target_sources` 指令添加我們需要構建的模組 C++ 原始碼。預設情況下，React Native 已經會將預設原始碼填充到 `appmodules` 庫中，這裡我們包含我們自定義的部分。你可以看到我們需要從 `jni` 資料夾回溯到我們 C++ Turbo Module 所在的 `shared` 資料夾。
- 指定 CMake 可以找到模組標頭檔的位置。同樣在這種情況下，我們需要從 `jni` 資料夾回溯。

#### 2. 修改 `build.gradle` 以包含自定義 C++ 程式碼

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

1. 從 `SampleApp/android/app/src/main/jni` 資料夾運行以下命令：

```sh
curl -O https://raw.githubusercontent.com/facebook/react-native/v0.76.0/packages/react-native/ReactAndroid/cmake-utils/default-app-setup/OnLoad.cpp
```

2. 接著，修改此檔案如下：

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

這些步驟會從 React Native 下載原始的 `OnLoad.cpp` 檔案，以便我們能安全地覆寫它，將 C++ Turbo Native Module 載入應用程式中。

下載檔案後，我們可以透過以下方式修改它：

- 引入指向我們模組的標頭檔
- 註冊 Turbo Native Module，讓當 JS 需要時，應用程式能回傳它。

現在，你可以從專案根目錄執行 `yarn android`，確認應用程式能成功建置。

### iOS

為了確保 iOS 應用程式能有效建置 C++ Turbo Native Module，我們需要：

1. 安裝 pods 並執行 Codegen。
2. 將 `shared` 資料夾加入我們的 iOS 專案。
3. 在應用程式中註冊 C++ Turbo Native Module。

#### 1. 安裝 Pods 並執行 Codegen

首先我們需要執行每次準備 iOS 應用程式時的例行步驟。CocoaPods 是我們用來設定和安裝 React Native 相依套件的工具，在此過程中，它也會為我們執行 Codegen。

```bash
cd ios
bundle install
bundle exec pod install
```

#### 2. 將 shared 資料夾加入 iOS 專案

此步驟將 `shared` 資料夾加入專案，讓 Xcode 能看見它。

1. 開啟 CocoPods 產生的 Xcode Workspace。

```bash
cd ios
open SampleApp.xcworkspace
```

2. 點擊左側的 `SampleApp` 專案，選擇 `Add files to "Sample App"...`。

![Add Files to Sample App...](/docs/assets/AddFilesToXcode1.png)

3. 選擇 `shared` 資料夾並點擊 `Add`。

![Add Files to Sample App...](/docs/assets/AddFilesToXcode2.png)

如果一切正確，左側的專案應會顯示如下：

![Xcode Project](/docs/assets/CxxTMGuideXcodeProject.png)

#### 3. 在應用程式中註冊 Cxx Turbo Native Module

要在應用程式中註冊純 Cxx Turbo Native Module，你需要：

1. Create a `ModuleProvider` for the Native Module
2. Configure the `package.json` to associate the JS module name with the ModuleProvider class.

ModuleProvider 是一個 Objective-C++ 類別，負責將純 C++ 模組與 iOS 應用程式的其他部分連接起來。

##### 3.1 建立 ModuleProvider

1. From Xcode, select the `SampleApp` project and press <kbd>⌘</kbd> + <kbd>N</kbd> to create a new file.
2. Select the `Cocoa Touch Class` template
3. Add the name `SampleNativeModuleProvider` (keep the other field as `Subclass of: NSObject` and `Language: Objective-C`)
4. Click Next to generate the files.
5. Rename the `SampleNativeModuleProvider.m` to `SampleNativeModuleProvider.mm`. The `mm` extension denotes an Objective-C++ file.
6. Implement the content of the `SampleNativeModuleProvider.h` with the following:

```objc title="NativeSampleModuleProvider.h"

#import <Foundation/Foundation.h>
#import <ReactCommon/RCTTurboModule.h>

NS_ASSUME_NONNULL_BEGIN

@interface NativeSampleModuleProvider : NSObject <RCTModuleProvider>

@end

NS_ASSUME_NONNULL_END
```

This declares a `NativeSampleModuleProvider` object that conforms to the `RCTModuleProvider` protocol.

7. 在 `SampleNativeModuleProvider.mm` 中實作以下內容：

```objc title="NativeSampleModuleProvider.mm"

#import "NativeSampleModuleProvider.h"
#import <ReactCommon/CallInvoker.h>
#import <ReactCommon/TurboModule.h>
#import "NativeSampleModule.h"

@implementation NativeSampleModuleProvider

- (std::shared_ptr<facebook::react::TurboModule>)getTurboModule:
    (const facebook::react::ObjCTurboModule::InitParams &)params
{
  return std::make_shared<facebook::react::NativeSampleModule>(params.jsInvoker);
}

@end
```

This code implements the `RCTModuleProvider` protocol by creating the pure C++ `NativeSampleModule` when the `getTurboModule:` method is called.

##### 3.2 更新 package.json

最後一步是更新 `package.json`，告知 React Native 關於原生模組的 JS 規格與實際原生程式碼實作之間的關聯。

請依照以下方式修改 `package.json`：

```json title="package.json"
     "start": "react-native start",
     "test": "jest"
   },
   "codegenConfig": {
     "name": "AppSpecs",
     "type": "modules",
     "jsSrcsDir": "specs",
     "android": {
       "javaPackageName": "com.sampleapp.specs"
     // highlight-add-start
     },
     "ios": {
        "modulesProvider": {
          "NativeSampleModule":  "NativeSampleModuleProvider"
        }
     }
     // highlight-add-end
   },

   "dependencies": {
```

此時，您需要重新安裝 pods 以確保 Codegen 再次執行並生成新檔案：

```bash
# from the ios folder
bundle exec pod install
open SampleApp.xcworkspace
```

若您現在從 Xcode 建置應用程式，應該能成功完成建置。

## 5. 測試您的程式碼

現在是時候從 JS 端存取我們的 C++ Turbo 原生模組了。為此，我們需要修改 `App.tsx` 檔案，導入 Turbo 原生模組並在程式碼中呼叫它。

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

此應用程式中值得注意的程式行包括：

- `import SampleTurboModule from './specs/NativeSampleModule';`: this line imports the Turbo Native Module in the app,
- `const revString = SampleTurboModule.reverseString(value);` in the `onPress` callback: this is how you can use the Turbo Native Module in your app.

:::warning
為了簡化範例並保持簡潔，我們直接將規格檔案導入應用程式中。
最佳實踐是建立一個單獨的檔案來封裝規格，並在應用程式中使用該檔案。
這讓您能為規格準備輸入資料，並在 JS 端獲得更多控制權。
:::

恭喜，您已成功編寫了第一個 C++ Turbo 原生模組！

| <center>Android</center>                                                                             | <center>iOS</center>                                                                          |
| ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| <center><img src="/docs/assets/CxxGuideAndroidVideo.gif" alt="Android Video" height="600"/></center> | <center><img src="/docs/assets/CxxGuideIOSVideo.gif" alt="iOS video" height="600" /></center> |