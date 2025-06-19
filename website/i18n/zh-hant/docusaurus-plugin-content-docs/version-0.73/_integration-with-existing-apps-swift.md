import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 核心概念

將 React Native 元件整合至 iOS 應用的關鍵在於：

1. 設定 React Native 依賴項與目錄結構
2. 確認應用中將使用的 React Native 元件
3. 使用 CocoaPods 添加這些元件作為依賴項
4. 用 JavaScript 開發 React Native 元件
5. 在 iOS 應用中添加 `RCTRootView` 作為 React Native 元件的容器
6. 啟動 React Native 伺服器並運行原生應用
7. 驗證 React Native 功能是否正常運作

## 必要條件

請依照[環境設定指南](environment-setup)中的 React Native CLI 快速入門，配置 iOS 版 React Native 應用的開發環境。

### 1. 設定目錄結構

為確保流程順暢，請為整合專案新建資料夾，並將現有 iOS 專案複製到 `/ios` 子資料夾中。

### 2. 安裝 JavaScript 依賴項

進入專案根目錄，創建包含以下內容的 `package.json` 檔案：

```
{
  "name": "MyReactNativeApp",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "yarn react-native start"
  }
}
```

接著安裝 `react` 和 `react-native` 套件。開啟終端機並導航至含 `package.json` 的目錄，執行：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm install react-native
```

</TabItem>
<TabItem value="yarn">

```shell
yarn add react-native
```

</TabItem>
</Tabs>

這將顯示類似以下的訊息（請向上滾動安裝命令輸出以查看）：

> 警告："`react-native@0.52.2`" 有未滿足的對等依賴 "`react@16.2.0`"。

此為正常現象，表示我們還需安裝 React：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm install react@version_printed_above
```

</TabItem>
<TabItem value="yarn">

```shell
yarn add react@version_printed_above
```

</TabItem>
</Tabs>

安裝過程會創建 `/node_modules` 資料夾，存放專案所需的所有 JavaScript 依賴項。

請將 `node_modules/` 加入 `.gitignore` 檔案。

### 3. 安裝 CocoaPods

[CocoaPods](https://cocoapods.org) 是 iOS 和 macOS 開發的套件管理工具，我們用它將 React Native 框架代碼本地化加入專案。

建議透過 [Homebrew](https://brew.sh/) 安裝 CocoaPods：

```shell
$ brew install cocoapods
```

> 技術上可不使用 CocoaPods，但這需要手動添加函式庫與連結器配置，會大幅增加流程複雜度。

## 將 React Native 加入應用

假設[待整合應用](https://github.com/JoelMarcey/swift-2048)是 [2048](https://en.wikipedia.org/wiki/2048_%28video_game%29) 遊戲。下圖為未整合 React Native 時的原生應用主選單：

![整合前的畫面](/docs/assets/react-native-existing-app-integration-ios-before.png)

### Xcode 命令行工具

請安裝命令行工具。在 Xcode 選單中選擇 **Settings... (或 Preferences...)**，進入 Locations 面板，從 Command Line Tools 下拉選單中選擇最新版本進行安裝。

![Xcode 命令行工具](/docs/assets/GettingStartedXcodeCommandLineTools.png)

### 配置 CocoaPods 依賴項

在整合 React Native 前，需決定要整合框架中的哪些部分。我們將使用 CocoaPods 指定應用依賴的「子規格」(subspecs)。

The list of supported `subspec`s is available in [`/node_modules/react-native/React.podspec`](https://github.com/facebook/react-native/blob/main/packages/react-native/React.podspec). They are generally named by functionality. For example, you will generally always want the `Core` `subspec`. That will get you the `AppRegistry`, `StyleSheet`, `View` and other core React Native libraries. If you want to add the React Native `Text` library (e.g., for `<Text>` elements), then you will need the `RCTText` `subspec`. If you want the `Image` library (e.g., for `<Image>` elements), then you will need the `RCTImage` `subspec`.

You can specify which `subspec`s your app will depend on in a `Podfile` file. The easiest way to create a `Podfile` is by running the CocoaPods `init` command in the `/ios` subfolder of your project:

```shell
$ pod init
```

The `Podfile` will contain a boilerplate setup that you will tweak for your integration purposes.

> The `Podfile` version changes depending on your version of `react-native`. Refer to https://react-native-community.github.io/upgrade-helper/ for the specific version of `Podfile` you should be using.

最終，您的 `Podfile` 應類似於以下內容：
[Podfile 範本](https://github.com/facebook/react-native/blob/main/packages/react-native/template/ios/Podfile)

建立 `Podfile` 後，您就可以安裝 React Native 的 pod 了。

```shell
$ pod install
```

您應該會看到類似以下的輸出：

```
Analyzing dependencies
Fetching podspec for `React` from `../node_modules/react-native`
Downloading dependencies
Installing React (0.62.0)
Generating Pods project
Integrating client project
Sending stats
Pod installation complete! There are 3 dependencies from the Podfile and 1 total pod installed.
```

> 如果因提及 `xcrun` 的錯誤而失敗，請確保在 Xcode 的 **設定...（或偏好設定...）> 位置** 中已指定命令列工具。

> If you get a warning such as "_The `swift-2048 [Debug]` target overrides the `FRAMEWORK_SEARCH_PATHS` build setting defined in `Pods/Target Support Files/Pods-swift-2048/Pods-swift-2048.debug.xcconfig`. This can lead to problems with the CocoaPods installation_", then make sure the `Framework Search Paths` in `Build Settings` for both `Debug` and `Release` only contain `$(inherited)`.

### 程式碼整合

現在我們將實際修改原生 iOS 應用程式以整合 React Native。對於我們的 2048 範例應用程式，我們將在 React Native 中加入一個「高分」畫面。

#### React Native 元件

我們將編寫的第一段程式碼是用於新「高分」畫面的實際 React Native 程式碼，該畫面將整合到我們的應用程式中。

##### 1. 建立 `index.js` 檔案

首先，在 React Native 專案的根目錄中建立一個空的 `index.js` 檔案。

`index.js` 是 React Native 應用程式的起點，且始終是必需的。它可以是一個小檔案，用於 `require` 其他屬於您的 React Native 元件或應用程式的檔案，也可以包含所需的所有程式碼。在我們的案例中，我們將把所有內容放在 `index.js` 中。

##### 2. 加入您的 React Native 程式碼

In your `index.js`, create your component. In our sample here, we will add a `<Text>` component within a styled `<View>`

```jsx
import React from 'react';
import {AppRegistry, StyleSheet, Text, View} from 'react-native';

const RNHighScores = ({scores}) => {
  const contents = scores.map(score => (
    <Text key={score.name}>
      {score.name}:{score.value}
      {'\n'}
    </Text>
  ));
  return (
    <View style={styles.container}>
      <Text style={styles.highScoresTitle}>
        2048 High Scores!
      </Text>
      <Text style={styles.scores}>{contents}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#FFFFFF',
  },
  highScoresTitle: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  scores: {
    textAlign: 'center',
    color: '#333333',
    marginBottom: 5,
  },
});

// Module name
AppRegistry.registerComponent('RNHighScores', () => RNHighScores);
```

> `RNHighScores` 是您的模組名稱，當您從 iOS 應用程式中加入一個視圖到 React Native 時將使用此名稱。

#### 魔法：`RCTRootView`

現在，您的 React Native 元件已透過 `index.js` 建立，您需要將該元件加入到一個新的或現有的 `ViewController` 中。最簡單的方法是選擇性地建立一個通往您的元件的事件路徑，然後將該元件加入到現有的 `ViewController` 中。

我們將把 React Native 元件與 `ViewController` 中的新原生視圖綁定，該視圖將實際包含稱為 `RCTRootView` 的元件。

##### 1. 建立事件路徑

您可以在主遊戲選單中新增一個連結，導向「高分」的 React Native 頁面。

![事件路徑](/docs/assets/react-native-add-react-native-integration-link.png)

##### 2. 事件處理器

我們現在將從選單連結新增一個事件處理器。一個方法將被新增到應用程式的主 `ViewController` 中。這就是 `RCTRootView` 發揮作用的地方。

當您建構 React Native 應用程式時，您會使用 [Metro 打包工具][metro] 來建立一個 `index.bundle`，該檔案將由 React Native 伺服器提供。在 `index.bundle` 內部將是我們的 `RNHighScore` 模組。因此，我們需要將 `RCTRootView` 指向 `index.bundle` 資源的位置（透過 `NSURL`）並將其與模組綁定。

為了除錯目的，我們將記錄事件處理器被調用的情況。然後，我們將建立一個字串，其中包含存在於 `index.bundle` 中的 React Native 程式碼的位置。最後，我們將建立主要的 `RCTRootView`。請注意，我們提供了 `RNHighScores` 作為 `moduleName`，這是我們在編寫 React Native 元件程式碼時[上方](#the-react-native-component)建立的。

首先 `import` `React` 函式庫。

```jsx
import React
```

> The `initialProperties` are here for illustration purposes so we have some data for our high score screen. In our React Native component, we will use `this.props` to get access to that data.

```swift
@IBAction func highScoreButtonTapped(sender : UIButton) {
  NSLog("Hello")
  let jsCodeLocation = URL(string: "http://localhost:8081/index.bundle?platform=ios")
  let mockData:NSDictionary = ["scores":
      [
          ["name":"Alex", "value":"42"],
          ["name":"Joel", "value":"10"]
      ]
  ]

  let rootView = RCTRootView(
      bundleURL: jsCodeLocation,
      moduleName: "RNHighScores",
      initialProperties: mockData as [NSObject : AnyObject],
      launchOptions: nil
  )
  let vc = UIViewController()
  vc.view = rootView
  self.present(vc, animated: true, completion: nil)
}
```

> Note that `RCTRootView bundleURL` starts up a new JSC VM. To save resources and simplify the communication between RN views in different parts of your native app, you can have multiple views powered by React Native that are associated with a single JS runtime. To do that, instead of using `RCTRootView bundleURL`, use [`RCTBridge initWithBundleURL`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTBridge.h#94) to create a bridge and then use `RCTRootView initWithBridge`.

> When moving your app to production, the `NSURL` can point to a pre-bundled file on disk via something like `let mainBundle = NSBundle(URLForResource: "main" withExtension:"jsbundle")`. You can use the `react-native-xcode.sh` script in `node_modules/react-native/scripts/` to generate that pre-bundled file.

##### 3. 連接

將主選單中的新連結連接到新增的事件處理器方法。

![事件路徑](/docs/assets/react-native-add-react-native-integration-wire-up.png)

> 其中一種較簡單的方法是打開故事板中的視圖，然後右鍵點擊新連結。選擇類似 `Touch Up Inside` 的事件，將其拖曳到故事板，然後從提供的列表中選擇已建立的方法。

##### 3. 視窗參考

在您的 AppDelegate.swift 檔案中新增一個視窗參考。最終，您的 AppDelegate 應該看起來類似這樣：

```swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    // Add window reference
    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        return true
    }

    ....
}
```

### 測試您的整合

您現在已經完成了將 React Native 與當前應用程式整合的所有基本步驟。現在我們將啟動 [Metro 打包工具][metro] 來建構 `index.bundle` 套件，並啟動運行在 `localhost` 上的伺服器來提供它。

##### 1. 新增應用程式傳輸安全性例外

Apple 已阻止隱式的明文 HTTP 資源加載。因此，我們需要在專案的 `Info.plist`（或等效）檔案中新增以下內容。

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>localhost</key>
        <dict>
            <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
            <true/>
        </dict>
    </dict>
</dict>
```

> 應用程式傳輸安全性對您的使用者有好處。請確保在發布應用程式之前重新啟用它。

##### 2. 執行封裝伺服器

要運行您的應用程式，首先需要啟動開發伺服器。為此，請在 React Native 專案的根目錄中執行以下命令：

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

##### 3. 運行應用程式

如果您使用 Xcode 或偏好的編輯器，請像平常一樣建置並運行您的原生 iOS 應用程式。或者，您也可以從 React Native 專案的根目錄使用以下命令來運行應用程式：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run ios
```

</TabItem>
<TabItem value="yarn">

```shell
yarn ios
```

</TabItem>
</Tabs>

在我們的範例應用程式中，您應該會看到「高分榜」的連結，點擊後將看到 React Native 元件的渲染結果。

以下是 _原生_ 應用程式的主畫面：

![主畫面](/docs/assets/react-native-add-react-native-integration-example-home-screen.png)

以下是 _React Native_ 高分榜畫面：

![高分榜](/docs/assets/react-native-add-react-native-integration-example-high-scores.png)

> 如果在運行應用程式時遇到模組解析問題，請參閱 [此 GitHub 議題](https://github.com/facebook/react-native/issues/4968) 以獲取資訊和可能的解決方案。[此評論](https://github.com/facebook/react-native/issues/4968#issuecomment-220941717) 似乎是目前最新的解決方案。

### 接下來呢？

此時您可以繼續如常開發應用程式。請參考我們的 [除錯](debugging) 和 [部署](running-on-device) 文件，以了解更多關於使用 React Native 的資訊。

[metro]: https://facebook.github.io/metro/