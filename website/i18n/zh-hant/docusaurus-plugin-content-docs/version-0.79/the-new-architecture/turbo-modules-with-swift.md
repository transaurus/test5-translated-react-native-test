# iOS - 在原生模組中使用 Swift

Swift 是開發 iOS 原生應用程式的官方預設語言。

本指南將帶您探索如何使用 Swift 編寫原生模組。

:::note
React Native 的核心主要使用 C++ 編寫，而 Swift 與 C++ 之間的互通性並不理想，儘管 Apple 開發了[互通層](https://www.swift.org/documentation/cxx-interop/)。

因此，由於語言間的不相容性，本指南中您將編寫的模組不會是純 Swift 實現。您需要編寫一些 Objective-C++ 的黏合程式碼，但本指南的目標是盡量減少所需的 Objective-C++ 程式碼量。如果您正在將現有原生模組從舊架構遷移到新架構，這種方法應能讓您重用大部分程式碼。
:::

This guide starts from the iOS implementation of the [Native Module](/docs/next/turbo-native-modules-introduction) guide.
Make sure to be familiar with that guide before diving into this one, potentially implementing the example in the guide.

## 適配器模式

目標是使用 Swift 模組實現所有業務邏輯，並在 Objective-C++ 中建立一個薄黏合層，以連接應用程式與 Swift 實現。

您可以透過運用[適配器](https://en.wikipedia.org/wiki/Adapter_pattern)設計模式來實現這一點，以連接 Swift 模組與 Objective-C++ 層。

Objective-C++ 物件由 React Native 創建，它持有對 Swift 模組的引用並管理其生命週期。Objective-C++ 物件將所有方法調用轉發給 Swift。

### 創建 Swift 模組

第一步是將實現從 Objective-C++ 層移至 Swift 層。

為實現這一點，請按照以下步驟操作：

1. 在 Xcode 專案中創建一個新空白檔案，並命名為 `NativeLocalStorage.swift`
2. 在 Swift 模組中添加如下實現：

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

接著，您需要更新 `RCTNativeLocalStorage` 的實現，以便能夠創建 Swift 模組並調用其方法。

1. 開啟 `RCTNativeLocalStorage.mm` 檔案
2. 按如下方式更新：

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

程式碼並未真正改變。與直接創建 `NSUserDefaults` 的引用不同，您現在使用 Swift 實現創建一個新的 `NativeLocalStorage`，並且每當調用原生模組函數時，調用會被轉發到 Swift 實現的 `NativeLocalStorage`。

記得導入 `"SampleApp-Swift.h"` 標頭檔。這是 Xcode 自動生成的標頭檔，包含 Swift 檔案的公開 API，並以 Objective-C 可消費的格式呈現。標頭檔中的 `SampleApp` 部分實際上是您的應用程式名稱，因此如果您創建的應用程式名稱**不同**於 `SampleApp`，則需要更改它。

另外請注意，不再需要 `RCT_EXPORT_MODULE` 宏，因為原生模組現在是透過 `package.json` 註冊，如[此處](/docs/next/turbo-native-modules-introduction?platforms=ios#register-the-native-module-in-your-app)所述。

這種方法在介面上引入了一些程式碼重複，但它允許您以最小的額外努力重用現有程式碼庫中的 Swift 程式碼。

### 實現橋接標頭檔

:::note
若您是函式庫開發者，正在開發將作為獨立函式庫分發的原生模組，則此步驟非必要。
:::

將 Swift 程式碼與 Objective-C++ 對應部分連接的最後必要步驟是橋接標頭檔（bridging header）。

橋接標頭檔是一個可匯入所有需讓 Swift 程式碼可見的 Objective-C 標頭檔的檔案。

您的程式碼庫中可能已有橋接標頭檔，若尚未建立，請依下列步驟新增：

1. 在 Xcode 中建立新檔案，命名為 `"SampleApp-Bridging-Header.h"`
2. 更新 `"SampleApp-Bridging-Header.h"` 內容如下：

```diff title="SampleApp-Bridging-Header.h"
//
//  Use this file to import your target's public headers that you would like to expose to Swift.
//

+ #import <React-RCTAppDelegate/RCTDefaultReactNativeFactoryDelegate.h>
```

3. 在專案中連結橋接標頭檔：
   1. 在專案導覽器中選擇您的應用程式名稱（左側的 `SampleApp`）
   2. 點擊 `Build Settings`
   3. 篩選 `"Bridging Header"`
   4. 新增橋接標頭檔的相對路徑，範例中為 `SampleApp-Bridging-Header.h`

![橋接標頭檔](/docs/assets/BridgingHeader.png)

## 建置並執行應用程式

現在您可以依照[原生模組指南](/docs/turbo-native-modules-introduction#build-and-run-your-code-on-a-simulator)的最後步驟操作，即可看到應用程式執行時使用 Swift 編寫的原生模組。