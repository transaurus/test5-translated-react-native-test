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

請先遵循[設定開發環境](set-up-your-environment)指南及[無框架使用 React Native](getting-started-without-a-framework)的說明，配置 iOS 版 React Native 應用的開發環境。

### 1. 設定目錄結構

為確保流程順暢，請為整合專案新建資料夾，並將現有 iOS 專案複製至 `/ios` 子目錄。

### 2. 安裝 JavaScript 依賴項

進入專案根目錄，建立包含以下內容的 `package.json` 檔案：

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

接著請確認已安裝 [yarn 套件管理工具](https://yarnpkg.com/lang/en/docs/install/)。

安裝 `react` 與 `react-native` 套件。開啟終端機，導航至含 `package.json` 的目錄後執行：

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

安裝過程將輸出類似以下訊息（請向上滾動查看完整輸出）：

> 警告："`react-native@0.52.2`" 缺少必要依賴項 "`react@16.2.0`"

此為正常現象，表示需額外安裝 React：

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

安裝完成後會自動建立 `/node_modules` 資料夾，此目錄存放專案所需的所有 JavaScript 依賴項。

請將 `node_modules/` 加入 `.gitignore` 檔案。

### 3. 安裝 CocoaPods

[CocoaPods](https://cocoapods.org) 是 iOS/macOS 開發的套件管理工具，我們用它將 React Native 框架代碼本地化至現有專案。

建議透過 [Homebrew](https://brew.sh/) 安裝 CocoaPods：

```shell
brew install cocoapods
```

> 技術上雖可不使用 CocoaPods，但需手動處理函式庫與連結器設定，將大幅增加流程複雜度。

## 將 React Native 添加至應用

假設[待整合應用](https://github.com/austinzheng/iOS-2048)為 [2048](https://en.wikipedia.org/wiki/2048_%28video_game%29) 遊戲。下圖為未整合 React Native 時的原生應用主選單：

![整合前的介面](/docs/assets/react-native-existing-app-integration-ios-before.png)

### 安裝 Xcode 命令列工具

請安裝命令列工具。在 Xcode 選單中選擇 **Settings... (或 Preferences...)**，進入 Locations 面板後，從 Command Line Tools 下拉選單選擇最新版本進行安裝。

![Xcode 命令列工具](/docs/assets/GettingStartedXcodeCommandLineTools.png)

### 配置 CocoaPods 依賴項

在整合 React Native 前，需先決定要引入哪些框架功能模組（subspecs）。我們將透過 CocoaPods 指定應用所需的模組。

The list of supported `subspec`s is available in [`/node_modules/react-native/React.podspec`](https://github.com/facebook/react-native/blob/main/packages/react-native/React.podspec). They are generally named by functionality. For example, you will generally always want the `Core` `subspec`. That will get you the `AppRegistry`, `StyleSheet`, `View` and other core React Native libraries. If you want to add the React Native `Text` library (e.g., for `<Text>` elements), then you will need the `RCTText` `subspec`. If you want the `Image` library (e.g., for `<Image>` elements), then you will need the `RCTImage` `subspec`.

You can specify which `subspec`s your app will depend on in a `Podfile` file. The easiest way to create a `Podfile` is by running the CocoaPods `init` command in the `/ios` subfolder of your project:

```shell
pod init
```

The `Podfile` will contain a boilerplate setup that you will tweak for your integration purposes.

> The `Podfile` version changes depending on your version of `react-native`. Refer to https://react-native-community.github.io/upgrade-helper/ for the specific version of `Podfile` you should be using.

最終，您的 `Podfile` 應類似以下內容：

```
# The target name is most likely the name of your project.
target 'NumberTileGame' do

  # Your 'node_modules' directory is probably in the root of your project,
  # but if not, adjust the `:path` accordingly
  pod 'FBLazyVector', :path => "../node_modules/react-native/Libraries/FBLazyVector"
  pod 'FBReactNativeSpec', :path => "../node_modules/react-native/Libraries/FBReactNativeSpec"
  pod 'RCTRequired', :path => "../node_modules/react-native/Libraries/RCTRequired"
  pod 'RCTTypeSafety', :path => "../node_modules/react-native/Libraries/TypeSafety"
  pod 'React', :path => '../node_modules/react-native/'
  pod 'React-Core', :path => '../node_modules/react-native/'
  pod 'React-CoreModules', :path => '../node_modules/react-native/React/CoreModules'
  pod 'React-Core/DevSupport', :path => '../node_modules/react-native/'
  pod 'React-RCTActionSheet', :path => '../node_modules/react-native/Libraries/ActionSheetIOS'
  pod 'React-RCTAnimation', :path => '../node_modules/react-native/Libraries/NativeAnimation'
  pod 'React-RCTBlob', :path => '../node_modules/react-native/Libraries/Blob'
  pod 'React-RCTImage', :path => '../node_modules/react-native/Libraries/Image'
  pod 'React-RCTLinking', :path => '../node_modules/react-native/Libraries/LinkingIOS'
  pod 'React-RCTNetwork', :path => '../node_modules/react-native/Libraries/Network'
  pod 'React-RCTSettings', :path => '../node_modules/react-native/Libraries/Settings'
  pod 'React-RCTText', :path => '../node_modules/react-native/Libraries/Text'
  pod 'React-RCTVibration', :path => '../node_modules/react-native/Libraries/Vibration'
  pod 'React-Core/RCTWebSocket', :path => '../node_modules/react-native/'

  pod 'React-cxxreact', :path => '../node_modules/react-native/ReactCommon/cxxreact'
  pod 'React-jsi', :path => '../node_modules/react-native/ReactCommon/jsi'
  pod 'React-jsiexecutor', :path => '../node_modules/react-native/ReactCommon/jsiexecutor'
  pod 'React-jsinspector', :path => '../node_modules/react-native/ReactCommon/jsinspector'
  pod 'ReactCommon/callinvoker', :path => "../node_modules/react-native/ReactCommon"
  pod 'ReactCommon/turbomodule/core', :path => "../node_modules/react-native/ReactCommon"
  pod 'Yoga', :path => '../node_modules/react-native/ReactCommon/yoga'

  pod 'DoubleConversion', :podspec => '../node_modules/react-native/third-party-podspecs/DoubleConversion.podspec'
  pod 'glog', :podspec => '../node_modules/react-native/third-party-podspecs/glog.podspec'
  pod 'Folly', :podspec => '../node_modules/react-native/third-party-podspecs/Folly.podspec'

end
```

建立 `Podfile` 後，即可安裝 React Native 的 pod。

```shell
$ pod install
```

您應會看到類似以下的輸出：

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

> 若因 `xcrun` 相關錯誤導致失敗，請確認 Xcode 中 **Settings... (或 Preferences...) > Locations** 下的 Command Line Tools 已正確指定。

### 程式碼整合

現在我們將實際修改原生 iOS 應用程式以整合 React Native。以 2048 範例應用為例，我們將加入一個以 React Native 實作的「高分榜」畫面。

#### React Native 元件

我們要編寫的第一段程式碼是新「高分榜」畫面的實際 React Native 程式碼，這將被整合至我們的應用程式中。

##### 1. 建立 `index.js` 檔案

首先，在 React Native 專案的根目錄建立一個空的 `index.js` 檔案。

`index.js` 是 React Native 應用程式的起點，且為必要檔案。它可以是一個小檔案，僅 `require` 其他屬於您 React Native 元件或應用程式的檔案，也可以包含所有所需程式碼。本例中，我們會將所有內容放在 `index.js`。

##### 2. 加入 React Native 程式碼

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

> `RNHighScores` 是模組名稱，當您從 iOS 應用程式中加入視圖至 React Native 時會使用此名稱。

#### 關鍵步驟：`RCTRootView`

現在您的 React Native 元件已透過 `index.js` 建立，接下來需將該元件加入新的或現有的 `ViewController`。最簡單的方式是選擇性地建立通往元件的路徑，然後將元件加入現有的 `ViewController`。

我們將把 React Native 元件與 `ViewController` 中的新原生視圖綁定，該視圖實際會包含稱為 `RCTRootView` 的元件。

##### 1. 建立事件路徑

您可以在主遊戲選單中加入新連結，導向 React Native 的「高分榜」頁面。

![事件路徑](/docs/assets/react-native-add-react-native-integration-link.png)

##### 2. 事件處理器

我們現在將為選單連結添加事件處理器。一個方法會被加入應用程式的主要 `ViewController` 中，這裡就是 `RCTRootView` 發揮作用的地方。

當你建構 React Native 應用程式時，會使用 [Metro 打包工具][metro] 來建立一個由 React Native 伺服器提供的 `index.bundle` 檔案。在這個 `index.bundle` 中會包含我們的 `RNHighScore` 模組。因此，我們需要將 `RCTRootView` 指向 `index.bundle` 資源的位置（透過 `NSURL`）並將其與模組綁定。

為了除錯目的，我們會記錄事件處理器被調用的情況。接著，我們會建立一個字串，指向存在於 `index.bundle` 中的 React Native 程式碼位置。最後，我們會建立主要的 `RCTRootView`。請注意我們如何提供 `RNHighScores` 作為 `moduleName`，這是在[前面](#the-react-native-component)撰寫 React Native 元件程式碼時所建立的。

首先 `import` 引入 `RCTRootView` 標頭檔。

```objectivec
#import <React/RCTRootView.h>
```

> 此處的 `initialProperties` 僅為示範用途，用來為高分畫面提供一些資料。在我們的 React Native 元件中，會使用 `this.props` 來存取這些資料。

```objectivec
- (IBAction)highScoreButtonPressed:(id)sender {
    NSLog(@"High Score Button Pressed");
    NSURL *jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.bundle?platform=ios"];

    RCTRootView *rootView =
      [[RCTRootView alloc] initWithBundleURL: jsCodeLocation
                                  moduleName: @"RNHighScores"
                           initialProperties:
                             @{
                               @"scores" : @[
                                 @{
                                   @"name" : @"Alex",
                                   @"value": @"42"
                                  },
                                 @{
                                   @"name" : @"Joel",
                                   @"value": @"10"
                                 }
                               ]
                             }
                               launchOptions: nil];
    UIViewController *vc = [[UIViewController alloc] init];
    vc.view = rootView;
    [self presentViewController:vc animated:YES completion:nil];
}
```

> 請注意 `RCTRootView initWithBundleURL` 會啟動一個新的 JSC 虛擬機器。為了節省資源並簡化原生應用中不同部分 React Native 視圖間的溝通，你可以讓多個由 React Native 驅動的視圖共用單一 JS 執行環境。要做到這點，不要使用 `[RCTRootView alloc] initWithBundleURL`，而是使用 [`RCTBridge initWithBundleURL`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTBridge.h#L94) 建立橋接器，然後使用 `RCTRootView initWithBridge`。

> When moving your app to production, the `NSURL` can point to a pre-bundled file on disk via something like `[[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];`. You can use the `react-native-xcode.sh` script in `node_modules/react-native/scripts/` to generate that pre-bundled file.

##### 3. 連接設定

將主選單中的新連結與新添加的事件處理器方法連接起來。

![事件路徑](/docs/assets/react-native-add-react-native-integration-wire-up.png)

> 較簡單的做法是在故事板中打開視圖，右鍵點擊新連結。選擇類似 `Touch Up Inside` 的事件，將其拖曳到故事板，然後從提供的列表中選擇已建立的方法。

### 測試整合效果

你現在已完成將 React Native 與現有應用程式整合的所有基本步驟。接下來我們將啟動 [Metro 打包工具][metro] 來建構 `index.bundle` 套件，並啟動運行在 `localhost` 的伺服器來提供該套件。

##### 1. 添加應用程式傳輸安全性例外

Apple 已禁止隱式的明文 HTTP 資源載入。因此我們需要在專案的 `Info.plist`（或同等檔案）中添加以下內容。

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

> 應用程式傳輸安全性對用戶有好處。請確保在將應用程式發布至生產環境前重新啟用它。

##### 2. 執行打包工具

要運行你的應用程式，首先需要啟動開發伺服器。為此，請在 React Native 專案的根目錄下執行以下命令：

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

##### 3. 執行應用程式

如果你使用 Xcode 或偏好的編輯器，像平常一樣建置並運行你的原生 iOS 應用程式。或者，你也可以透過命令列執行應用程式：

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

在我們的範例應用程式中，您會看到「高分排行榜」的連結，點擊後將看到 React Native 元件的渲染畫面。

Here is the _native_ application home screen:

![主畫面](/docs/assets/react-native-add-react-native-integration-example-home-screen.png)

Here is the _React Native_ high score screen:

![高分排行榜](/docs/assets/react-native-add-react-native-integration-example-high-scores.png)

> 若執行應用程式時遇到模組解析問題，請參閱 [GitHub 議題](https://github.com/facebook/react-native/issues/4968)了解解決方案。[此評論](https://github.com/facebook/react-native/issues/4968#issuecomment-220941717)可能是最新的解決方法。

### 接下來？

至此您可繼續常規開發流程。更多 React Native 開發技巧，請參考[除錯指南](debugging)與[部署文件](running-on-device)。

[metro]: https://metrobundler.dev/