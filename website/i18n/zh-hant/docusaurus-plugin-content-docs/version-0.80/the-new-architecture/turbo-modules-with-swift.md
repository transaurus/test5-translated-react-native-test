# iOS - 在原生模組中使用 Swift

Swift 是開發 iOS 原生應用程式的官方預設語言。

本指南將帶您探索如何使用 Swift 撰寫原生模組。

:::note
React Native 的核心主要使用 C++ 撰寫，而 Swift 與 C++ 之間的互通性並不理想，儘管 Apple 開發了[互通性層](https://www.swift.org/documentation/cxx-interop/)。

因此，本指南中您將撰寫的模組不會是純 Swift 實作，由於語言間的不相容性，您仍需編寫一些 Objective-C++ 黏合程式碼。本指南的目標是盡量減少所需的 Objective-C++ 程式碼量。若您正將現有原生模組從舊架構遷移至新架構，此方法應能讓您重用大部分程式碼。
:::

This guide starts from the iOS implementation of the [Native Module](/docs/next/turbo-native-modules-introduction) guide.
Make sure to be familiar with that guide before diving into this one, potentially implementing the example in the guide.

## 適配器模式

目標是使用 Swift 模組實作所有業務邏輯，並透過一層薄薄的 Objective-C++ 黏合層來連接應用程式與 Swift 實作。

您可以運用[適配器](https://en.wikipedia.org/wiki/Adapter_pattern)設計模式來達成此目標，將 Swift 模組與 Objective-C++ 層連接。

Objective-C++ 物件由 React Native 創建，它持有 Swift 模組的參考並管理其生命週期。Objective-C++ 物件會將所有方法調用轉發至 Swift。

### 創建 Swift 模組

第一步是將實作從 Objective-C++ 層移至 Swift 層。

請依以下步驟操作：

1. 在 Xcode 專案中創建新空白檔案，命名為 `NativeLocalStorage.swift`
2. 在 Swift 模組中加入如下實作：

```swift title="NativeLocalStorage.swift"
import Foundation

@objc public class NativeLocalStorage: NSObject {
  let userDefaults = UserDefaults(suiteName: "local-storage");


  @objc public func getItem(for key: String) -> String? {
    return userDefaults?.string(forKey: key)
  }

  @objc public func setItem(for key: String, value: String) {
    userDefaults?.set(value, forKey: key)
  }

  @objc public func removeItem(for key: String) {
    userDefaults?.removeObject(forKey: key)
  }

  @objc public  func clear() {
    userDefaults?.dictionaryRepresentation().keys.forEach { removeItem(for: $0) }
  }
}

```

Notice that you have to declare all the methods that you need to call from Objective-C as `public` and with the `@objc` annotation.
Remember also to make your class inherit from `NSObject`, otherwise it would not be possible to use it from Objective-C.

### 更新 `RCTNativeLocalStorage` 檔案

接著，您需要更新 `RCTNativeLocalStorage` 的實作，使其能創建 Swift 模組並調用其方法。

1. 開啟 `RCTNativeLocalStorage.mm` 檔案
2. 依以下方式更新：

```diff title="RCTNativeLocalStorage.mm"
//  RCTNativeLocalStorage.m
//  TurboModuleExample

#import "RCTNativeLocalStorage.h"
+#import "SampleApp-Swift.h"

- static NSString *const RCTNativeLocalStorageKey = @"local-storage";

-@interface RCTNativeLocalStorage()
-@property (strong, nonatomic) NSUserDefaults *localStorage;
-@end

-@implementation RCTNativeLocalStorage
+@implementation RCTNativeLocalStorage {
+    NativeLocalStorage *storage;
+}

-RCT_EXPORT_MODULE(NativeLocalStorage)

 - (id) init {
   if (self = [super init]) {
-    _localStorage = [[NSUserDefaults alloc] initWithSuiteName:RCTNativeLocalStorageKey];
+    storage = [NativeLocalStorage new];
   }
   return self;
 }

 - (std::shared_ptr<facebook::react::TurboModule>)getTurboModule:(const facebook::react::ObjCTurboModule::InitParams &)params {
   return std::make_shared<facebook::react::NativeLocalStorageSpecJSI>(params);
 }

 - (NSString * _Nullable)getItem:(NSString *)key {
-   return [self.localStorage stringForKey:key];
+   return [storage getItemFor:key];
 }

 - (void)setItem:(NSString *)value key:(NSString *)key {
-   [self.localStorage setObject:value forKey:key];
+   [storage setItemFor:key value:value];
 }

 - (void)removeItem:(NSString *)key {
-   [self.localStorage removeObjectForKey:key];
+   [storage removeItemFor:key];
 }

 - (void)clear {
-   NSDictionary *keys = [self.localStorage dictionaryRepresentation];
-   for (NSString *key in keys) {
-     [self removeItem:key];
-   }
+  [storage clear];
 }

++ (NSString *)moduleName
+{
+  return @"NativeLocalStorage";
+}

@end
```

程式碼並未大幅變更。與直接創建 `NSUserDefaults` 參考不同，現在改為使用 Swift 實作創建新的 `NativeLocalStorage`，當原生模組函數被調用時，會將調用轉發至 Swift 實作的 `NativeLocalStorage`。

請記得導入 `"SampleApp-Swift.h"` 標頭檔。這是 Xcode 自動產生的標頭檔，包含您 Swift 檔案的公開 API，並以 Objective-C 可使用的格式呈現。標頭檔中的 `SampleApp` 部分實際上是您的應用程式名稱，若您創建應用時使用了**不同於** `SampleApp` 的名稱，則需要相應修改。

Note also that the `RCT_EXPORT_MODULE` macro is not required anymore, because native modules are registered using the `package.json` as described [here](/docs/next/turbo-native-modules-introduction?platforms=ios#register-the-native-module-in-your-app).

此方法雖然會在介面中引入一些程式碼重複，但能讓您以最少額外工作量重用現有程式碼庫中的 Swift 程式碼。

### 實作橋接標頭檔

:::note
如果您是函式庫作者，正在開發一個將作為獨立函式庫分發的原生模組，則此步驟並非必要。
:::

將 Swift 程式碼與 Objective-C++ 對應部分連接的最後一個必要步驟是橋接標頭檔（bridging header）。

橋接標頭檔是一個可以導入所有需要讓 Swift 程式碼看到的 Objective-C 標頭檔的檔案。

您的程式碼庫中可能已經有一個橋接標頭檔，但如果沒有，您可以按照以下步驟創建一個新的：

1. 在 Xcode 中，創建一個新檔案並命名為 `"SampleApp-Bridging-Header.h"`
2. 更新 `"SampleApp-Bridging-Header.h"` 的內容如下：

```diff title="SampleApp-Bridging-Header.h"
//
//  Use this file to import your target's public headers that you would like to expose to Swift.
//

+ #import <React-RCTAppDelegate/RCTDefaultReactNativeFactoryDelegate.h>
```

3. 在專案中連結橋接標頭檔：
   1. 在專案導航器中，選擇您的應用名稱（左側的 `SampleApp`）
   2. 點擊 `Build Settings`
   3. 搜尋 `"Bridging Header"`
   4. 添加橋接標頭檔的相對路徑，例如範例中的 `SampleApp-Bridging-Header.h`

![橋接標頭檔](/docs/assets/BridgingHeader.png)

## 建置並運行您的應用

現在您可以按照[原生模組指南](/docs/turbo-native-modules-introduction#build-and-run-your-code-on-a-simulator)的最後一步操作，您應該會看到應用運行時使用了以 Swift 編寫的原生模組。