import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

# Native Modules 生命週期

在 React Native 中，Native Modules 是單例模式。Native Module 基礎設施會在首次存取時惰性建立一個 Native Module，並在應用程式需要時保持其存在。這是一種效能優化，可避免在應用程式啟動時急切建立 Native Module 的開銷，並確保更快的啟動時間。

在純 React Native 應用程式中，Native Modules 只會建立一次且永遠不會被銷毀。然而，在更複雜的應用程式中，可能會遇到需要銷毀並重新建立 Native Modules 的情況。例如，設想一個混合了部分原生視圖與 React Native 介面的棕地應用程式，如[與現有應用程式整合指南](/docs/integration-with-existing-apps)所述。在這種情況下，當使用者導航離開 React Native 介面時銷毀 React Native 實例，並在使用者返回該介面時重新建立它，可能是合理的做法。

當這種情況發生時，無狀態的 Native Modules 不會造成任何問題。然而，對於有狀態的 Native Modules，可能需要正確地使其失效，以確保狀態被重置且資源被釋放。

在本指南中，你將探索如何正確初始化和使 Native Module 失效。本指南假設你已熟悉如何撰寫 Native Module 並能輕鬆撰寫原生程式碼。如果你不熟悉 Native Modules，請先閱讀 [Native Modules 指南](/docs/next/turbo-native-modules-introduction)。

## Android

在 Android 上，所有 Native Modules 都已實作了 [TurboModule](https://github.com/facebook/react-native/blob/main/packages/react-native/ReactAndroid/src/main/java/com/facebook/react/turbomodule/core/interfaces/TurboModule.kt) 介面，該介面定義了兩個方法：`initialize()` 和 `invalidate()`。

The `initialize()` method is called by the Native Module infrastructure when the Native Module is created. This is the best place to put all the initialization code that needs access to the ReactApplicationContext, for example. These are some Native Modules from core that implements the `initialize()` method: [BlobModule](https://github.com/facebook/react-native/blob/0617accecdcb11159ba15c34885f294bc206aa89/packages/react-native/ReactAndroid/src/main/java/com/facebook/react/modules/blob/BlobModule.java#L155-L157), [NetworkingModule](https://github.com/facebook/react-native/blob/0617accecdcb11159ba15c34885f294bc206aa89/packages/react-native/ReactAndroid/src/main/java/com/facebook/react/modules/network/NetworkingModule.java#L193-L197).

The `invalidate()` method is called by the Native Module infrastructure when the Native Module is destroyed. This is the best place to put all the cleanup code, resetting the Native Module state and release resources that are no longer needed, such as memory and files. These are some Native Modules from core that implements the `invalidate()` method: [DeviceInfoModule](https://github.com/facebook/react-native/blob/0617accecdcb11159ba15c34885f294bc206aa89/packages/react-native/ReactAndroid/src/main/java/com/facebook/react/modules/deviceinfo/DeviceInfoModule.kt#L72-L76), [NetworkModule](https://github.com/facebook/react-native/blob/0617accecdcb11159ba15c34885f294bc206aa89/packages/react-native/ReactAndroid/src/main/java/com/facebook/react/modules/network/NetworkingModule.java#L200-L212)

## iOS

On iOS, Native Modules conforms to the [`RCTTurboModule`](https://github.com/facebook/react-native/blob/0617accecdcb11159ba15c34885f294bc206aa89/packages/react-native/ReactCommon/react/nativemodule/core/platform/ios/ReactCommon/RCTTurboModule.h#L196-L200) protocol. However, this protocol does not expose the `initialize` and `invalidate` method that are exposed by the Android's `TurboModule` class.

相反地，在 iOS 平台上，存在兩個額外的協定：[`RCTInitializing`](https://github.com/facebook/react-native/blob/0617accecdcb11159ba15c34885f294bc206aa89/packages/react-native/React/Base/RCTInitializing.h) 和 [`RCTInvalidating`](https://github.com/facebook/react-native/blob/0617accecdcb11159ba15c34885f294bc206aa89/packages/react-native/React/Base/RCTInvalidating.h)。這些協定分別用於定義 `initialize` 和 `invalidate` 方法。

若您的模組需要執行初始化程式碼，則可遵循 `RCTInitializing` 協定並實作 `initialize` 方法。具體步驟如下：

1. 修改 `NativeModule.h` 檔案，加入以下程式碼：

```diff title="NativeModule.h"
+ #import <React/RCTInitializing.h>

//...

- @interface NativeModule : NSObject <NativeModuleSpec>
+ @interface NativeModule : NSObject <NativeModuleSpec, RCTInitializing>
//...
@end
```

2. Implement the `initialize` method in the `NativeModule.mm` file:

```diff title="NativeModule.mm"
// ...

@implementation NativeModule

+- (void)initialize {
+ // add the initialization code here
+}

@end
```

以下是核心模組中實作 `initialize` 方法的範例：[RCTBlobManager](https://github.com/facebook/react-native/blob/0617accecdcb11159ba15c34885f294bc206aa89/packages/react-native/Libraries/Blob/RCTBlobManager.mm#L58-L68)、[RCTTiming](https://github.com/facebook/react-native/blob/0617accecdcb11159ba15c34885f294bc206aa89/packages/react-native/React/CoreModules/RCTTiming.mm#L121-L124)。

若您的模組需要執行清理程式碼，則可遵循 `RCTInvalidating` 協定並實作 `invalidate` 方法。具體步驟如下：

1. 修改 `NativeModule.h` 檔案，加入以下程式碼：

```diff title="NativeModule.h"
+ #import <React/RCTInvalidating.h>

//...

- @interface NativeModule : NSObject <NativeModuleSpec>
+ @interface NativeModule : NSObject <NativeModuleSpec, RCTInvalidating>

//...

@end
```

2. Implement the `invalidate` method in the `NativeModule.mm` file:

```diff title="NativeModule.mm"

// ...

@implementation NativeModule

+- (void)invalidate {
+ // add the cleanup code here
+}

@end
```

以下是核心模組中實作 `invalidate` 方法的範例：[RCTAppearance](https://github.com/facebook/react-native/blob/0617accecdcb11159ba15c34885f294bc206aa89/packages/react-native/React/CoreModules/RCTAppearance.mm#L151-L155)、[RCTDeviceInfo](https://github.com/facebook/react-native/blob/0617accecdcb11159ba15c34885f294bc206aa89/packages/react-native/React/CoreModules/RCTDeviceInfo.mm#L127-L133)。