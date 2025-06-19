---
id: native-modules-ios
title: iOS Native Modules
---

import NativeDeprecated from './the-new-architecture/\_markdown_native_deprecation.mdx'
import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<NativeDeprecated />

歡迎來到 iOS 原生模組開發。請先閱讀 [原生模組簡介](native-modules-intro) 了解原生模組的基本概念。

## 建立日曆原生模組

In the following guide you will create a native module, `CalendarModule`, that will allow you to access Apple's calendar APIs from JavaScript. By the end you will be able to call `CalendarModule.createCalendarEvent('Dinner Party', 'My House');` from JavaScript, invoking a native method that creates a calendar event.

### 設定環境

首先，請在 Xcode 中開啟您 React Native 應用程式內的 iOS 專案。React Native 應用中的 iOS 專案路徑如下：

<figure>
  <img src="/docs/assets/native-modules-ios-open-project.png" width="500" alt="Image of opening up an iOS project within a React Native app inside of xCode." />
  <figcaption>Image of where you can find your iOS project</figcaption>
</figure>

我們建議使用 Xcode 來編寫原生程式碼。Xcode 專為 iOS 開發設計，能幫助您快速解決程式碼語法等小錯誤。

### 建立自訂原生模組檔案

第一步是建立我們的主要自訂原生模組標頭檔與實作檔。建立一個新檔案命名為 `RCTCalendarModule.h`

<figure>
  <img src="/docs/assets/native-modules-ios-add-class.png" width="500" alt="Image of creating a class called  RCTCalendarModule.h." />
  <figcaption>Image of creating a custom native module file within the same folder as AppDelegate</figcaption>
</figure>

並加入以下內容：

```objectivec
//  RCTCalendarModule.h
#import <React/RCTBridgeModule.h>
@interface RCTCalendarModule : NSObject <RCTBridgeModule>
@end

```

您可以根據開發的原生模組特性自由命名類別。此處命名為 `RCTCalendarModule` 因為您正在建立日曆原生模組。由於 Objective-C 不像 Java 或 C++ 有語言層級的命名空間支援，慣例是在類別名稱前加上前綴字串。這可以是應用程式名稱或基礎架構名稱的縮寫。本例中的 RCT 代表 React。

如下所示，CalendarModule 類別實作了 `RCTBridgeModule` 協定。原生模組就是實作 `RCTBridgeModule` 協定的 Objective-C 類別。

接著開始實作原生模組。在同個資料夾建立對應的實作檔 `RCTCalendarModule.m` 並包含以下內容：

```objectivec
// RCTCalendarModule.m
#import "RCTCalendarModule.h"

@implementation RCTCalendarModule

// To export a module named RCTCalendarModule
RCT_EXPORT_MODULE();

@end

```

### 模組名稱

目前您的 `RCTCalendarModule.m` 原生模組僅包含 `RCT_EXPORT_MODULE` 巨集，該巨集會將原生模組類別匯出並註冊到 React Native。`RCT_EXPORT_MODULE` 巨集也可接受一個可選參數，用於指定模組在 JavaScript 程式碼中的存取名稱。

請注意此參數不是字串常值。下方範例中傳入的是 `RCT_EXPORT_MODULE(CalendarModuleFoo)` 而非 `RCT_EXPORT_MODULE("CalendarModuleFoo")`。

```objectivec
// To export a module named CalendarModuleFoo
RCT_EXPORT_MODULE(CalendarModuleFoo);
```

接著在 JS 中可這樣存取原生模組：

```tsx
const {CalendarModuleFoo} = ReactNative.NativeModules;
```

若未指定名稱，JavaScript 模組名稱會與 Objective-C 類別名稱相同，但會移除 "RCT" 或 "RK" 前綴。

讓我們遵循下方範例，不帶參數呼叫 `RCT_EXPORT_MODULE`。結果模組會以 `CalendarModule` 名稱暴露給 React Native，因為這是移除 RCT 前綴後的 Objective-C 類別名稱。

```objectivec
// Without passing in a name this will export the native module name as the Objective-C class name with “RCT” removed
RCT_EXPORT_MODULE();
```

接著在 JS 中可這樣存取原生模組：

```tsx
const {CalendarModule} = ReactNative.NativeModules;
```

### 將原生方法匯出至 JavaScript

React Native will not expose any methods in a native module to JavaScript unless explicitly told to. This can be done using the `RCT_EXPORT_METHOD` macro. Methods written in the `RCT_EXPORT_METHOD` macro are asynchronous and the return type is therefore always void. In order to pass a result from a `RCT_EXPORT_METHOD` method to JavaScript you can use callbacks or emit events (covered below). Let’s go ahead and set up a native method for our `CalendarModule` native module using the `RCT_EXPORT_METHOD` macro. Call it `createCalendarEvent()` and for now have it take in name and location arguments as strings. Argument type options will be covered shortly.

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)name location:(NSString *)location)
{
}
```

> Please note that the `RCT_EXPORT_METHOD` macro will not be necessary with TurboModules unless your method relies on RCT argument conversion (see argument types below). Ultimately, React Native will remove `RCT_EXPORT_MACRO,` so we discourage people from using `RCTConvert`. Instead, you can do the argument conversion within the method body.

在構建 `createCalendarEvent()` 方法的功能之前，先在方法中添加一個控制台日誌，以便確認它已從 React Native 應用程序中的 JavaScript 調用。使用 React 的 `RCTLog` API。讓我們在文件的頂部導入該標頭，然後添加日誌調用。

```objectivec
#import <React/RCTLog.h>
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)name location:(NSString *)location)
{
 RCTLogInfo(@"Pretending to create an event %@ at %@", name, location);
}
```

### 同步方法

你可以使用 `RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD` 來創建一個同步的原生方法。

```objectivec
RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD(getName)
{
return [[UIDevice currentDevice] name];
}
```

該方法的返回類型必須是對象類型（id）並且可序列化為 JSON。這意味著該方法只能返回 nil 或 JSON 值（例如 NSNumber、NSString、NSArray、NSDictionary）。

目前，我們不建議使用同步方法，因為同步調用方法可能會帶來嚴重的性能損失，並在你的原生模組中引入與線程相關的錯誤。此外，請注意，如果你選擇使用 `RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD`，你的應用程序將無法再使用 Google Chrome 調試器。這是因為同步方法要求 JS VM 與應用程序共享內存。對於 Google Chrome 調試器，React Native 在 Google Chrome 的 JS VM 中運行，並通過 WebSockets 與移動設備進行異步通信。

### 測試你構建的內容

此時，你已經在 iOS 中為你的原生模組設置了基本的框架。通過訪問原生模組並在 JavaScript 中調用其導出的方法來測試它。

在你的應用程序中找到一個你想要添加調用原生模組 `createCalendarEvent()` 方法的地方。下面是一個組件 `NewModuleButton` 的示例，你可以在你的應用中添加它。你可以在 `NewModuleButton` 的 `onPress()` 函數中調用原生模組。

```tsx
import React from 'react';
import {NativeModules, Button} from 'react-native';

const NewModuleButton = () => {
  const onPress = () => {
    console.log('We will invoke the native module here!');
  };

  return (
    <Button
      title="Click to invoke your native module!"
      color="#841584"
      onPress={onPress}
    />
  );
};

export default NewModuleButton;
```

為了從 JavaScript 訪問你的原生模組，你需要首先從 React Native 導入 `NativeModules`：

```tsx
import {NativeModules} from 'react-native';
```

You can then access the `CalendarModule` native module off of `NativeModules`.

```tsx
const {CalendarModule} = NativeModules;
```

Now that you have the CalendarModule native module available, you can invoke your native method `createCalendarEvent()`. Below it is added to the `onPress()` method in `NewModuleButton`:

```tsx
const onPress = () => {
  CalendarModule.createCalendarEvent('testName', 'testLocation');
};
```

最後一步是重新構建 React Native 應用程序，以便你可以獲得最新的原生代碼（包含你的新原生模組！）。在你的命令行中，位於 react native 應用程序的位置，運行以下命令：

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

### 迭代構建

當你按照這些指南進行開發並迭代原生模組時，你需要重新編譯應用程式才能從 JavaScript 端獲取最新的變更。這是因為你所編寫的程式碼位於應用程式的原生部分。雖然 React Native 的 Metro 打包工具可以監控 JavaScript 的變更並即時重新打包 JS 程式碼，但它不會對原生程式碼進行同樣的操作。因此，如果你想測試最新的原生變更，就需要使用上述指令重新編譯。

### 回顧✨

你現在應該能夠在 JavaScript 中呼叫原生模組的 `createCalendarEvent()` 方法。由於你在函式中使用了 `RCTLog`，你可以透過[在應用程式中啟用除錯模式](https://reactnative.dev/docs/debugging#chrome-developer-tools)並查看 Chrome 的 JS 控制台或行動應用除錯工具 Flipper 來確認原生方法是否被呼叫。每次呼叫原生模組方法時，你都應該會看到 `RCTLogInfo(@"Pretending to create an event %@ at %@", name, location);` 的訊息。

<figure>
  <img src="/docs/assets/native-modules-ios-logs.png" width="1000" alt="Image of logs." />
  <figcaption>Image of iOS logs in Flipper</figcaption>
</figure>

至此，你已經建立了一個 iOS 原生模組，並從 React Native 應用程式的 JavaScript 端呼叫了其中的方法。你可以繼續閱讀，了解更多關於原生模組方法的參數類型，以及如何在原生模組中設定回調函式和 Promise 的內容。

## 超越日曆原生模組

### 更好的原生模組匯出方式

像上面那樣從 `NativeModules` 中提取原生模組的方式有點笨拙。

為了讓使用你的原生模組的開發者不必每次都這樣做，你可以為模組建立一個 JavaScript 包裝器。建立一個名為 NativeCalendarModule.js 的新 JavaScript 檔案，內容如下：

```tsx
/**
* This exposes the native CalendarModule module as a JS module. This has a
* function 'createCalendarEvent' which takes the following parameters:

* 1. String name: A string representing the name of the event
* 2. String location: A string representing the location of the event
*/
import {NativeModules} from 'react-native';
const {CalendarModule} = NativeModules;
export default CalendarModule;
```

這個 JavaScript 檔案也是你添加 JavaScript 端功能的好地方。例如，如果你使用 TypeScript 這樣的類型系統，你可以在這裡為你的原生模組添加類型註解。雖然 React Native 目前還不支援原生到 JavaScript 的類型安全，但有了這些類型註解，你所有的 JavaScript 程式碼都將是類型安全的。這些註解也將幫助你在未來更容易地切換到類型安全的原生模組。以下是一個為日曆模組添加類型安全的範例：

```tsx
/**
 * This exposes the native CalendarModule module as a JS module. This has a
 * function 'createCalendarEvent' which takes the following parameters:
 *
 * 1. String name: A string representing the name of the event
 * 2. String location: A string representing the location of the event
 */
import {NativeModules} from 'react-native';
const {CalendarModule} = NativeModules;
interface CalendarInterface {
  createCalendarEvent(name: string, location: string): void;
}
export default CalendarModule as CalendarInterface;
```

在其他 JavaScript 檔案中，你可以這樣存取原生模組並呼叫其方法：

```tsx
import NativeCalendarModule from './NativeCalendarModule';
NativeCalendarModule.createCalendarEvent('foo', 'bar');
```

> 注意：這假設你導入 `CalendarModule` 的位置與 `NativeCalendarModule.js` 位於相同的目錄層級。請根據需要更新相對導入路徑。

### 參數類型

當在 JavaScript 中呼叫原生模組方法時，React Native 會將參數從 JavaScript 物件轉換為對應的 Objective-C/Swift 物件。因此，舉例來說，如果你的 Objective-C 原生模組方法接受一個 NSNumber，在 JavaScript 中你需要用數字來呼叫該方法。React Native 會為你處理轉換。以下是原生模組方法支援的參數類型及其對應的 JavaScript 等效類型清單。

| Objective-C                                   | JavaScript         |
| --------------------------------------------- | ------------------ |
| NSString                                      | string, ?string    |
| BOOL                                          | boolean            |
| double                                        | number             |
| NSNumber                                      | ?number            |
| NSArray                                       | Array, ?Array      |
| NSDictionary                                  | Object, ?Object    |
| RCTResponseSenderBlock                        | Function (success) |
| RCTResponseSenderBlock, RCTResponseErrorBlock | Function (failure) |
| RCTPromiseResolveBlock, RCTPromiseRejectBlock | Promise            |

> 以下類型目前受支援，但在 TurboModules 中將不再支援。請避免使用它們。
>
> - Function (failure) -> RCTResponseErrorBlock
> - Number -> NSInteger
> - Number -> CGFloat
> - Number -> float

對於 iOS，你也可以使用 `RCTConvert` 類支援的任何參數類型來編寫原生模組方法（詳見 [RCTConvert](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTConvert.h) 以了解支援的內容）。RCTConvert 輔助函式都接受一個 JSON 值作為輸入，並將其映射到原生的 Objective-C 類型或類別。

### 匯出常數

原生模組可以透過覆寫原生方法 `constantsToExport()` 來匯出常數。以下範例中覆寫了 `constantsToExport()`，並返回一個字典，其中包含一個預設事件名稱屬性，你可以在 JavaScript 中這樣存取：

```objectivec
- (NSDictionary *)constantsToExport
{
 return @{ @"DEFAULT_EVENT_NAME": @"New Event" };
}
```

接著可以透過在 JavaScript 中呼叫原生模組的 `getConstants()` 來存取這個常數，如下所示：

```tsx
const {DEFAULT_EVENT_NAME} = CalendarModule.getConstants();
console.log(DEFAULT_EVENT_NAME);
```

Technically, it is possible to access constants exported in `constantsToExport()` directly off the `NativeModule` object. This will no longer be supported with TurboModules, so we encourage the community to switch to the above approach to avoid necessary migration down the line.

> 請注意，常數僅在初始化時匯出，因此若在執行階段變更 `constantsToExport()` 的值，並不會影響 JavaScript 環境。

在 iOS 上，若您覆寫了 `constantsToExport()`，則應同時實作 `+ requiresMainQueueSetup`，讓 React Native 知道您的模組是否需要先於任何 JavaScript 程式碼執行前在主執行緒初始化。否則您會看到警告，提示未來您的模組可能會在背景執行緒初始化，除非您透過 `+ requiresMainQueueSetup:` 明確選擇退出。若您的模組不需要存取 UIKit，則應對 `+ requiresMainQueueSetup` 回傳 NO。

### 回呼函式

原生模組還支援一種特殊的參數類型——回呼函式（callback）。回呼函式用於在非同步方法中將資料從 Objective-C 傳遞到 JavaScript，也可用於從原生端非同步執行 JavaScript。

在 iOS 中，回呼函式是透過 `RCTResponseSenderBlock` 類型實作的。以下範例將回呼參數 `myCallBack` 加入 `createCalendarEventMethod()`：

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title
                location:(NSString *)location
                myCallback:(RCTResponseSenderBlock)callback)

```

接著您可以在原生函式中呼叫回呼，並傳遞任何您想提供給 JavaScript 的結果（以陣列形式）。請注意，`RCTResponseSenderBlock` 僅接受一個參數——傳遞給 JavaScript 回呼的參數陣列。以下範例將回傳先前呼叫中建立的事件 ID。

> 需特別強調的是，回呼函式不會在原生物件完成後立即觸發——請記住這是一種非同步通訊。

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title location:(NSString *)location callback: (RCTResponseSenderBlock)callback)
{
 NSInteger eventId = ...
 callback(@[@(eventId)]);

 RCTLogInfo(@"Pretending to create an event %@ at %@", title, location);
}

```

然後可以透過以下方式在 JavaScript 中存取此方法：

```tsx
const onSubmit = () => {
  CalendarModule.createCalendarEvent(
    'Party',
    '04-12-2020',
    eventId => {
      console.log(`Created a new event with id ${eventId}`);
    },
  );
};
```

原生模組應僅呼叫其回呼一次。不過，它可以儲存回呼並稍後再呼叫。這種模式常用於封裝需要委派（delegate）的 iOS API——參見 [`RCTAlertManager`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/CoreModules/RCTAlertManager.mm) 的範例。若回呼從未被呼叫，會導致部分記憶體洩漏。

回呼函式的錯誤處理有兩種方式。第一種是遵循 Node 的慣例，將傳遞給回呼陣列的第一個參數視為錯誤物件。

```objectivec
RCT_EXPORT_METHOD(createCalendarEventCallback:(NSString *)title location:(NSString *)location callback: (RCTResponseSenderBlock)callback)
{
  NSNumber *eventId = [NSNumber numberWithInt:123];
  callback(@[[NSNull null], eventId]);
}
```

在 JavaScript 中，您可以檢查第一個參數來判斷是否有錯誤傳入：

```tsx
const onPress = () => {
  CalendarModule.createCalendarEventCallback(
    'testName',
    'testLocation',
    (error, eventId) => {
      if (error) {
        console.error(`Error found! ${error}`);
      }
      console.log(`event id ${eventId} returned`);
    },
  );
};
```

另一種選項是使用兩個獨立的回呼：onFailure 和 onSuccess。

```objectivec
RCT_EXPORT_METHOD(createCalendarEventCallback:(NSString *)title
                  location:(NSString *)location
                  errorCallback: (RCTResponseSenderBlock)errorCallback
                  successCallback: (RCTResponseSenderBlock)successCallback)
{
  @try {
    NSNumber *eventId = [NSNumber numberWithInt:123];
    successCallback(@[eventId]);
  }

  @catch ( NSException *e ) {
    errorCallback(@[e]);
  }
}
```

然後在 JavaScript 中，您可以為錯誤和成功回應分別添加回呼：

```tsx
const onPress = () => {
  CalendarModule.createCalendarEventCallback(
    'testName',
    'testLocation',
    error => {
      console.error(`Error found! ${error}`);
    },
    eventId => {
      console.log(`event id ${eventId} returned`);
    },
  );
};
```

If you want to pass error-like objects to JavaScript, use `RCTMakeError` from [`RCTUtils.h.`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTUtils.h) Right now this only passes an Error-shaped dictionary to JavaScript, but React Native aims to automatically generate real JavaScript Error objects in the future. You can also provide a `RCTResponseErrorBlock` argument, which is used for error callbacks and accepts an `NSError \* object`. Please note that this argument type will not be supported with TurboModules.

### Promise

原生模組也可以實現 Promise，這能簡化您的 JavaScript 程式碼，特別是在使用 ES2016 的 `async/await` 語法時。當原生模組方法的最後一個參數是 `RCTPromiseResolveBlock` 和 `RCTPromiseRejectBlock` 時，其對應的 JS 方法將回傳一個 JS Promise 物件。

將上述程式碼重構為使用 Promise 而非回呼函式的範例如下：

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title
                 location:(NSString *)location
                 resolver:(RCTPromiseResolveBlock)resolve
                 rejecter:(RCTPromiseRejectBlock)reject)
{
 NSInteger eventId = createCalendarEvent();
 if (eventId) {
    resolve(@(eventId));
  } else {
    reject(@"event_failure", @"no event id returned", nil);
  }
}

```

該方法的 JavaScript 對應部分會返回一個 Promise。這意味著您可以在 async 函數中使用 `await` 關鍵字來調用它並等待其結果：

```tsx
const onSubmit = async () => {
  try {
    const eventId = await CalendarModule.createCalendarEvent(
      'Party',
      'my house',
    );
    console.log(`Created a new event with id ${eventId}`);
  } catch (e) {
    console.error(e);
  }
};
```

### 向 JavaScript 發送事件

Native modules can signal events to JavaScript without being invoked directly. For example, you might want to signal to JavaScript a reminder that a calendar event from the native iOS calendar app will occur soon. The preferred way to do this is to subclass `RCTEventEmitter`, implement `supportedEvents` and call self `sendEventWithName`:

更新您的頭文件類以導入 `RCTEventEmitter` 並繼承 `RCTEventEmitter`：

```objectivec
//  CalendarModule.h

#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>

@interface CalendarModule : RCTEventEmitter <RCTBridgeModule>
@end

```

JavaScript 代碼可以通過在您的模組周圍創建一個新的 `NativeEventEmitter` 實例來訂閱這些事件。

You will receive a warning if you expend resources unnecessarily by emitting an event while there are no listeners. To avoid this, and to optimize your module's workload (e.g. by unsubscribing from upstream notifications or pausing background tasks), you can override `startObserving` and `stopObserving` in your `RCTEventEmitter` subclass.

```objectivec
@implementation CalendarManager
{
  bool hasListeners;
}

// Will be called when this module's first listener is added.
-(void)startObserving {
    hasListeners = YES;
    // Set up any upstream listeners or background tasks as necessary
}

// Will be called when this module's last listener is removed, or on dealloc.
-(void)stopObserving {
    hasListeners = NO;
    // Remove upstream listeners, stop unnecessary background tasks
}

- (void)calendarEventReminderReceived:(NSNotification *)notification
{
  NSString *eventName = notification.userInfo[@"name"];
  if (hasListeners) {// Only send events if anyone is listening
    [self sendEventWithName:@"EventReminder" body:@{@"name": eventName}];
  }
}

```

### 線程處理

除非原生模組提供自己的方法隊列，否則不應對其被調用的線程做出任何假設。目前，如果原生模組未提供方法隊列，React Native 會為其創建一個單獨的 GCD 隊列並在那裡調用其方法。請注意，這是一個實現細節，可能會發生變化。如果您想為原生模組明確提供一個方法隊列，請在原生模組中覆寫 `(dispatch_queue_t) methodQueue` 方法。例如，如果需要使用僅限主線程的 iOS API，應通過以下方式指定：

```objectivec
- (dispatch_queue_t)methodQueue
{
  return dispatch_get_main_queue();
}
```

同樣，如果某個操作可能需要很長時間才能完成，原生模組可以指定自己的隊列來運行操作。再次強調，目前 React Native 會為您的原生模組提供一個單獨的方法隊列，但這是一個您不應依賴的實現細節。如果您不提供自己的方法隊列，將來您的原生模組的長時間運行操作可能會阻塞在其他不相關原生模組上執行的異步調用。例如，這裡的 `RCTAsyncLocalStorage` 模組創建了自己的隊列，這樣 React 隊列就不會被潛在的慢速磁盤訪問阻塞。

```objectivec
- (dispatch_queue_t)methodQueue
{
 return dispatch_queue_create("com.facebook.React.AsyncLocalStorageQueue", DISPATCH_QUEUE_SERIAL);
}
```

指定的 `methodQueue` 將由模組中的所有方法共享。如果只有一個方法是長時間運行的（或由於某些原因需要與其他方法在不同的隊列上運行），您可以在方法內部使用 `dispatch_async` 將該特定方法的代碼放在另一個隊列上執行，而不影響其他方法：

```objectivec
RCT_EXPORT_METHOD(doSomethingExpensive:(NSString *)param callback:(RCTResponseSenderBlock)callback)
{
 dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
   // Call long-running code on background thread
   ...
   // You can invoke callback from any thread/queue
   callback(@[...]);
 });
}

```

> Sharing dispatch queues between modules
>
> The `methodQueue` method will be called once when the module is initialized, and then retained by React Native, so there is no need to keep a reference to the queue yourself, unless you wish to make use of it within your module. However, if you wish to share the same queue between multiple modules then you will need to ensure that you retain and return the same queue instance for each of them.

### 依賴注入

React Native 會自動創建並初始化任何已註冊的原生模組。但是，您可能希望創建並初始化自己的模組實例，例如注入依賴項。

您可以通過創建一個實現 `RCTBridgeDelegate` 協議的類，使用該委託作為參數初始化一個 `RCTBridge`，然後使用初始化的橋接器初始化一個 `RCTRootView` 來實現這一點。

```objectivec
id<RCTBridgeDelegate> moduleInitialiser = [[classThatImplementsRCTBridgeDelegate alloc] init];

RCTBridge *bridge = [[RCTBridge alloc] initWithDelegate:moduleInitialiser launchOptions:nil];

RCTRootView *rootView = [[RCTRootView alloc]
                        initWithBridge:bridge
                            moduleName:kModuleName
                     initialProperties:nil];
```

### 導出 Swift

Swift 不支持宏，因此將原生模組及其方法暴露給 React Native 中的 JavaScript 需要更多的設置。然而，它的工作方式相對相同。假設您有相同的 `CalendarModule`，但作為一個 Swift 類：

```swift
// CalendarManager.swift

@objc(CalendarManager)
class CalendarManager: NSObject {

 @objc(addEvent:location:date:)
 func addEvent(_ name: String, location: String, date: NSNumber) -> Void {
   // Date is ready to use!
 }

 @objc
 func constantsToExport() -> [String: Any]! {
   return ["someKey": "someValue"]
 }

}
```

> 必須使用 `@objc` 修飾符，以確保類別和函式能正確導出至 Objective-C 運行時環境。

接著建立一個私有實作檔案，用於向 React Native 註冊必要資訊：

```objectivec
// CalendarManagerBridge.m
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(CalendarManager, NSObject)

RCT_EXTERN_METHOD(addEvent:(NSString *)name location:(NSString *)location date:(nonnull NSNumber *)date)

@end
```

For those of you new to Swift and Objective-C, whenever you [mix the two languages in an iOS project](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html), you will also need an additional bridging file, known as a bridging header, to expose the Objective-C files to Swift. Xcode will offer to create this header file for you if you add your Swift file to your app through the Xcode `File>New File` menu option. You will need to import `RCTBridgeModule.h` in this header file.

```objectivec
// CalendarManager-Bridging-Header.h
#import <React/RCTBridgeModule.h>
```

你也可以使用 `RCT_EXTERN_REMAP_MODULE` 和 `_RCT_EXTERN_REMAP_METHOD` 來變更導出模組或方法在 JavaScript 中的名稱。更多資訊請參閱 [`RCTBridgeModule`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTBridgeModule.h)。

> 第三方模組重要注意事項：僅 Xcode 9 及以上版本支援包含 Swift 的靜態函式庫。若你在模組的 iOS 靜態函式庫中使用 Swift，主應用程式專案必須包含 Swift 程式碼與橋接標頭檔才能成功建置 Xcode 專案。若應用程式專案不含任何 Swift 程式碼，可暫時以一個空白的 .swift 檔案和空白橋接標頭檔作為解決方案。

### 保留方法名稱

#### invalidate()

Native modules can conform to the [RCTInvalidating](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTInvalidating.h) protocol on iOS by implementing the `invalidate()` method. This method [can be invoked](https://github.com/facebook/react-native/blob/0.62-stable/ReactCommon/turbomodule/core/platform/ios/RCTTurboModuleManager.mm#L456) when the native bridge is invalidated (ie: on devmode reload). Please use this mechanism as necessary to do the required cleanup for your native module.