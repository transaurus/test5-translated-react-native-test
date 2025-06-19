# 使用 Codegen

本指南將教您如何：

- 配置 **Codegen**。
- 為每個平台手動調用它。

同時也描述了生成的程式碼結構。

## 先決條件

即使手動調用 **Codegen**，您始終需要一個 React Native 應用程式才能正確生成程式碼。

The **Codegen** process is tightly coupled with the build of the app, and the scripts are located in the `react-native` NPM package.

為本指南之便，請使用 React Native CLI 創建專案如下：

```bash
npx @react-native-community/cli@latest init SampleApp --version 0.76.0
```

**Codegen** 用於為您的自訂模組或元件生成黏合程式碼。有關如何創建它們的詳細資訊，請參閱 Turbo Native Modules 和 Fabric Native Components 的指南。

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
      "modulesConformingToProtocol": {
        "RCTImageURLLoader": [
          "<iOS-class-conforming-to-RCTImageURLLoader>",
          // example from react-native-camera-roll: https://github.com/react-native-cameraroll/react-native-cameraroll/blob/8a6d1b4279c76e5682a4b443e7a4e111e774ec0a/package.json#L118-L127
          // "RNCPHAssetLoader",
        ],
        "RCTURLRequestHandler": [
          "<iOS-class-conforming-to-RCTURLRequestHandler>",
          // example from react-native-camera-roll: https://github.com/react-native-cameraroll/react-native-cameraroll/blob/8a6d1b4279c76e5682a4b443e7a4e111e774ec0a/package.json#L118-L127
          // "RNCPHAssetUploader",
        ],
        "RCTImageDataDecoder": [
          "<iOS-class-conforming-to-RCTImageDataDecoder>",
          // we don't have a good example for this, but it works in the same way. Pass the name of the class that implements the RCTImageDataDecoder. It must be a Native Module.
        ]
      },
      "componentProvider": {
        "<componentName>": "<iOS-class-implementing-the-component>"
      },
    }
  },
```

您可以將以下片段添加到您的應用程式中並自訂各個欄位：

- `name:` 此名稱將用於創建包含您規格的文件。按照慣例，它應具有後綴 `Spec`，但這不是強制性的。
- `type`: 需要生成的程式碼類型。允許的值為 `modules`、`components`、`all`。
  - `modules:` 如果僅需為 Turbo Native Modules 生成程式碼，請使用此值。
  - `components:` 如果僅需為 Native Fabric Components 生成程式碼，請使用此值。
  - `all`: 如果同時擁有元件和模組，請使用此值。
- `jsSrcsDir`: 這是存放所有規格的根目錄。
- `android.javaPackageName`: 這是一個 Android 特定的設定，用於讓 **Codegen** 在自訂套件中生成文件。
- `ios`: `ios` 欄位是一個物件，可供應用程式開發者和函式庫維護者用來提供一些進階功能。以下所有欄位均為**可選**。
  - `ios.modulesConformingToProtocol`: React Native 提供了一些協議，原生模組可以實現這些協議以自訂某些行為。這些欄位允許您定義符合這些協議的模組列表。這些模組將在應用程式啟動時注入到 React Native 運行時中。
    - `ios.modulesConformingToProtocol.RCTImageURLLoader`: 實現 [`RCTImageURLLoader` 協議](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/Libraries/Image/RCTImageURLLoader.h#L26-L81) 的 iOS 原生模組列表。您需要傳遞實現 `RCTImageURLLoader` 的 iOS 類別名稱。它們必須是 Native Modules。
    - `ios.modulesConformingToProtocol.RCTURLRequestHandler`: 實現 [`RCTURLRequestHandler` 協議](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/React/Base/RCTURLRequestHandler.h#L11-L52) 的 iOS 原生模組列表。您需要傳遞實現 `RCTURLRequestHandler` 的 iOS 類別名稱。它們必須是 Native Modules。
    - `ios.modulesConformingToProtocol.RCTImageDataDecoder`: 實現 [`RCTImageDataDecoder` 協議](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/Libraries/Image/RCTImageDataDecoder.h#L15-L53) 的 iOS 原生模組列表。您需要傳遞實現 `RCTImageDataDecoder` 的 iOS 類別名稱。它們必須是 Native Modules。
  - `ios.componentProvider`: 此欄位是一個映射，用於生成自訂 JS React 元件與實現它的原生類別之間的關聯。映射的鍵是元件的 JS 名稱（例如 `TextInput`），值是實現該元件的 iOS 類別（例如 `RCTTextInput`）。

當 **Codegen** 運行時，它會搜尋應用程式的所有依賴項，尋找符合特定慣例的 JS 檔案，並生成所需的程式碼：

- Turbo Native Modules 要求規格檔案的前綴為 `Native`。例如，`NativeLocalStorage.ts` 是一個有效的規格檔案名稱。
- Native Fabric Components 要求規格檔案的後綴為 `NativeComponent`。例如，`WebViewNativeComponent.ts` 是一個有效的規格檔案名稱。

## 運行 **Codegen**

The rest of this guide assumes that you have a Native Turbo Module, a Native Fabric Component or both already set up in your project. We also assume that you have valid specification files in the `jsSrcsDir` specified in the `package.json`.

### Android

**Codegen** for Android is integrated with the React Native Gradle Plugin (RNGP). The RNGP contains a task that can be invoked that reads the configurations defined in the `package.json` file and execute **Codegen**. To run the gradle task, first navigate inside the `android` folder of your project. Then run:

```bash
./gradlew generateCodegenArtifactsFromSchema
```

此任務會在應用程式的所有導入專案（應用程式及其所有連結的 node 模組）上調用 `generateCodegenArtifactsFromSchema` 命令。它會在相應的 `node_modules/<dependency>` 資料夾中生成程式碼。例如，如果你有一個 Node 模組名為 `my-fabric-component` 的 Fabric Native Component，生成的程式碼位於 `SampleApp/node_modules/my-fabric-component/android/build/generated/source/codegen` 路徑。對於應用程式本身，程式碼會生成在 `android/app/build/generated/source/codegen` 資料夾中。

#### 生成的程式碼

運行上述 gradle 命令後，你可以在 `SampleApp/android/app/build` 資料夾中找到生成的程式碼。結構如下：

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

- `java` 包含平台特定的程式碼
- `jni` 包含讓 JS 和 Java 正確互動所需的 C++ 程式碼。

在 `java` 資料夾中，你可以在 `com/facebook/viewmanagers` 子資料夾中找到 Fabric Native Component 的生成程式碼。

- the `<nativeComponent>ManagerDelegate.java` contains the methods that the `ViewManager` can call on the custom Native Component
- the `<nativeComponent>ManagerInterface.java` contains the interface of the `ViewManager`.

在 `codegenConfig.android.javaPackageName` 設定的名稱資料夾中，你可以找到 Turbo Native Module 必須實現的抽象類別，以執行其任務。

最後，在 `jni` 資料夾中，包含了所有連接 JS 和 Android 的樣板程式碼。

- `<codegenConfig.name>.h` 包含自定義 C++ Turbo Native Modules 的介面。
- `<codegenConfig.name>-generated.cpp` 包含自定義 C++ Turbo Native Modules 的粘合程式碼。
- `react/renderer/components/<codegenConfig.name>`: 此資料夾包含自定義元件所需的所有粘合程式碼。

This structure has been generated by using the value `all` for the `codegenConfig.type` field. If you use the value `modules`, expect to see no `react/renderer/components/` folder. If you use the value `components`, expect not to see any of the other files.

### iOS

**Codegen** for iOS relies on some Node scripts that are invoked during the build process. The scripts are located in the `SampleApp/node_modules/react-native/scripts/` folder.

主要腳本是 `generate-codegen-artifacts.js`。要調用此腳本，你可以從應用程式的根資料夾運行以下命令：

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

- `--path` 參數指定應用程式的根目錄路徑。
- `--outputPath` 參數指定 **Codegen** 生成檔案的輸出位置。
- `--targetPlatform` 參數指定要生成程式碼的目標平台。

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

這些生成的檔案中，部分會被 React Native 核心使用。其餘檔案則包含你在 package.json 的 `codegenConfig.name` 欄位中指定的名稱。

- `<codegenConfig.name>/<codegenConfig.name>.h`：此檔案包含自訂 iOS Turbo Native Modules 的介面。
- `<codegenConfig.name>/<codegenConfig.name>-generated.mm`：此檔案包含自訂 iOS Turbo Native Modules 的黏合程式碼。
- `<codegenConfig.name>JSI.h`：此檔案包含自訂 C++ Turbo Native Modules 的介面。
- `<codegenConfig.name>JSI-generated.h`：此檔案包含自訂 C++ Turbo Native Modules 的黏合程式碼。
- `react/renderer/components/<codegenConfig.name>`：此資料夾包含自訂元件所需的所有黏合程式碼。

This structure has been generated by using the value `all` for the `codegenConfig.type` field. If you use the value `modules`, expect to see no `react/renderer/components/` folder. If you use the value `components`, expect not to see any of the other files.