---
id: fabric-native-components-ios
title: 'Fabric Native Components: iOS'
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

現在是時候撰寫一些 iOS 平台程式碼來渲染網頁視圖。您需要遵循的步驟如下：

- Run Codegen.
- Write the code for the `RCTWebView`
- Register the `RCTWebView` in the application

### 1. 執行 Codegen

您可以[手動執行](the-new-architecture/codegen-cli) Codegen，但更簡單的方法是使用您將用來展示該元件的應用程式來代為執行。

```bash
cd ios
bundle install
bundle exec pods install
```

重要的是，您將看到來自 Codegen 的日誌輸出，我們將在 Xcode 中使用這些輸出建置我們的 WebView 原生元件。

:::warning
您應謹慎考慮是否將生成的程式碼提交至儲存庫。生成的程式碼專屬於特定版本的 React Native。請使用 npm 的 [peerDependencies](https://nodejs.org/en/blog/npm/peer-dependencies) 來限制與 React Native 版本的相容性。
:::

### 3. 撰寫 `RCTWebView`

我們需要透過 Xcode 準備您的 iOS 專案，完成以下 **5 個步驟**：

1. 開啟由 CocoPods 生成的 Xcode Workspace：

```bash
cd ios
open Demo.xcworkspace
```

<img class="half-size" alt="Open Xcode Workspace" src="/docs/assets/fabric-native-components/1.webp" />

2. 在應用程式上按右鍵並選擇 <code>New Group</code>，將新群組命名為 `WebView`。

<img class="half-size" alt="Right click on app and select New Group" src="/docs/assets/fabric-native-components/2.webp" />

3. 在 `WebView` 群組中，建立 <code>New</code>→<code>File from Template</code>。

<img class="half-size" alt="Create a new file using the Cocoa Touch Classs template" src="/docs/assets/fabric-native-components/3.webp" />

4. 使用 <code>Objective-C File</code> 模板，並將其命名為 <code>RCTWebView</code>。

<img class="half-size" alt="Create an Objective-C RCTWebView class" src="/docs/assets/fabric-native-components/4.webp" />

5. 將 <code>RCTWebView.m</code> 重新命名為 <code>RCTWebView.mm</code>，使其成為 Objective-C++ 檔案

```text title="Demo/ios"
Podfile
...
Demo
├── AppDelegate.h
├── AppDelegate.mm
...
// highlight-start
├── RCTWebView.h
├── RCTWebView.mm
// highlight-end
└── main.m
```

建立標頭檔和實作檔案後，您可以開始實作它們。

以下是 `RCTWebView.h` 檔案的程式碼，該檔案宣告了元件介面。

```objc title="Demo/RCTWebView/RTNWebView.h"
#import <React/RCTViewComponentView.h>
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface RCTWebView : RCTViewComponentView

// You would declare native methods you'd want to access from the view here

@end

NS_ASSUME_NONNULL_END
```

此類別定義了一個 `RCTWebView`，它繼承自 `RCTViewComponentView` 類別。這是所有原生元件的基礎類別，由 React Native 提供。

實作檔案 (`RCTWebView.mm`) 的程式碼如下：

```objc title="Demo/RCTWebView/RCTWebView.mm"
#import "RCTWebView.h"

#import <react/renderer/components/AppSpecs/ComponentDescriptors.h>
#import <react/renderer/components/AppSpecs/EventEmitters.h>
#import <react/renderer/components/AppSpecs/Props.h>
#import <react/renderer/components/AppSpecs/RCTComponentViewHelpers.h>
// highlight-next-line
#import <WebKit/WebKit.h>

using namespace facebook::react;

@interface RCTWebView () <RCTCustomWebViewViewProtocol, WKNavigationDelegate>
@end

@implementation RCTWebView {
  NSURL * _sourceURL;
  WKWebView * _webView;
}

-(instancetype)init
{
  if(self = [super init]) {
    // highlight-start
    _webView = [WKWebView new];
    _webView.navigationDelegate = self;
    [self addSubview:_webView];
    // highlight-end
  }
  return self;
}

- (void)updateProps:(Props::Shared const &)props oldProps:(Props::Shared const &)oldProps
{
  const auto &oldViewProps = *std::static_pointer_cast<CustomWebViewProps const>(_props);
  const auto &newViewProps = *std::static_pointer_cast<CustomWebViewProps const>(props);

  // Handle your props here
  if (oldViewProps.sourceURL != newViewProps.sourceURL) {
    NSString *urlString = [NSString stringWithCString:newViewProps.sourceURL.c_str() encoding:NSUTF8StringEncoding];
    _sourceURL = [NSURL URLWithString:urlString];
    // highlight-start
    if ([self urlIsValid:newViewProps.sourceURL]) {
      [_webView loadRequest:[NSURLRequest requestWithURL:_sourceURL]];
    }
    // highlight-end
  }

  [super updateProps:props oldProps:oldProps];
}

-(void)layoutSubviews
{
  [super layoutSubviews];
  _webView.frame = self.bounds;

}

#pragma mark - WKNavigationDelegate

// highlight-start
-(void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation
{
  CustomWebViewEventEmitter::OnScriptLoaded result = CustomWebViewEventEmitter::OnScriptLoaded{CustomWebViewEventEmitter::OnScriptLoadedResult::Success};
  self.eventEmitter.onScriptLoaded(result);
}

- (BOOL)urlIsValid:(std::string)propString
{
  if (propString.length() > 0 && !_sourceURL) {
    CustomWebViewEventEmitter::OnScriptLoaded result = CustomWebViewEventEmitter::OnScriptLoaded{CustomWebViewEventEmitter::OnScriptLoadedResult::Error};

    self.eventEmitter.onScriptLoaded(result);
    return NO;
  }
  return YES;
}

// Event emitter convenience method
- (const CustomWebViewEventEmitter &)eventEmitter
{
  return static_cast<const CustomWebViewEventEmitter &>(*_eventEmitter);
}
// highlight-end

+ (ComponentDescriptorProvider)componentDescriptorProvider
{
  return concreteComponentDescriptorProvider<CustomWebViewComponentDescriptor>();
}

Class<RCTComponentViewProtocol> WebViewCls(void)
{
  return RCTWebView.class;
}

@end
```

此程式碼以 Objective-C++ 撰寫，包含以下細節：

- the `@interface` implements two protocols:
  - `RCTCustomWebViewViewProtocol`, generated by Codegen;
  - `WKNavigationDelegate`, provided by the WebKit frameworks to handle the web view navigation events;
- the `init` method that instantiate the `WKWebView`, adds it to the subviews and that set the `navigationDelegate`;
- the `updateProps` method that is called by React Native when the component's props change;
- the `layoutSubviews` method that describe how the custom view needs to be laid out;
- the `webView:didFinishNavigation:` method that let you handle the what do when the `WKWebView` finishes loading the page;
- the `urlIsValid:(std::string)propString` method that checks whether the URL received as prop is valid;
- the `eventEmitter` method which is a utility to retrieve a strongly typed `eventEmitter` instance
- the `componentDescriptorProvider` which returns the `ComponentDescriptor` generated by Codegen;
- the `WebViewCls` which is a helper method to register the `RCTWebView` in the application.

#### AppDelegate.mm

最後，您可以在應用程式中註冊該元件。
更新 `AppDelegate.mm`，讓您的應用程式知曉我們的自訂 WebView 元件：

```objc title="Demo/ios/Demo/AppDelegate.mm"
#import "AppDelegate.h"

#import <React/RCTBundleURLProvider.h>
#import <React/RCTBridge+Private.h>
// highlight-next-line
#import "RCTWebView.h"

@implementation AppDelegate
// ...
// highlight-start
- (NSDictionary<NSString *,Class<RCTComponentViewProtocol>> *)thirdPartyFabricComponents
{
  NSMutableDictionary * dictionary = [super thirdPartyFabricComponents].mutableCopy;
  dictionary[@"CustomWebView"] = [RCTWebView class];
  return dictionary;
}
// highlight-end

@end
```

這段程式碼覆寫了 `thirdPartyFabricComponents` 方法，透過取得來自其他來源（如第三方函式庫）的第三方元件字典的可變動副本。

接著在字典中添加一個條目，使用 Codegen 規格檔案中定義的名稱。如此一來，當 React 需要載入名為 `CustomWebView` 的元件時，React Native 就會實例化一個 `RCTWebView`。

最後，它會回傳這個新的字典。

#### 添加 WebKit 框架

:::note
此步驟僅因我們要建立網頁視圖而需要。iOS 上的網頁元件需要連結到 Apple 提供的 WebKit 框架。如果你的元件不需要存取網頁相關功能，可以跳過此步驟。
:::

網頁視圖需要存取 Apple 透過 Xcode 和裝置內建的框架之一 WebKit 提供的某些功能。
你可以在原生程式碼中看到 `#import <WebKit/WebKit.h>` 這行，它被添加到 `RCTWebView.mm` 中。

要在你的應用程式中連結 WebKit 框架，請按照以下步驟操作：

1. In Xcode, Click on your project
2. Select the app target
3. Select the General tab
4. Scroll down until you find the _"Frameworks, Libraries, and Embedded Contents"_ section, and press the `+` button

<img class="half-size" alt="Add webkit framework to your app 1" src="/docs/assets/AddWebKitFramework1.png" />

5. 在搜尋欄中，過濾出 WebKit
6. 選擇 WebKit 框架
7. 點擊 Add。

<img class="half-size" alt="Add webkit framework to your app 2" src="/docs/assets/AddWebKitFramework2.png" />