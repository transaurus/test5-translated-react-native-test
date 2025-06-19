import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 核心概念

將 React Native 元件整合至 iOS 應用的關鍵在於：

1. 建立正確的目錄結構
2. 安裝必要的 NPM 依賴項
3. 在 Podfile 配置中添加 React Native
4. 為首個 React Native 畫面編寫 TypeScript 程式碼
5. 使用 `RCTRootView` 將 React Native 與 iOS 程式碼整合
6. 執行打包器測試整合結果，並查看應用運行狀況

## 使用社群模板

遵循本指南時，建議您參考 [React Native 社群模板](https://github.com/react-native-community/template/)。該模板包含一個**最小化 iOS 應用**，可幫助您理解如何將 React Native 整合至現有 iOS 應用。

## 必要條件

請先完成 [開發環境設定指南](set-up-your-environment) 並學習 [無框架的 React Native 使用方式](getting-started-without-a-framework)，以配置 iOS 版 React Native 應用的開發環境。
本指南同時假設您已掌握 iOS 開發基礎知識，例如建立 `UIViewController` 和編輯 `Podfile` 文件。

### 1. 建立目錄結構

為確保流程順暢，請先為整合專案建立新資料夾，然後**將現有 iOS 專案**移動至 `/ios` 子資料夾。

## 2. 安裝 NPM 依賴項

進入根目錄執行以下指令：

```shell
curl -O https://raw.githubusercontent.com/react-native-community/template/refs/heads/0.76-stable/template/package.json
```

This will copy the `package.json` [file from the Community template](https://github.com/react-native-community/template/blob/0.76-stable/template/package.json) to your project.

接著執行以下指令安裝 NPM 套件：

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

安裝過程會建立新的 `node_modules` 資料夾，此資料夾儲存專案建構所需的所有 JavaScript 依賴項。

請將 `node_modules/` 加入您的 `.gitignore` 文件（可參考[社群預設版本](https://github.com/react-native-community/template/blob/0.76-stable/template/_gitignore)）。

### 3. 安裝開發工具

### Xcode 命令行工具

安裝命令行工具：在 Xcode 選單中選擇 **Settings... (或 Preferences...)**，進入 Locations 面板後，從 Command Line Tools 下拉選單中選擇最新版本進行安裝。

![Xcode 命令行工具](/docs/assets/GettingStartedXcodeCommandLineTools.png)

### CocoaPods

[CocoaPods](https://cocoapods.org) 是 iOS 和 macOS 開發的套件管理工具，我們用它將實際的 React Native 框架程式碼本地化加入您的專案。

建議透過 [Homebrew](https://brew.sh/) 安裝 CocoaPods：

```shell
brew install cocoapods
```

## 4. 將 React Native 加入應用

### 配置 CocoaPods

配置 CocoaPods 需要兩個文件：

- A **Gemfile** that defines which Ruby dependencies we need.
- A **Podfile** that defines how to properly install our dependencies.

對於 **Gemfile**，請進入專案根目錄執行：

```sh
curl -O https://raw.githubusercontent.com/react-native-community/template/refs/heads/0.76-stable/template/Gemfile
```

此指令會從模板下載 Gemfile。
對於 **Podfile**，請進入專案的 `ios` 資料夾執行：

```sh
curl -O https://raw.githubusercontent.com/react-native-community/template/refs/heads/0.76-stable/template/ios/Podfile
```

請使用社群範本作為 [Gemfile](https://github.com/react-native-community/template/blob/0.76-stable/template/Gemfile) 和 [Podfile](https://github.com/react-native-community/template/blob/0.76-stable/template/ios/Podfile) 的參考依據。

:::note
請記得修改 Podfile 中的[這一行](https://github.com/react-native-community/template/blob/0.76-stable/template/ios/Podfile#L17)和[這一行](https://github.com/react-native-community/template/blob/0.76-stable/template/ios/Podfile#L26)以符合你的應用程式名稱。

如果你的應用程式沒有測試，請記得移除[這個區塊](https://github.com/react-native-community/template/blob/0.76-stable/template/ios/Podfile#L26-L29)。
:::

現在，我們需要執行幾個額外指令來安裝 Ruby gems 和 Pods。
請導航至 `ios` 資料夾並執行以下指令：

```sh
bundle install
bundle exec pod install
```

第一個指令會安裝 Ruby 相依套件，第二個指令則會實際將 React Native 程式碼整合至你的應用程式，讓你的 iOS 檔案能夠導入 React Native 標頭檔。

## 5. 撰寫 TypeScript 程式碼

現在我們將實際修改原生 iOS 應用程式來整合 React Native。

我們要撰寫的第一段程式碼，是為將整合至應用程式的新畫面所寫的 React Native 程式碼。

### 建立 `index.js` 檔案

首先，在你的 React Native 專案根目錄建立一個空的 `index.js` 檔案。

`index.js` 是 React Native 應用程式的進入點，且必須存在。它可以是一個小檔案，僅用來 `import` 其他屬於你的 React Native 元件或應用程式的檔案，也可以包含所需的所有程式碼。

我們的 `index.js` 應該如下所示（此處以[社群範本檔案為參考](https://github.com/react-native-community/template/blob/0.76-stable/template/index.js)）：

```js
import {AppRegistry} from 'react-native';
import App from './App';

AppRegistry.registerComponent('HelloWorld', () => App);
```

### 建立 `App.tsx` 檔案

讓我們建立一個 `App.tsx` 檔案。這是一個 [TypeScript](https://www.typescriptlang.org/) 檔案，可以包含 [JSX](<https://en.wikipedia.org/wiki/JSX_(JavaScript)>) 表達式。它包含了我們將整合至 iOS 應用程式的根 React Native 元件（[連結](https://github.com/react-native-community/template/blob/0.76-stable/template/App.tsx)）：

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

此處以[社群範本檔案為參考](https://github.com/react-native-community/template/blob/0.76-stable/template/App.tsx)

## 5. 與 iOS 程式碼整合

我們現在需要添加一些原生程式碼，以啟動 React Native 運行時環境並告訴它渲染我們的 React 元件。

### 需求

React Native 預期與 `AppDelegate` 協同工作。以下部分假設你的 `AppDelegate` 如下所示：

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

最後，我們可以呈現我們的 React Native 視圖。為此，我們需要一個新的 View Controller 來承載一個視圖，以便在其中載入 JS 內容。

1. 從 Xcode 建立一個新的 `UIViewController`（我們將其命名為 `ReactViewController`）。
2. 讓初始的 `ViewController` 呈現 `ReactViewController`。根據你的應用程式，有幾種方式可以實現。在此範例中，我們假設你有一個按鈕會以模態方式呈現 React Native。

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
+      if reactViewController == nil {
+       reactViewController = ReactViewController()
+      }
+      self?.present(reactViewController, animated: true)
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

3. 按照以下方式更新 `ReactViewController` 代碼：

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

4. 確保停用沙箱腳本功能。在 Xcode 中點擊您的應用程式，然後進入建置設定，篩選「script」並將 `User Script Sandboxing` 設為 `NO`。此步驟對於正確切換 React Native 內建的 [Hermes 引擎](https://github.com/facebook/hermes/blob/main/README.md) 除錯版與發行版至關重要。

![停用沙箱功能](/docs/assets/disable-sandboxing.png);

## 6. 測試整合結果

您已完成整合 React Native 的所有基本步驟。現在我們將啟動 [Metro 打包工具](https://metrobundler.dev/)，將您的 TypeScript 應用程式代碼打包成套件。Metro 的 HTTP 伺服器會將套件從開發環境的 `localhost` 傳送至模擬器或實體裝置，這使得 [熱重載](https://reactnative.dev/blog/2016/03/24/introducing-hot-reloading) 成為可能。

首先，您需要在專案根目錄建立 `metro.config.js` 檔案，內容如下：

```js
const {getDefaultConfig} = require('@react-native/metro-config');
module.exports = getDefaultConfig(__dirname);
```

您可以參考社群範本中的 [metro.config.js 檔案](https://github.com/react-native-community/template/blob/0.76-stable/template/metro.config.js)。

設定檔就位後，即可執行打包工具。在專案根目錄執行以下命令：

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

當應用程式載入 React 驅動的畫面時，應會從開發伺服器取得 JavaScript 代碼並顯示：

<center><img src="/docs/assets/EmbeddedAppIOSVideo.gif" width="300" /></center>

### 在 Xcode 建立發行版本

您也可以使用 Xcode 建立發行版本！唯一需要增加的步驟是添加在建置應用程式時執行的腳本，將 JS 和圖片打包進 iOS 應用程式中。

1. 在 Xcode 中選取您的應用程式
2. 點擊 `Build Phases`
3. 點擊左上角的 `+` 並選擇 `New Run Script Phase`
4. 點擊 `Run Script` 這行，將腳本命名為 `Bundle React Native code and images`
5. 在文字框中貼上以下腳本

```sh title="Build React Native code and image"
set -e

WITH_ENVIRONMENT="$REACT_NATIVE_PATH/scripts/xcode/with-environment.sh"
REACT_NATIVE_XCODE="$REACT_NATIVE_PATH/scripts/react-native-xcode.sh"

/bin/sh -c "$WITH_ENVIRONMENT $REACT_NATIVE_XCODE"
```

6. 將此腳本拖放至名為 `[CP] Embed Pods Frameworks` 的腳本之前。

現在，當您建置發行版本時，應用程式將能正常運作。

### 接下來呢？

至此，您可以繼續如常開發應用程式。請參考我們的[除錯指南](debugging)和[裝置部署文件](running-on-device)，了解更多關於 React Native 開發的資訊。