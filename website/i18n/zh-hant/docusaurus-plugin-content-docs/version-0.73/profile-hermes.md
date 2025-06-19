---
id: profile-hermes
title: Profiling with Hermes
---

您可以使用 [Hermes](https://github.com/facebook/hermes) 來視覺化 React Native 應用程式中 JavaScript 的效能表現。Hermes 是一個輕量級的 JavaScript 引擎，專為運行 React Native 而優化（您可以在[這裡閱讀更多關於與 React Native 搭配使用的資訊](hermes)）。Hermes 有助於提升應用程式效能，同時也提供了分析其所運行 JavaScript 效能的方法。

In this section, you will learn how to profile your React Native app running on Hermes and how to visualize the profile using [the Performance tab on Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference)

:::caution
在開始之前，請確保您已[在應用程式中啟用 Hermes](hermes)！
:::

請按照以下指示開始進行效能分析：

1. [記錄 Hermes 取樣分析檔](profile-hermes.md#record-a-hermes-sampling-profile)
2. [從 CLI 執行指令](profile-hermes.md#execute-command-from-cli)
3. [在 Chrome DevTools 中開啟下載的分析檔](profile-hermes.md#open-the-downloaded-profile-on-chrome-devtools)

## 記錄 Hermes 取樣分析檔

要從開發者選單記錄取樣分析器：

1. Navigate to your running Metro server terminal.
2. Press `d` to open the **Dev Menu.**
3. Select **Enable Sampling Profiler.**
4. Execute your JavaScript by in your app (press buttons, etc.)
5. Open the **Dev Menu** by pressing `d` again.
6. Select **Disable Sampling Profiler** to stop recording and save the sampling profiler.

A toast will show the location where the sampling profiler has been saved, usually in `/data/user/0/com.appName/cache/*.cpuprofile`

<img src="/docs/assets/HermesProfileSaved.png" height="465" width="250" alt="Toast Notification of Profile saving" />

## 從 CLI 執行指令

您可以使用 [React Native CLI](https://github.com/react-native-community/cli) 將 Hermes 追蹤分析檔轉換為 Chrome 追蹤分析檔，並透過以下指令將其拉取至本地機器：

```sh
npx react-native profile-hermes [destinationDir]
```

### 啟用原始碼映射

:::info
您可以在[除錯發行版本](debugging-release-builds.md)頁面閱讀關於原始碼映射的更多資訊。
:::

### 常見錯誤

#### `adb: no devices/emulators found` 或 `adb: device offline`

- **原因** CLI 無法存取您用來運行應用程式的裝置或模擬器（透過 adb）。
- **解決方法** 確保您的 Android 裝置/模擬器已連接並正在運行。此指令僅在能夠存取 adb 時才會生效。

#### `There is no file in the cache/ directory`

- **Why this happens** The CLI cannot find any **.cpuprofile** file in your app's **cache/** directory. You might have forgotten to record a profile from the device.
- **How to fix** Follow the [instructions](profile-hermes.md#record-a-hermes-sampling-profile) to enable/disable profiler from device.

#### `Error: your_profile_name.cpuprofile is an empty file`

- **原因** 分析檔為空，可能是因為 Hermes 未正確運行。
- **解決方法** 確保您的應用程式運行於最新版本的 Hermes 上。

## 在 Chrome DevTools 中開啟下載的分析檔

要在 Chrome DevTools 中開啟分析檔：

1. 開啟 Chrome DevTools。
2. 選擇 **Performance** 分頁。
3. 右鍵點擊並選擇 **載入分析檔...**

<img src="/docs/assets/openChromeProfile.png" alt="Loading a performance profile on Chrome DevTools" />

## Hermes 分析轉換器是如何運作的？

The Hermes Sample Profile is of the `JSON object format`, while the format that Google's DevTools supports is `JSON Array Format`. (More information about the formats can be found on the [Trace Event Format Document](https://docs.google.com/document/d/1CvAClvFfyA5R-PhYUmn5OOQtYMH4h6I0nSsKchNAySU/preview))

```ts
export interface HermesCPUProfile {
  traceEvents: SharedEventProperties[];
  samples: HermesSample[];
  stackFrames: {[key in string]: HermesStackFrame};
}
```

Hermes 效能分析檔案的大部分資訊都編碼在 `samples` 和 `stackFrames` 屬性中。每個取樣點都是特定時間戳記下的函式呼叫堆疊快照，每個取樣點都有一個 `sf` 屬性，對應到一個函式呼叫。

```ts
export interface HermesSample {
  cpu: string;
  name: string;
  ts: string;
  pid: number;
  tid: string;
  weight: string;
  /**
   * Will refer to an element in the stackFrames object of the Hermes Profile
   */
  sf: number;
  stackFrameData?: HermesStackFrame;
}
```

關於函式呼叫的資訊可在 `stackFrames` 中找到，其中包含鍵值對，鍵是 `sf` 編號，對應的物件則提供該函式的所有相關資訊，包括其父函式的 `sf` 編號。這種父子關係可向上追溯，以找出特定時間戳記下所有執行中函式的資訊。

```ts
export interface HermesStackFrame {
  line: string;
  column: string;
  funcLine: string;
  funcColumn: string;
  name: string;
  category: string;
  /**
   * A parent function may or may not exist
   */
  parent?: number;
}
```

此時，您應先定義幾個術語：

1. Nodes: The objects corresponding to `sf` numbers in `stackFrames`
2. Active Nodes: The nodes which are currently running at a particular timestamp. A node is classified as running if its `sf` number is in the function call stack. This call stack can be obtained from the `sf` number of the sample and tracing upwards till parent `sf`s are available

接著可結合 `samples` 和 `stackFrames` 來產生對應時間戳記下的所有開始與結束事件，其中：

1. 開始節點/事件（Start Nodes/Events）：在前一個取樣點的函式呼叫堆疊中不存在，但在當前取樣點中出現的節點
2. 結束節點/事件（End Nodes/Events）：在前一個取樣點的函式呼叫堆疊中存在，但在當前取樣點中消失的節點

<img src="/docs/assets/CallStackDemo.jpg" height="800" width="600" alt="CallStack Terms Explained" />

You can now construct a `flamechart` of function calls as you have all the function information including its start and end timestamps.

The `hermes-profile-transformer` can convert any profile generated using Hermes into a format that can be directly displayed in Chrome DevTools. More information about this can be found on [ `@react-native-community/hermes-profile-transformer` ](https://github.com/react-native-community/hermes-profile-transformer)