## 核心概念

將 React Native 元件整合至 iOS 應用程式的關鍵步驟如下：

1. 設定 React Native 的依賴項與目錄結構。
2. 了解應用程式中將使用的 React Native 元件。
3. 使用 CocoaPods 添加這些元件作為依賴項。
4. 以 JavaScript 開發 React Native 元件。
5. 在 iOS 應用程式中加入 `RCTRootView`，此視圖將作為 React Native 元件的容器。
6. 啟動 React Native 伺服器並執行原生應用程式。
7. 驗證應用程式中 React Native 的部分是否如預期運作。

## 必要條件

請遵循[環境設定指南](environment-setup)中的 React Native CLI 快速入門，配置開發環境以建構 iOS 版的 React Native 應用程式。

### 1. 設定目錄結構

為確保流程順暢，請為整合的 React Native 專案建立新資料夾，並將現有的 iOS 專案複製到 `/ios` 子資料夾中。

### 2. 安裝 JavaScript 依賴項

進入專案的根目錄，並建立包含以下內容的新 `package.json` 檔案：

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

接著，請確認已[安裝 yarn 套件管理工具](https://yarnpkg.com/lang/en/docs/install/)。

安裝 `react` 與 `react-native` 套件。開啟終端機或命令提示字元，導航至包含 `package.json` 檔案的目錄並執行：

```shell
$ yarn add react-native
```

這將輸出類似以下的訊息（請在 yarn 的輸出中向上滾動以查看）：

> 警告："react-native@0.52.2" 有未滿足的對等依賴項 "react@16.2.0"。

這是正常的，表示我們還需要安裝 React：

```shell
$ yarn add react@version_printed_above
```

Yarn 已建立新的 `/node_modules` 資料夾，此資料夾儲存了建構專案所需的所有 JavaScript 依賴項。

請將 `node_modules/` 加入你的 `.gitignore` 檔案中。

### 3. 安裝 CocoaPods

[CocoaPods](http://cocoapods.org) 是 iOS 與 macOS 開發的套件管理工具。我們使用它將實際的 React Native 框架程式碼本地化加入至當前專案中。

建議使用 [Homebrew](http://brew.sh/) 安裝 CocoaPods。

```shell
$ brew install cocoapods
```

> 技術上來說可以不使用 CocoaPods，但這將需要手動添加函式庫與連結器，使流程過於複雜。

## 將 React Native 加入你的應用程式

假設[待整合的應用程式](https://github.com/JoelMarcey/swift-2048)是 [2048](https://en.wikipedia.org/wiki/2048_%28video_game%29) 遊戲。以下是未整合 React Native 時原生應用程式的主選單畫面。

![整合前的畫面](/docs/assets/react-native-existing-app-integration-ios-before.png)

### Xcode 的命令列工具

安裝命令列工具。在 Xcode 選單中選擇「偏好設定...」，進入「位置」面板，並從命令列工具的下拉選單中選擇最新版本進行安裝。

![Xcode 命令列工具](/docs/assets/GettingStartedXcodeCommandLineTools.png)

### 配置 CocoaPods 依賴項

在將 React Native 整合至應用程式之前，需決定要整合 React Native 框架的哪些部分。我們將使用 CocoaPods 來指定應用程式將依賴的這些「子規格」。

The list of supported `subspec`s is available in [`/node_modules/react-native/React.podspec`](https://github.com/facebook/react-native/blob/0.71-stable/React.podspec). They are generally named by functionality. For example, you will generally always want the `Core` `subspec`. That will get you the `AppRegistry`, `StyleSheet`, `View` and other core React Native libraries. If you want to add the React Native `Text` library (e.g., for `<Text>` elements), then you will need the `RCTText` `subspec`. If you want the `Image` library (e.g., for `<Image>` elements), then you will need the `RCTImage` `subspec`.

You can specify which `subspec`s your app will depend on in a `Podfile` file. The easiest way to create a `Podfile` is by running the CocoaPods `init` command in the `/ios` subfolder of your project:

```shell
$ pod init
```

The `Podfile` will contain a boilerplate setup that you will tweak for your integration purposes.

> The `Podfile` version changes depending on your version of `react-native`. Refer to https://react-native-community.github.io/upgrade-helper/ for the specific version of `Podfile` you should be using.

最終，您的 `Podfile` 應類似以下範本：
[Podfile 範本](https://github.com/facebook/react-native/blob/0.71-stable/template/ios/Podfile)

建立 `Podfile` 後，即可安裝 React Native 的 pod。

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

> 若失敗並出現提及 `xcrun` 的錯誤，請確認在 Xcode 的 **Preferences > Locations** 中已指定 Command Line Tools。

> If you get a warning such as "_The `swift-2048 [Debug]` target overrides the `FRAMEWORK_SEARCH_PATHS` build setting defined in `Pods/Target Support Files/Pods-swift-2048/Pods-swift-2048.debug.xcconfig`. This can lead to problems with the CocoaPods installation_", then make sure the `Framework Search Paths` in `Build Settings` for both `Debug` and `Release` only contain `$(inherited)`.

### 程式碼整合

現在我們將實際修改原生 iOS 應用程式以整合 React Native。以 2048 範例應用為例，我們將加入一個以 React Native 實作的「高分」畫面。

#### React Native 元件

我們要編寫的第一段程式碼是新「高分」畫面的實際 React Native 程式碼，它將被整合到我們的應用程式中。

##### 1. 建立 `index.js` 檔案

首先，在 React Native 專案的根目錄建立一個空的 `index.js` 檔案。

`index.js` 是 React Native 應用程式的起點，且為必要檔案。它可以是一個小檔案，僅 `require` 其他屬於 React Native 元件或應用程式的檔案，也可以包含所需的所有程式碼。在此範例中，我們會將所有程式碼放在 `index.js` 中。

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

> `RNHighScores` 是模組名稱，當您從 iOS 應用程式中加入視圖到 React Native 時會使用此名稱。

#### 魔法所在：`RCTRootView`

現在您的 React Native 元件已透過 `index.js` 建立，接下來需要將該元件加入新的或現有的 `ViewController` 中。最簡單的方式是選擇性地建立通往元件的路徑，然後將該元件加入現有的 `ViewController`。

我們將把 React Native 元件與 `ViewController` 中的新原生視圖綁定，該視圖將實際包含稱為 `RCTRootView` 的元件。

##### 1. 建立事件路徑

您可以在主遊戲選單中新增一個連結，導向「高分」的 React Native 頁面。

![事件路徑](/docs/assets/react-native-add-react-native-integration-link.png)

##### 2. 事件處理器

我們現在將從選單連結新增一個事件處理器。一個方法將被新增到您應用程式的主要 `ViewController` 中。這就是 `RCTRootView` 發揮作用的地方。

當您建構 React Native 應用程式時，您會使用 [Metro 打包工具][metro] 來建立一個 `index.bundle`，該檔案將由 React Native 伺服器提供。在 `index.bundle` 內部將是我們的 `RNHighScore` 模組。因此，我們需要將 `RCTRootView` 指向 `index.bundle` 資源的位置（透過 `NSURL`）並將其與模組綁定。

為了除錯目的，我們將記錄事件處理器被調用的情況。然後，我們將建立一個字串，包含存在於 `index.bundle` 中的 React Native 程式碼的位置。最後，我們將建立主要的 `RCTRootView`。請注意，我們提供了 `RNHighScores` 作為 `moduleName`，這是我們在撰寫 React Native 元件程式碼時[上方](#the-react-native-component)建立的。

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

> Note that `RCTRootView bundleURL` starts up a new JSC VM. To save resources and simplify the communication between RN views in different parts of your native app, you can have multiple views powered by React Native that are associated with a single JS runtime. To do that, instead of using `RCTRootView bundleURL`, use [`RCTBridge initWithBundleURL`](https://github.com/facebook/react-native/blob/0.71-stable/React/Base/RCTBridge.h#L89) to create a bridge and then use `RCTRootView initWithBridge`.

> When moving your app to production, the `NSURL` can point to a pre-bundled file on disk via something like `let mainBundle = NSBundle(URLForResource: "main" withExtension:"jsbundle")`. You can use the `react-native-xcode.sh` script in `node_modules/react-native/scripts/` to generate that pre-bundled file.

##### 3. 連接

將主選單中的新連結連接到新增的事件處理器方法。

![事件路徑](/docs/assets/react-native-add-react-native-integration-wire-up.png)

> 其中一種較簡單的方法是打開故事板中的視圖，然後右鍵點擊新連結。選擇類似 `Touch Up Inside` 的事件，將其拖曳到故事板，然後從提供的列表中選擇建立的方法。

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

##### 1. 新增 App Transport Security 例外

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

> 應用程式傳輸安全機制對您的使用者有益。請確保在發佈應用程式至生產環境前重新啟用它。

##### 2. 執行打包伺服器

要運行您的應用程式，首先需要啟動開發伺服器。為此，請在您的 React Native 專案根目錄下執行以下命令：

```shell
$ npm start
```

##### 3. 運行應用程式

如果您使用 Xcode 或偏好的編輯器，照常建置並運行您的原生 iOS 應用程式。或者，您也可以透過命令列運行應用程式：

```
# From the root of your project
$ npx react-native run-ios
```

在我們的範例應用程式中，您應該會看到「高分榜」的連結，點擊後將看到 React Native 元件的渲染結果。

以下是 _原生_ 應用程式的主畫面：

![主畫面](/docs/assets/react-native-add-react-native-integration-example-home-screen.png)

以下是 _React Native_ 的高分榜畫面：

![高分榜](/docs/assets/react-native-add-react-native-integration-example-high-scores.png)

> 若在運行應用程式時遇到模組解析問題，請參閱 [此 GitHub 議題](https://github.com/facebook/react-native/issues/4968) 以獲取資訊與可能的解決方案。[此評論](https://github.com/facebook/react-native/issues/4968#issuecomment-220941717) 似乎是目前最新的解決方法。

### 接下來呢？

至此，您可以繼續照常開發應用程式。參考我們的 [除錯指南](debugging) 和 [部署指南](running-on-device) 以了解更多關於 React Native 開發的資訊。

[metro]: https://metrobundler.dev/