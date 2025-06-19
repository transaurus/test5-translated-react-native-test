---
id: fabric-native-components-ios
title: 'Fabric Native Components: iOS'
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

現在是時候撰寫一些 iOS 平台程式碼來渲染網頁視圖。你需要遵循的步驟如下：

- Run Codegen.
- Write the code for the `RCTWebView`
- Register the `RCTWebView` in the application

### 1. 執行 Codegen

你可以[手動執行](the-new-architecture/codegen-cli) Codegen，但更簡單的方式是使用你將用來展示元件的應用程式來代為執行。

```bash
cd ios
bundle install
bundle exec pod install
```

重要的是你會看到來自 Codegen 的日誌輸出，我們將在 Xcode 中使用這些日誌來建構我們的 WebView 原生元件。

:::warning
你應該謹慎考慮是否將生成的程式碼提交到你的儲存庫。生成的程式碼是針對特定版本的 React Native。使用 npm 的 [peerDependencies](https://nodejs.org/en/blog/npm/peer-dependencies) 來限制與 React Native 版本的相容性。
:::

### 3. 撰寫 `RCTWebView`

我們需要使用 Xcode 準備你的 iOS 專案，完成以下 **5 個步驟**：

1. 開啟由 CocoPods 生成的 Xcode Workspace：

```bash
cd ios
open Demo.xcworkspace
```

<img class="half-size" alt="Open Xcode Workspace" src="/docs/assets/fabric-native-components/1.webp" />

2. 右鍵點擊應用程式並選擇 <code>New Group</code>，將新群組命名為 `WebView`。

<img class="half-size" alt="Right click on app and select New Group" src="/docs/assets/fabric-native-components/2.webp" />

3. 在 `WebView` 群組中，建立 <code>New</code>→<code>File from Template</code>。

<img class="half-size" alt="Create a new file using the Cocoa Touch Classs template" src="/docs/assets/fabric-native-components/3.webp" />

4. 使用 <code>Objective-C File</code> 模板，並將其命名為 <code>RCTWebView</code>。

<img class="half-size" alt="Create an Objective-C RCTWebView class" src="/docs/assets/fabric-native-components/4.webp" />

5. 重複步驟 4，建立一個名為 `RCTWebView.h` 的標頭檔。

6. 將 <code>RCTWebView.m</code> 重新命名為 <code>RCTWebView.mm</code>，使其成為 Objective-C++ 檔案。

```text title="Demo/ios"
Podfile
...
Demo
├── AppDelegate.swift
...
// highlight-start
├── RCTWebView.h
└── RCTWebView.mm
// highlight-end
```

在建立標頭檔和實作檔案後，你可以開始實作它們。

這是 `RCTWebView.h` 檔案的程式碼，它宣告了元件的介面。

```objc title="Demo/RCTWebView/RCTWebView.h"
#import <React/RCTViewComponentView.h>
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface RCTWebView : RCTViewComponentView

// You would declare native methods you'd want to access from the view here

@end

NS_ASSUME_NONNULL_END
```

這個類別定義了一個 `RCTWebView`，它繼承自 `RCTViewComponentView` 類別。這是所有原生元件的基礎類別，由 React Native 提供。

實作檔案 (`RCTWebView.mm`) 的程式碼如下：

```objc title="Demo/RCTWebView/RCTWebView.mm"
#import "RCTWebView.h"

#import <react/renderer/components/AppSpec/ComponentDescriptors.h>
#import <react/renderer/components/AppSpec/EventEmitters.h>
#import <react/renderer/components/AppSpec/Props.h>
#import <react/renderer/components/AppSpec/RCTComponentViewHelpers.h>
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

@end
```

這段程式碼是用 Objective-C++ 撰寫的，包含以下細節：

- the `@interface` implements two protocols:
  - `RCTCustomWebViewViewProtocol`, generated by Codegen;
  - `WKNavigationDelegate`, provided by the WebKit framework to handle the web view navigation events;
- the `init` method that instantiates the `WKWebView`, adds it to the subviews and that sets the `navigationDelegate`;
- the `updateProps` method that is called by React Native when the component's props change;
- the `layoutSubviews` method that describes how the custom view needs to be laid out;
- the `webView:didFinishNavigation:` method that lets you handle what to do when the `WKWebView` finishes loading the page;
- the `urlIsValid:(std::string)propString` method that checks whether the URL received as prop is valid;
- the `eventEmitter` method which is a utility to retrieve a strongly typed `eventEmitter` instance
- the `componentDescriptorProvider` which returns the `ComponentDescriptor` generated by Codegen;

#### 加入 WebKit 框架

:::note
此步驟僅因我們正在建立一個網頁視圖而需要。iOS 上的網頁元件需要連結至 Apple 提供的 WebKit 框架。若您的元件不需要存取網頁相關功能，可跳過此步驟。
:::

網頁視圖需要存取 Apple 透過 Xcode 和裝置提供的某些功能：WebKit 框架。
您可以在原生程式碼中看到 `#import <WebKit/WebKit.h>` 這行已加入至 `RCTWebView.mm` 檔案。

要在您的應用程式中連結 WebKit 框架，請遵循以下步驟：

1. 在 Xcode 中，點擊您的專案
2. 選擇應用程式目標
3. 選擇 General 標籤頁
4. 向下滾動直到找到 _"Frameworks, Libraries, and Embedded Contents"_ 區段，並按下 `+` 按鈕

<img class="half-size" alt="Add webkit framework to your app 1" src="/docs/assets/AddWebKitFramework1.png" />

5. 在搜尋欄中，篩選 WebKit
6. 選擇 WebKit 框架
7. 點擊 Add。

<img class="half-size" alt="Add webkit framework to your app 2" src="/docs/assets/AddWebKitFramework2.png" />