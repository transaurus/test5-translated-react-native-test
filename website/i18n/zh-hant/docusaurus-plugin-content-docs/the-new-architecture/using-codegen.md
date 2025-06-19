# 使用 Codegen

本指南將教您如何：

- 配置 **Codegen**。
- 為每個平台手動調用它。

同時也描述了生成的程式碼。

## 先決條件

即使手動調用 **Codegen**，您始終需要一個 React Native 應用程式來正確生成程式碼。

The **Codegen** process is tightly coupled with the build of the app, and the scripts are located in the `react-native` NPM package.

為了本指南的緣故，請使用 React Native CLI 創建一個專案，如下所示：

```bash
npx @react-native-community/cli@latest init SampleApp --version 0.76.0
```

**Codegen** 用於為您的自訂模組或元件生成粘合程式碼。有關如何創建它們的詳細信息，請參閱 Turbo Native Modules 和 Fabric Native Components 的指南。

<!-- TODO: add links -->

## 配置 **Codegen**

**Codegen** can be configured in your app by modifying the `package.json` file. **Codegen** is controlled by a custom field called `codegenConfig`.

```json title="package.json"
  "codegenConfig": {
    "name": "<SpecName>",
    "type": "<types>",
    "jsSrcsDir": "<source_dir>",
    "android": {
      "javaPackageName": "<java.package.name>"
    },
    "ios": {
      "modules": {
        "TestModule": {
          "className": "<iOS-class-implementing-the-RCTModuleProvider-protocol>",
          "unstableRequiresMainQueueSetup": false
          "conformsToProtocols": ["RCTImageURLLoader", "RCTURLRequestHandler", "RCTImageDataDecoder"],
        }
      },
      "components": {
        "TestComponent": {
          "className": "<iOS-class-implementing-the-component>"
        }
      }
    }
  },
```

您可以將此片段添加到您的應用程式中，並自訂各個字段：

- `name:` Codegen 配置的名稱。這將自訂 codegen 的輸出：文件名和程式碼。
- `type:`
  - `modules:` 僅為模組生成程式碼。
  - `components:` 僅為元件生成程式碼。
  - `all`: 為所有內容生成程式碼。
- `jsSrcsDir`: 所有規格文件所在的根目錄。
- `android`: Android 的 Codegen 配置（全部可選）：
  - `.javaPackageName`: 配置 Android Java codegen 輸出的套件名稱。
- `ios`: iOS 的 Codegen 配置（全部可選）：
  - `.modules[moduleName]:`
    - `.className`: 此模組的 ObjC 類。或者，如果它是 [純 C++ 模組](/docs/next/the-new-architecture/pure-cxx-modules)，則是其 `RCTModuleProvider` 類。
    - `.unstableRequiresMainQueueSetup`: 在運行任何 JavaScript 之前，在 UI 線程上初始化此模組。
    - `.conformsToProtocols`: 註釋此模組符合以下哪些協議：[`RCTImageURLLoader`](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/Libraries/Image/RCTImageURLLoader.h#L26-L81)、[`RCTURLRequestHandler`](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/React/Base/RCTURLRequestHandler.h#L11-L52)、[`RCTImageDataDecoder`](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/Libraries/Image/RCTImageDataDecoder.h#L15-L53)。
  - `.components[componentName]`:
    - `.className`: 此元件的 ObjC 類（例如：`TextInput` -> `RCTTextInput`）。

當 **Codegen** 運行時，它會在應用程式的所有依賴項中搜索符合特定約定的 JS 文件，並生成所需的程式碼：

- Turbo Native Modules 要求規格文件以 `Native` 為前綴。例如，`NativeLocalStorage.ts` 是規格文件的有效名稱。
- Native Fabric Components 要求規格文件以 `NativeComponent` 為後綴。例如，`WebViewNativeComponent.ts` 是規格文件的有效名稱。

## 運行 **Codegen**

The rest of this guide assumes that you have a Native Turbo Module, a Native Fabric Component or both already set up in your project. We also assume that you have valid specification files in the `jsSrcsDir` specified in the `package.json`.

### Android

**Codegen** for Android is integrated with the React Native Gradle Plugin (RNGP). The RNGP contains a task that can be invoked that reads the configurations defined in the `package.json` file and execute **Codegen**. To run the gradle task, first navigate inside the `android` folder of your project. Then run:

```bash
./gradlew generateCodegenArtifactsFromSchema
```

此任務會在應用程式的所有導入專案（應用程式本身及其所有連結的 node 模組）上呼叫 `generateCodegenArtifactsFromSchema` 指令。生成的程式碼會存放在對應的 `node_modules/<dependency>` 資料夾中。例如，若你有一個名為 `my-fabric-component` 的 Fabric Native Component Node 模組，生成的程式碼會位於 `SampleApp/node_modules/my-fabric-component/android/build/generated/source/codegen` 路徑。而應用程式本身的程式碼則會生成在 `android/app/build/generated/source/codegen` 資料夾中。

#### 生成的程式碼

執行上述 gradle 指令後，你可以在 `SampleApp/android/app/build` 資料夾中找到 codegen 生成的程式碼。其結構如下：

```
build
└── generated
    └── source
        └── codegen
            ├── java
            │   └── com
            │       ├── facebook
            │       │   └── react
            │       │       └── viewmanagers
            │       │           ├── <nativeComponent>ManagerDelegate.java
            │       │           └── <nativeComponent>ManagerInterface.java
            │       └── sampleapp
            │           └── NativeLocalStorageSpec.java
            ├── jni
            │   ├── <codegenConfig.name>-generated.cpp
            │   ├── <codegenConfig.name>.h
            │   ├── CMakeLists.txt
            │   └── react
            │       └── renderer
            │           └── components
            │               └── <codegenConfig.name>
            │                   ├── <codegenConfig.name>JSI-generated.cpp
            │                   ├── <codegenConfig.name>.h
            │                   ├── ComponentDescriptors.cpp
            │                   ├── ComponentDescriptors.h
            │                   ├── EventEmitters.cpp
            │                   ├── EventEmitters.h
            │                   ├── Props.cpp
            │                   ├── Props.h
            │                   ├── ShadowNodes.cpp
            │                   ├── ShadowNodes.h
            │                   ├── States.cpp
            │                   └── States.h
            └── schema.json
```

生成的程式碼分為兩個資料夾：

- `java`：包含平台特定的程式碼
- `jni`：包含讓 JavaScript 與 Java 正確互動所需的 C++ 程式碼

在 `java` 資料夾中，你可以在 `com/facebook/viewmanagers` 子資料夾找到 Fabric Native Component 的生成程式碼。

- the `<nativeComponent>ManagerDelegate.java` contains the methods that the `ViewManager` can call on the custom Native Component
- the `<nativeComponent>ManagerInterface.java` contains the interface of the `ViewManager`.

而在 `codegenConfig.android.javaPackageName` 設定的名稱對應資料夾中，則可找到 Turbo Native Module 需實作的抽象類別，以執行其任務。

最後，在 `jni` 資料夾中，存放了所有連接 JavaScript 與 Android 的樣板程式碼。

- `<codegenConfig.name>.h`：包含自訂 C++ Turbo Native Modules 的介面
- `<codegenConfig.name>-generated.cpp`：包含自訂 C++ Turbo Native Modules 的黏合程式碼
- `react/renderer/components/<codegenConfig.name>`：此資料夾包含自訂元件所需的所有黏合程式碼

This structure has been generated by using the value `all` for the `codegenConfig.type` field. If you use the value `modules`, expect to see no `react/renderer/components/` folder. If you use the value `components`, expect not to see any of the other files.

### iOS

**Codegen** for iOS relies on some Node scripts that are invoked during the build process. The scripts are located in the `SampleApp/node_modules/react-native/scripts/` folder.

主要腳本為 `generate-codegen-artifacts.js`。要呼叫此腳本，可從應用程式的根目錄執行以下指令：

```bash
node node_modules/react-native/scripts/generate-codegen-artifacts.js

Usage: generate-codegen-artifacts.js -p [path to app] -t [target platform] -o [output path]

Options:
      --help            Show help                                      [boolean]
      --version         Show version number                            [boolean]
  -p, --path            Path to the React Native project root.        [required]
  -t, --targetPlatform  Target platform. Supported values: "android", "ios",
                        "all".                                        [required]
  -o, --outputPath      Path where generated artifacts will be output to.
```

其中：

- `--path` 為應用程式根目錄的路徑
- `--outputPath` 為 **Codegen** 寫入生成檔案的目標位置
- `--targetPlatform` 為你想生成程式碼的目標平台

#### 生成的程式碼

使用以下參數執行腳本：

```shell
node node_modules/react-native/scripts/generate-codegen-artifacts.js \
    --path . \
    --outputPath ios/ \
    --targetPlatform ios
```

將在 `ios/build` 資料夾中生成以下檔案：

```
build
└── generated
    └── ios
        ├── <codegenConfig.name>
        │   ├── <codegenConfig.name>-generated.mm
        │   └── <codegenConfig.name>.h
        ├── <codegenConfig.name>JSI-generated.cpp
        ├── <codegenConfig.name>JSI.h
        ├── FBReactNativeSpec
        │   ├── FBReactNativeSpec-generated.mm
        │   └── FBReactNativeSpec.h
        ├── FBReactNativeSpecJSI-generated.cpp
        ├── FBReactNativeSpecJSI.h
        ├── RCTModulesConformingToProtocolsProvider.h
        ├── RCTModulesConformingToProtocolsProvider.mm
        └── react
            └── renderer
                └── components
                    └── <codegenConfig.name>
                        ├── ComponentDescriptors.cpp
                        ├── ComponentDescriptors.h
                        ├── EventEmitters.cpp
                        ├── EventEmitters.h
                        ├── Props.cpp
                        ├── Props.h
                        ├── RCTComponentViewHelpers.h
                        ├── ShadowNodes.cpp
                        ├── ShadowNodes.h
                        ├── States.cpp
                        └── States.h
```

部分生成的檔案會被 React Native 核心使用。此外還有一組檔案的名稱與你在 package.json 的 `codegenConfig.name` 欄位中指定的名稱相同。

- `<codegenConfig.name>/<codegenConfig.name>.h`：此檔案包含您自訂 iOS Turbo 原生模組的介面。
- `<codegenConfig.name>/<codegenConfig.name>-generated.mm`：此檔案包含您自訂 iOS Turbo 原生模組的黏合程式碼。
- `<codegenConfig.name>JSI.h`：此檔案包含您自訂 C++ Turbo 原生模組的介面。
- `<codegenConfig.name>JSI-generated.h`：此檔案包含您自訂 C++ Turbo 原生模組的黏合程式碼。
- `react/renderer/components/<codegenConfig.name>`：此資料夾包含您自訂元件所需的所有黏合程式碼。

This structure has been generated by using the value `all` for the `codegenConfig.type` field. If you use the value `modules`, expect to see no `react/renderer/components/` folder. If you use the value `components`, expect not to see any of the other files.