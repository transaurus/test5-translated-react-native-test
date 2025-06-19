---
id: native-modules-ios
title: iOS Native Modules
---

import NativeDeprecated from '../the-new-architecture/\_markdown_native_deprecation.mdx'
import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<NativeDeprecated />

歡迎來到 iOS 原生模組開發。請先閱讀 [原生模組簡介](native-modules-intro) 了解什麼是原生模組。

## 建立日曆原生模組

In the following guide you will create a native module, `CalendarModule`, that will allow you to access Apple's calendar APIs from JavaScript. By the end you will be able to call `CalendarModule.createCalendarEvent('Dinner Party', 'My House');` from JavaScript, invoking a native method that creates a calendar event.

### 設定

首先，請在 Xcode 中開啟您的 React Native 應用程式內的 iOS 專案。您可以在以下路徑找到 React Native 應用程式中的 iOS 專案：

<figure>
  <img src="/docs/assets/native-modules-ios-open-project.png" width="500" alt="Image of opening up an iOS project within a React Native app inside of xCode." />
  <figcaption>Image of where you can find your iOS project</figcaption>
</figure>

我們建議使用 Xcode 來編寫原生程式碼。Xcode 專為 iOS 開發設計，能幫助您快速解決程式碼語法等小錯誤。

### 建立自訂原生模組檔案

The first step is to create our main custom native module header and implementation files. Create a new file called `RCTCalendarModule.h`

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

您可以為原生模組使用任何合適的名稱。由於您正在建立日曆原生模組，因此將類別命名為 `RCTCalendarModule`。由於 Objective-C 不像 Java 或 C++ 有語言層級的命名空間支援，慣例是在類別名稱前加上子字串。這可以是您應用程式名稱或基礎架構名稱的縮寫。在此範例中，RCT 代表 React。

如下所示，CalendarModule 類別實現了 `RCTBridgeModule` 協定。原生模組就是實現 `RCTBridgeModule` 協定的 Objective-C 類別。

接下來，讓我們開始實作原生模組。在 Xcode 中使用 cocoa touch class 在同一個資料夾中建立對應的實作檔 `RCTCalendarModule.m`，並包含以下內容：

```objectivec
// RCTCalendarModule.m
#import "RCTCalendarModule.h"

@implementation RCTCalendarModule

// To export a module named RCTCalendarModule
RCT_EXPORT_MODULE();

@end

```

### 模組名稱

目前，您的 `RCTCalendarModule.m` 原生模組僅包含 `RCT_EXPORT_MODULE` 巨集，該巨集會匯出並向 React Native 註冊原生模組類別。`RCT_EXPORT_MODULE` 巨集還接受一個可選參數，用於指定模組在 JavaScript 程式碼中的可存取名稱。

此參數不是字串字面量。在下面的範例中傳遞的是 `RCT_EXPORT_MODULE(CalendarModuleFoo)`，而不是 `RCT_EXPORT_MODULE("CalendarModuleFoo")`。

```objectivec
// To export a module named CalendarModuleFoo
RCT_EXPORT_MODULE(CalendarModuleFoo);
```

然後可以在 JS 中這樣存取原生模組：

```tsx
const {CalendarModuleFoo} = ReactNative.NativeModules;
```

如果您不指定名稱，JavaScript 模組名稱將與 Objective-C 類別名稱相同，但會移除任何 "RCT" 或 "RK" 前綴。

讓我們遵循下面的範例，不帶任何參數呼叫 `RCT_EXPORT_MODULE`。因此，模組將以 `CalendarModule` 的名稱暴露給 React Native，因為這是移除 RCT 後的 Objective-C 類別名稱。

```objectivec
// Without passing in a name this will export the native module name as the Objective-C class name with “RCT” removed
RCT_EXPORT_MODULE();
```

然後可以在 JS 中這樣存取原生模組：

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

在完善 `createCalendarEvent()` 方法的功能之前，先在方法中添加一個控制台日誌，以便確認它已從 React Native 應用程序的 JavaScript 端被調用。使用 React 提供的 `RCTLog` API。首先在文件頂部導入該標頭文件，然後添加日誌調用。

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

該方法的返回類型必須是對象類型（id），並且應該可序列化為 JSON。這意味著該方法只能返回 nil 或 JSON 值（例如 NSNumber、NSString、NSArray、NSDictionary）。

目前，我們不建議使用同步方法，因為同步調用方法可能會帶來嚴重的性能損失，並在原生模組中引入與線程相關的錯誤。此外，請注意，如果你選擇使用 `RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD`，你的應用程序將無法再使用 Google Chrome 調試器。這是因為同步方法要求 JS 虛擬機與應用程序共享內存。對於 Google Chrome 調試器，React Native 運行在 Google Chrome 的 JS 虛擬機中，並通過 WebSockets 與移動設備進行異步通信。

### 測試已構建的內容

至此，你已經在 iOS 中為原生模組設置了基本框架。現在可以通過訪問原生模組並在 JavaScript 中調用其導出的方法來進行測試。

在你的應用程序中找到一個地方，添加對原生模組 `createCalendarEvent()` 方法的調用。以下是一個組件示例 `NewModuleButton`，你可以在應用中添加它。你可以在 `NewModuleButton` 的 `onPress()` 函數中調用原生模組。

```tsx
import React from 'react';
import {Button} from 'react-native';

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

為了從 JavaScript 訪問你的原生模組，首先需要從 React Native 導入 `NativeModules`：

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

最後一步是重新構建 React Native 應用程序，以便讓最新的原生代碼（包含你的新原生模組！）生效。在 React Native 應用程序所在的命令行中，運行以下命令：

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

### 迭代開發時的構建

當你依照這些指南進行原生模組的迭代開發時，需要透過上述指令重新建置應用程式，才能讓JavaScript端取得最新的變更。這是因為你所編寫的程式碼位於應用的原生部分。雖然React Native的Metro打包工具能監控JavaScript變更並即時重建JS套件，但不會自動處理原生程式碼。因此若要測試最新的原生端修改，必須使用前述指令手動重建。

### 重點回顧✨

現在你應該能在JavaScript中呼叫原生模組的`createCalendarEvent()`方法。由於函式中使用了`RCTLog`，可透過[啟用應用除錯模式](https://reactnative.dev/docs/debugging#chrome-developer-tools)，在Chrome的JS控制台或行動端除錯工具Flipper中確認原生方法是否被觸發。每次呼叫原生模組方法時，都應看到`RCTLogInfo(@"Pretending to create an event %@ at %@", name, location);`的日誌訊息。

<figure>
  <img src="/docs/assets/native-modules-ios-logs.png" width="1000" alt="Image of logs." />
  <figcaption>Image of iOS logs in Flipper</figcaption>
</figure>

至此你已建立iOS原生模組，並從React Native應用的JavaScript端成功呼叫其方法。後續可進一步學習原生模組方法的參數類型設定，以及如何在模組中配置回調函式與Promise。

## 日曆原生模組的進階應用

### 優化模組導出方式

透過`NativeModules`提取原生模組的現行方式稍顯笨拙。

為避免模組使用者每次存取時重複此操作，可建立JavaScript封裝層。新建名為NativeCalendarModule.js的檔案，內容如下：

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

此JavaScript檔案同時也是添加前端功能的理想位置。例如若使用TypeScript等類型系統，可在此為原生模組添加類型註解。雖然React Native尚未支援原生與JS間的類型安全，但這些註解能確保所有JS程式碼具備類型安全，也為未來轉換至類型安全的原生模組預作準備。以下是為日曆模組添加類型安全的範例：

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

在其他JavaScript檔案中，可透過以下方式存取原生模組並呼叫其方法：

```tsx
import NativeCalendarModule from './NativeCalendarModule';
NativeCalendarModule.createCalendarEvent('foo', 'bar');
```

> 注意：此處假設導入`CalendarModule`的位置與`NativeCalendarModule.js`處於相同目錄層級，請根據實際情況調整相對路徑。

### 參數類型對應

當JavaScript呼叫原生模組方法時，React Native會將JS物件參數轉換為對應的Objective-C/Swift物件類型。例如若Objective-C原生模組方法接受NSNumber，在JS端需傳入數字類型，React Native會自動處理轉換。以下是原生模組方法支援的參數類型與JS對應關係清單：

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

> 以下類型目前雖被支援，但TurboModules將不再兼容，建議避免使用：
>
> - Function (failure) -> RCTResponseErrorBlock
> - Number -> NSInteger
> - Number -> CGFloat
> - Number -> float

iOS端還可使用`RCTConvert`類支援的任何參數類型（詳見[RCTConvert](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTConvert.h)）。RCTConvert輔助方法皆接受JSON值作為輸入，並將其映射至原生Objective-C類型或類別。

### 導出常數

A native module can export constants by overriding the native method `constantsToExport()`. Below `constantsToExport()` is overridden, and returns a Dictionary that contains a default event name property you can access in JavaScript like so:

```objectivec
- (NSDictionary *)constantsToExport
{
 return @{ @"DEFAULT_EVENT_NAME": @"New Event" };
}
```

接著可以透過在 JavaScript 中呼叫原生模組的 `getConstants()` 方法來存取這個常數：

```tsx
const {DEFAULT_EVENT_NAME} = CalendarModule.getConstants();
console.log(DEFAULT_EVENT_NAME);
```

Technically, it is possible to access constants exported in `constantsToExport()` directly off the `NativeModule` object. This will no longer be supported with TurboModules, so we encourage the community to switch to the above approach to avoid necessary migration down the line.

> 請注意，常數僅在初始化時匯出，因此如果在執行階段變更 `constantsToExport()` 的值，並不會影響 JavaScript 環境。

在 iOS 上，如果你覆寫了 `constantsToExport()`，則應同時實作 `+ requiresMainQueueSetup` 來告知 React Native 你的模組是否需要先於任何 JavaScript 程式碼執行前在主執行緒初始化。否則你會看到警告，提示未來你的模組可能會在背景執行緒初始化，除非你明確透過 `+ requiresMainQueueSetup:` 選擇退出。如果你的模組不需要存取 UIKit，則應對 `+ requiresMainQueueSetup` 回傳 NO。

### 回呼函式

原生模組還支援一種特殊的參數類型——回呼函式(callback)。回呼函式用於在非同步方法中將資料從 Objective-C 傳遞到 JavaScript，也可用於從原生端非同步執行 JavaScript。

For iOS, callbacks are implemented using the type `RCTResponseSenderBlock`. Below the callback parameter `myCallBack` is added to the `createCalendarEventMethod()`:

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title
                location:(NSString *)location
                myCallback:(RCTResponseSenderBlock)callback)

```

接著你可以在原生函式中呼叫這個回呼，並透過陣列傳遞任何想傳給 JavaScript 的結果。請注意，`RCTResponseSenderBlock` 只接受一個參數——要傳遞給 JavaScript 回呼的參數陣列。以下範例將回傳先前呼叫中建立的事件 ID。

> 需要特別強調的是，回呼函式不會在原生函式完成後立即被呼叫——請記住這種通訊是非同步的。

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title location:(NSString *)location callback: (RCTResponseSenderBlock)callback)
{
 NSInteger eventId = ...
 callback(@[@(eventId)]);

 RCTLogInfo(@"Pretending to create an event %@ at %@", title, location);
}

```

然後可以透過以下方式在 JavaScript 中存取這個方法：

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

原生模組應該只呼叫其回呼函式一次。不過，它可以儲存回呼並稍後再呼叫。這種模式常用於封裝需要委派(delegate)的 iOS API——參見 [`RCTAlertManager`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/CoreModules/RCTAlertManager.mm) 的範例。如果回呼從未被呼叫，會導致一些記憶體洩漏。

使用回呼函式時有兩種錯誤處理方式。第一種是遵循 Node 的慣例，將傳遞給回呼陣列的第一個參數視為錯誤物件。

```objectivec
RCT_EXPORT_METHOD(createCalendarEventCallback:(NSString *)title location:(NSString *)location callback: (RCTResponseSenderBlock)callback)
{
  NSNumber *eventId = [NSNumber numberWithInt:123];
  callback(@[[NSNull null], eventId]);
}
```

在 JavaScript 中，你可以檢查第一個參數來判斷是否有錯誤被傳遞：

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

另一種選擇是使用兩個獨立的回呼函式：onFailure 和 onSuccess。

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

然後在 JavaScript 中，你可以為錯誤和成功回應分別添加回呼：

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

原生模組也可以實現 Promise，這能簡化你的 JavaScript 程式碼，特別是在使用 ES2016 的 `async/await` 語法時。當原生模組方法的最後一個參數是 `RCTPromiseResolveBlock` 和 `RCTPromiseRejectBlock` 時，其對應的 JS 方法將回傳一個 JS Promise 物件。

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

JavaScript 代碼可以通過圍繞您的模組創建一個新的 `NativeEventEmitter` 實例來訂閱這些事件。

You will receive a warning if you expend resources unnecessarily by emitting an event while there are no listeners. To avoid this, and to optimize your module's workload (e.g. by unsubscribing from upstream notifications or pausing background tasks), you can override `startObserving` and `stopObserving` in your `RCTEventEmitter` subclass.

```objectivec
@implementation CalendarModule
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

除非原生模組提供自己的方法隊列，否則不應對其被調用的線程做出任何假設。目前，如果原生模組未提供方法隊列，React Native 會為其創建一個單獨的 GCD 隊列並在其中調用其方法。請注意，這是一個實現細節，可能會發生變化。如果您想為原生模組顯式提供方法隊列，請在原生模組中覆寫 `(dispatch_queue_t) methodQueue` 方法。例如，如果需要使用僅限主線程的 iOS API，應通過以下方式指定：

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

React Native 會自動創建並初始化任何已註冊的原生模組。但是，您可能希望創建並初始化自己的模組實例，例如為了注入依賴項。

您可以通過創建一個實現 `RCTBridgeDelegate` 協議的類，使用該委託作為參數初始化一個 `RCTBridge`，然後使用初始化後的橋接器初始化一個 `RCTRootView` 來實現這一點。

```objectivec
id<RCTBridgeDelegate> moduleInitialiser = [[classThatImplementsRCTBridgeDelegate alloc] init];

RCTBridge *bridge = [[RCTBridge alloc] initWithDelegate:moduleInitialiser launchOptions:nil];

RCTRootView *rootView = [[RCTRootView alloc]
                        initWithBridge:bridge
                            moduleName:kModuleName
                     initialProperties:nil];
```

### 導出 Swift

Swift 不支持宏，因此將原生模組及其方法暴露給 React Native 中的 JavaScript 需要更多的設置。但其工作原理基本相同。假設您有相同的 `CalendarModule`，但作為一個 Swift 類：

```swift
// CalendarModule.swift

@objc(CalendarModule)
class CalendarModule: NSObject {

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
// CalendarModuleBridge.m
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(CalendarModule, NSObject)

RCT_EXTERN_METHOD(addEvent:(NSString *)name location:(NSString *)location date:(nonnull NSNumber *)date)

@end
```

For those of you new to Swift and Objective-C, whenever you [mix the two languages in an iOS project](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html), you will also need an additional bridging file, known as a bridging header, to expose the Objective-C files to Swift. Xcode will offer to create this header file for you if you add your Swift file to your app through the Xcode `File>New File` menu option. You will need to import `RCTBridgeModule.h` in this header file.

```objectivec
// CalendarModule-Bridging-Header.h
#import <React/RCTBridgeModule.h>
```

您亦可使用 `RCT_EXTERN_REMAP_MODULE` 和 `_RCT_EXTERN_REMAP_METHOD` 來調整導出模組或方法在 JavaScript 中的名稱。詳情請參閱 [`RCTBridgeModule`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTBridgeModule.h)。

> 開發第三方模組時的重要注意事項：僅 Xcode 9 及以上版本支援含 Swift 的靜態函式庫。若模組中的 iOS 靜態函式庫使用 Swift，主應用專案必須包含 Swift 程式碼及橋接標頭檔才能成功建構 Xcode 專案。若應用專案不含任何 Swift 程式碼，可暫時加入一個空的 .swift 檔案與空白橋接標頭檔作為解決方案。

### 保留方法名稱

#### invalidate()

Native modules can conform to the [RCTInvalidating](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTInvalidating.h) protocol on iOS by implementing the `invalidate()` method. This method [can be invoked](https://github.com/facebook/react-native/blob/0.62-stable/ReactCommon/turbomodule/core/platform/ios/RCTTurboModuleManager.mm#L456) when the native bridge is invalidated (ie: on devmode reload). Please use this mechanism as necessary to do the required cleanup for your native module.