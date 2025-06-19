import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 核心概念

將 React Native 元件整合至 iOS 應用的關鍵在於：

1. 建立正確的目錄結構
2. 安裝必要的 NPM 依賴項
3. 在 Podfile 配置中添加 React Native
4. 為首個 React Native 畫面編寫 TypeScript 程式碼
5. 使用 `RCTRootView` 將 React Native 與 iOS 程式碼整合
6. 通過運行打包器並查看應用運行情況來測試整合效果

## 使用社群模板

遵循本指南時，建議您參考 [React Native 社群模板](https://github.com/react-native-community/template/)。該模板包含一個**最簡 iOS 應用**，可幫助您理解如何將 React Native 整合至現有 iOS 應用。

## 必要條件

請先完成 [設定開發環境](set-up-your-environment) 和 [不使用框架的 React Native](getting-started-without-a-framework) 指南，以配置用於構建 iOS 版 React Native 應用的開發環境。
本指南同時假設您已掌握 iOS 開發基礎知識，例如創建 `UIViewController` 和編輯 `Podfile` 文件。

### 1. 設定目錄結構

為確保流程順暢，請為整合式 React Native 專案新建文件夾，然後**將現有 iOS 專案移動**至 `/ios` 子文件夾。

## 2. 安裝 NPM 依賴項

進入根目錄並運行以下命令：

```shell
curl -O https://raw.githubusercontent.com/react-native-community/template/refs/heads/0.77-stable/template/package.json
```

This will copy the `package.json` [file from the Community template](https://github.com/react-native-community/template/blob/0.77-stable/template/package.json) to your project.

接著運行以下命令安裝 NPM 套件：

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

安裝過程會創建新的 `node_modules` 文件夾，該文件夾存儲構建專案所需的所有 JavaScript 依賴項。

請將 `node_modules/` 添加到您的 `.gitignore` 文件（可參考[社群默認版本](https://github.com/react-native-community/template/blob/0.77-stable/template/_gitignore)）。

### 3. 安裝開發工具

### Xcode 命令行工具

安裝命令行工具。在 Xcode 選單中選擇 **Settings... (或 Preferences...)**，進入 Locations 面板，從 Command Line Tools 下拉選單中選擇最新版本進行安裝。

![Xcode 命令行工具](/docs/assets/GettingStartedXcodeCommandLineTools.png)

### CocoaPods

[CocoaPods](https://cocoapods.org) 是 iOS 和 macOS 開發的套件管理工具。我們使用它將實際的 React Native 框架程式碼本地化添加到當前專案。

建議通過 [Homebrew](https://brew.sh/) 安裝 CocoaPods：

```shell
brew install cocoapods
```

## 4. 將 React Native 添加至應用

### 配置 CocoaPods

配置 CocoaPods 需要兩個文件：

- A **Gemfile** that defines which Ruby dependencies we need.
- A **Podfile** that defines how to properly install our dependencies.

對於 **Gemfile**，請進入專案根目錄並運行此命令：

```sh
curl -O https://raw.githubusercontent.com/react-native-community/template/refs/heads/0.77-stable/template/Gemfile
```

這會從模板下載 Gemfile。
同理，對於 **Podfile**，請進入專案的 `ios` 文件夾並運行：

```sh
curl -O https://raw.githubusercontent.com/react-native-community/template/refs/heads/0.77-stable/template/ios/Podfile
```

請使用社群範本作為 [Gemfile](https://github.com/react-native-community/template/blob/0.77-stable/template/Gemfile) 和 [Podfile](https://github.com/react-native-community/template/blob/0.77-stable/template/ios/Podfile) 的參考基準。

:::note
請記得修改 Podfile 中的[這一行](https://github.com/react-native-community/template/blob/0.77-stable/template/ios/Podfile#L17)和[這一行](https://github.com/react-native-community/template/blob/0.77-stable/template/ios/Podfile#L26)，使其符合你的應用程式名稱。

若你的應用程式沒有測試項目，請記得刪除[此區塊](https://github.com/react-native-community/template/blob/0.77-stable/template/ios/Podfile#L26-L29)。
:::

現在我們需要執行幾個額外指令來安裝 Ruby gems 和 Pods。
請導航至 `ios` 資料夾並執行以下指令：

```sh
bundle install
bundle exec pod install
```

第一個指令會安裝 Ruby 相依套件，第二個指令則會實際將 React Native 程式碼整合至你的應用程式，讓 iOS 檔案能導入 React Native 標頭檔。

## 5. 撰寫 TypeScript 程式碼

現在我們將實際修改原生 iOS 應用程式來整合 React Native。

我們要撰寫的第一段程式碼，是為即將整合至應用程式的新畫面所寫的 React Native 程式碼。

### 建立 `index.js` 檔案

首先，請在 React Native 專案的根目錄建立一個空的 `index.js` 檔案。

`index.js` 是 React Native 應用程式的進入點，且為必要檔案。它可以是一個僅用來 `import` 其他 React Native 元件或應用程式檔案的小檔案，也可以包含所需的所有程式碼。

我們的 `index.js` 應如下所示（此為[社群範本檔案參考](https://github.com/react-native-community/template/blob/0.77-stable/template/index.js)）：

```js
import {AppRegistry} from 'react-native';
import App from './App';

AppRegistry.registerComponent('HelloWorld', () => App);
```

### 建立 `App.tsx` 檔案

Let's create an `App.tsx` file. This is a [TypeScript](https://www.typescriptlang.org/) file that can have [JSX](<https://en.wikipedia.org/wiki/JSX_(JavaScript)>) expressions. It contains the root React Native component that we will integrate into our iOS application ([link](https://github.com/react-native-community/template/blob/0.77-stable/template/App.tsx)):

```tsx
import React from 'react';
import {
  SafeAreaView,
  ScrollView,
  StatusBar,
  StyleSheet,
  Text,
  useColorScheme,
  View,
} from 'react-native';

import {
  Colors,
  DebugInstructions,
  Header,
  ReloadInstructions,
} from 'react-native/Libraries/NewAppScreen';

function App(): React.JSX.Element {
  const isDarkMode = useColorScheme() === 'dark';

  const backgroundStyle = {
    backgroundColor: isDarkMode ? Colors.darker : Colors.lighter,
  };

  return (
    <SafeAreaView style={backgroundStyle}>
      <StatusBar
        barStyle={isDarkMode ? 'light-content' : 'dark-content'}
        backgroundColor={backgroundStyle.backgroundColor}
      />
      <ScrollView
        contentInsetAdjustmentBehavior="automatic"
        style={backgroundStyle}>
        <Header />
        <View
          style={{
            backgroundColor: isDarkMode
              ? Colors.black
              : Colors.white,
            padding: 24,
          }}>
          <Text style={styles.title}>Step One</Text>
          <Text>
            Edit <Text style={styles.bold}>App.tsx</Text> to
            change this screen and see your edits.
          </Text>
          <Text style={styles.title}>See your changes</Text>
          <ReloadInstructions />
          <Text style={styles.title}>Debug</Text>
          <DebugInstructions />
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  title: {
    fontSize: 24,
    fontWeight: '600',
  },
  bold: {
    fontWeight: '700',
  },
});

export default App;
```

此為[社群範本檔案參考](https://github.com/react-native-community/template/blob/0.77-stable/template/App.tsx)

## 5. 與 iOS 程式碼整合

我們現在需要添加一些原生程式碼，以啟動 React Native 運行時環境並告訴它渲染我們的 React 元件。

### 需求條件

React Native 預期與 `AppDelegate` 協同運作。以下部分假設你的 `AppDelegate` 如下所示：

<Tabs groupId="ios-language" queryString defaultValue={constants.defaultAppleLanguage} values={constants.appleLanguages}>
<TabItem value="objc">

```objc title="AppDelegate.m"
#import "AppDelegate.h"
#import "ViewController.h"

@interface AppDelegate ()

@end

@implementation AppDelegate {
  UIWindow *window;
}

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  window = [UIWindow new];
  window.rootViewController = [ViewController new];
  [window makeKeyAndVisible];
  return YES;
}

@end
```

</TabItem>
<TabItem value="swift">

```swift title="AppDelegate.swift"
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

  var window: UIWindow?

  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.
    window = UIWindow()
    window?.rootViewController = ViewController()
    window?.makeKeyAndVisible()
    return true
  }
}
```

</TabItem>
</Tabs>

### 更新 `AppDelegate` 類別

首先，我們需要擴展 `AppDelegate` 以繼承 React Native 提供的其中一個類別：`RCTAppDelegate`。

<Tabs groupId="ios-language" queryString defaultValue={constants.defaultAppleLanguage} values={constants.appleLanguages}>
<TabItem value="objc">

To achieve this, we have to modify the `AppDelegate.h` file and the `AppDelegate.m` files:

1. Open the `AppDelegate.h` files and modify it as it follows (See the official template's [AppDelegate.h](https://github.com/react-native-community/template/blob/0.76-stable/template/ios/HelloWorld/AppDelegate.h) as reference):

```diff title="AppDelegate.h changes"
#import <UIKit/UIKit.h>
+#import <React-RCTAppDelegate/RCTAppDelegate.h>

-@interface AppDelegate : UIResponder <UIApplicationDelegate>
+@interface AppDelegate : RCTAppDelegate


@end
```

2. Open the `AppDelegate.mm` file and modify it as it follows (See the official template's [AppDelegate.mm](https://github.com/react-native-community/template/blob/0.76-stable/template/ios/HelloWorld/AppDelegate.mm) as reference

```diff title="AppDelegate.mm"
#import "AppDelegate.h"
#import "ViewController.h"
+#import <React/RCTBundleURLProvider.h>

@interface AppDelegate ()

@end

@implementation AppDelegate {
  UIWindow *window;
}

 - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
+ self.automaticallyLoadReactNativeWindow = NO;
+ return [super application:application didFinishLaunchingWithOptions:launchOptions];
   window = [UIWindow new];
   window.rootViewController = [ViewController new];
   [window makeKeyAndVisible];
   return YES;

 }

+- (NSURL *)sourceURLForBridge:(RCTBridge *)bridge
+{
+  return [self bundleURL];
+}

+- (NSURL *)bundleURL
+{
+#if DEBUG
+  return [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index"];
+#else
+  return [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
+#endif
+}
 @end
```

Let's have a look at the code above:

1. We are inheriting from the `RCTAppDelegate` and we are calling the `application:didFinishLaunchingWithOptions` of the `RCTAppDelegate`. This delegates all the React Native initialization processes to the base class.
2. We are customizing the `RCTAppDelegate` by setting the `automaticallyLoadReactNativeWindow` to `NO`. This step instruct React Native that the app is handling the `UIWindow` and React Native should not worry about that.
3. The methods `sourceURLForBridge:` and `bundleURL` are used by the App to tell to React Native where it can find the JS bundle that needs to be rendered. The `sourceURLForBridge:` is from the Old Architecture and you can see that it is deferring the decision to the `bundleURL` method, required by the New Architecture.

</TabItem>
<TabItem value="swift">

To achieve this, we have to modify the `AppDelegate.swift`

1. Open the `AppDelegate.swift` files and modify it as it follows (See the official template's [AppDelegate.swift](https://github.com/react-native-community/template/blob/main/template/ios/HelloWorld/AppDelegate.swift) as reference):

```diff title="AppDelegate.swift"
import UIKit
+import React_RCTAppDelegate

@main
-class AppDelegate: UIResponder, UIApplicationDelegate {
+class AppDelegate: RCTAppDelegate {

-  var window: UIWindow?

-  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
+  override func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.
+    self.automaticallyLoadReactNativeWindow = false
+    super.application(application, didFinishLaunchingWithOptions: launchOptions)
    window = UIWindow()
-    window?.rootViewController = ViewController()
-    window?.makeKeyAndVisible()
+    window.rootViewController = ViewController()
+    window.makeKeyAndVisible()
    return true
  }

+  override func sourceURL(for bridge: RCTBridge) -> URL? {
+    self.bundleURL()
+  }

+  override func bundleURL() -> URL? {
+#if DEBUG
+    RCTBundleURLProvider.sharedSettings().jsBundleURL(forBundleRoot: "index")
+#else
+    Bundle.main.url(forResource: "main", withExtension: "jsbundle")
+#endif
+  }
}
```

Let's have a look at the code above:

1. We are inheriting from the `RCTAppDelegate` and we are calling the `application(_:didFinishLaunchingWithOptions:)` of the `RCTAppDelegate`. This delegates all the React Native initialization processes to the base class.
2. We are customizing the `RCTAppDelegate` by setting the `automaticallyLoadReactNativeWindow` to `false`. This step instruct React Native that the app is handling the `UIWindow` and React Native should not worry about that.
3. The methods `sourceURLForBridge(for:)` and `bundleURL()` are used by the App to tell to React Native where it can find the JS bundle that needs to be rendered. The `sourceURLForBridge(for:)` is from the Old Architecture and you can see that it is deferring the decision to the `bundleURL()` method, required by the New Architecture.

</TabItem>
</Tabs>

#### 在 rootViewController 中呈現 React Native 視圖

最後，我們可以呈現 React Native 視圖。為此，我們需要一個新的 View Controller 來裝載可載入 JS 內容的視圖。

1. 在 Xcode 中建立一個新的 `UIViewController`（我們將其命名為 `ReactViewController`）。
2. 讓初始的 `ViewController` 呈現 `ReactViewController`。根據你的應用程式架構，有數種方式可達成此目的。本範例假設你有一個按鈕會以模態方式呈現 React Native。

<Tabs groupId="ios-language" queryString defaultValue={constants.defaultAppleLanguage} values={constants.appleLanguages}>
<TabItem value="objc">

```diff title="ViewController.m"
#import "ViewController.h"
+#import "ReactViewController.h"

@interface ViewController ()

@end

- @implementation ViewController
+@implementation ViewController {
+  ReactViewController *reactViewController;
+}

 - (void)viewDidLoad {
   [super viewDidLoad];
   // Do any additional setup after loading the view.
   self.view.backgroundColor = UIColor.systemBackgroundColor;
+  UIButton *button = [UIButton new];
+  [button setTitle:@"Open React Native" forState:UIControlStateNormal];
+  [button setTitleColor:UIColor.systemBlueColor forState:UIControlStateNormal];
+  [button setTitleColor:UIColor.blueColor forState:UIControlStateHighlighted];
+  [button addTarget:self action:@selector(presentReactNative) forControlEvents:UIControlEventTouchUpInside];
+  [self.view addSubview:button];

+  button.translatesAutoresizingMaskIntoConstraints = NO;
+  [NSLayoutConstraint activateConstraints:@[
+    [button.leadingAnchor constraintEqualToAnchor:self.view.leadingAnchor],
+    [button.trailingAnchor constraintEqualToAnchor:self.view.trailingAnchor],
+    [button.centerYAnchor constraintEqualToAnchor:self.view.centerYAnchor],
+    [button.centerXAnchor constraintEqualToAnchor:self.view.centerXAnchor],
+  ]];
 }

+- (void)presentReactNative
+{
+  if (reactViewController == NULL) {
+    reactViewController = [ReactViewController new];
+  }
+  [self presentViewController:reactViewController animated:YES completion:nil];
+}

@end
```

</TabItem>
<TabItem value="swift">

```diff title="ViewController.swift"
import UIKit

class ViewController: UIViewController {

+  var reactViewController: ReactViewController?

  override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.
    self.view.backgroundColor = .systemBackground

+    let button = UIButton()
+    button.setTitle("Open React Native", for: .normal)
+    button.setTitleColor(.systemBlue, for: .normal)
+    button.setTitleColor(.blue, for: .highlighted)
+    button.addAction(UIAction { [weak self] _ in
+      guard let self else { return }
+      if reactViewController == nil {
+       reactViewController = ReactViewController()
+      }
+      present(reactViewController!, animated: true)
+    }, for: .touchUpInside)
+    self.view.addSubview(button)
+
+    button.translatesAutoresizingMaskIntoConstraints = false
+    NSLayoutConstraint.activate([
+      button.leadingAnchor.constraint(equalTo: self.view.leadingAnchor),
+      button.trailingAnchor.constraint(equalTo: self.view.trailingAnchor),
+      button.centerXAnchor.constraint(equalTo: self.view.centerXAnchor),
+      button.centerYAnchor.constraint(equalTo: self.view.centerYAnchor),
+    ])
  }
}
```

</TabItem>
</Tabs>

3. 按照以下方式更新 `ReactViewController` 的程式碼：

<Tabs groupId="ios-language" queryString defaultValue={constants.defaultAppleLanguage} values={constants.appleLanguages}>
<TabItem value="objc">

```diff title="ReactViewController.m"
#import "ReactViewController.h"
+#import <React-RCTAppDelegate/RCTRootViewFactory.h>
+#import <React-RCTAppDelegate/RCTAppDelegate.h>

@interface ReactViewController ()

@end

@implementation ReactViewController

 - (void)viewDidLoad {
   [super viewDidLoad];
   // Do any additional setup after loading the view.
+   RCTRootViewFactory *factory = ((RCTAppDelegate *)RCTSharedApplication().delegate).rootViewFactory;
+   self.view = [factory viewWithModuleName:@"HelloWorld"];
 }

@end
```

</TabItem>
<TabItem value="swift">

```diff title="ReactViewController.swift"
import UIKit
+import React_RCTAppDelegate

class ReactViewController: UIViewController {

  override func viewDidLoad() {
    super.viewDidLoad()

+    let factory = (RCTSharedApplication()?.delegate as? RCTAppDelegate)?.rootViewFactory
+    self.view = factory?.view(withModuleName: "HelloWorld")
  }
}
```

</TabItem>
</Tabs>

4. 請務必停用沙箱腳本功能。在 Xcode 中點選您的應用程式，然後進入建置設定，篩選「script」並將 `User Script Sandboxing` 設為 `NO`。此步驟是為了讓 React Native 內建的 [Hermes 引擎](https://github.com/facebook/hermes/blob/main/README.md) 能在 Debug 和 Release 版本間正確切換。

![停用沙箱功能](/docs/assets/disable-sandboxing.png);

## 6. 測試整合結果

您已完成整合 React Native 與應用程式的基本步驟。現在我們將啟動 [Metro 打包工具](https://metrobundler.dev/)，將您的 TypeScript 應用程式碼打包成 bundle。Metro 的 HTTP 伺服器會將 bundle 從開發環境的 `localhost` 分享至模擬器或裝置，這使得 [熱重載](https://reactnative.dev/blog/2016/03/24/introducing-hot-reloading) 成為可能。

首先，您需要在專案根目錄建立一個 `metro.config.js` 檔案，內容如下：

```js
const {getDefaultConfig} = require('@react-native/metro-config');
module.exports = getDefaultConfig(__dirname);
```

您可以參考社群範本中的 [metro.config.js 檔案](https://github.com/react-native-community/template/blob/0.77-stable/template/metro.config.js)。

設定檔就位後，即可執行打包工具。在專案根目錄執行以下指令：

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

接著如常建置並執行您的 iOS 應用程式。

當應用程式載入 React 驅動的 Activity 時，應會從開發伺服器取得 JavaScript 程式碼並顯示：

<center><img src="/docs/assets/EmbeddedAppIOSVideo.gif" width="300" /></center>

### 在 Xcode 中建立正式版建置

您也可以使用 Xcode 建立正式版建置！唯一的額外步驟是新增一個在建置應用程式時執行的腳本，用於將 JS 和圖片打包至 iOS 應用程式中。

1. 在 Xcode 中選取您的應用程式
2. 點選 `Build Phases`
3. 點擊左上角的 `+` 並選擇 `New Run Script Phase`
4. 點選 `Run Script` 這一行，將腳本重新命名為 `Bundle React Native code and images`
5. 在文字框中貼上以下腳本

```sh title="Build React Native code and image"
set -e

WITH_ENVIRONMENT="$REACT_NATIVE_PATH/scripts/xcode/with-environment.sh"
REACT_NATIVE_XCODE="$REACT_NATIVE_PATH/scripts/react-native-xcode.sh"

/bin/sh -c "$WITH_ENVIRONMENT $REACT_NATIVE_XCODE"
```

6. 將此腳本拖放至名為 `[CP] Embed Pods Frameworks` 的腳本之前。

現在，當您為正式版建置應用程式時，它將如預期運作。

### 接下來呢？

至此，您可以如常繼續開發應用程式。請參考我們的[除錯](debugging)和[部署](running-on-device)文件，以進一步了解如何使用 React Native。