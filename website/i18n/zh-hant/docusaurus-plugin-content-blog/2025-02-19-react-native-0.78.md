---
title: 'React Native 0.78 - React 19 and more'
authors: [vonovak, shubham, fabriziocucci, cipolleschi]
tags: [engineering]
date: 2025-02-19
---

# React Native 0.78 - React 19 與更多新功能

我們很高興今天發布 React Native 0.78！

此版本整合了 React 19 至 React Native，並帶來其他重要功能，例如原生支援 Android 向量圖形繪製資源，以及更完善的 iOS 棕地整合方案。

### 重點更新

- [React 19](/blog/2025/02/19/react-native-0.78#react-19)
- [邁向更輕量快速的發布週期](/blog/2025/02/19/react-native-0.78#towards-smaller-and-faster-releases)
- [Metro 中的 JavaScript 日誌記錄功能選擇加入機制](/blog/2025/02/19/react-native-0.78#opt-in-for-javascript-logs-in-metro)
- [新增 Android XML 繪製資源支援](/blog/2025/02/19/react-native-0.78#added-support-for-android-xml-drawables)
- [iOS 的 ReactNativeFactory 機制](/blog/2025/02/19/react-native-0.78#reactnativefactory-on-ios)

<!--truncate-->

## 重點功能詳述

### React 19

React 19 現已正式支援 React Native！

升級至 React 19 需進行應用程式調整，因我們移除了部分 API（如 `propTypes`），您需根據新版 React 特性進行相容性改動。

請遵循我們的[逐步升級指南](https://react.dev/blog/2024/04/25/react-19-upgrade-guide)將應用遷移至 React 19。

完成遷移後，您將能使用 React 的所有新特性，包括（但不限於）：

- **[Actions](https://react.dev/blog/2024/12/05/react-19#actions):** these are functions that use async transitions. Async transitions automatically manage submitting data for you: they handle pending states, optimistic updates, error handling and more.
- **[useActionState](https://react.dev/reference/react/useActionState):** a utility hook built on top of Actions. It takes a function and returns a wrapped Action to call. When the action is called, it will return the last result of the Action and its `pending` state.
- **[useOptimistic](https://react.dev/reference/react/useOptimistic):** a new hook that simplifies showing the final state of an update optimistically while the async request is underway. If the request errors, React will switch back to the previous value automatically.
- **[`use`](https://react.dev/reference/react/use):** this is a new API that allows access to resources during render. You can now read a promise or a context with `use` and React will Suspend until they resolve.
- **[`ref` as `props`](https://react.dev/blog/2024/12/05/react-19#ref-as-a-prop):** you can now pass `ref`as a `prop` like you do with any other prop. Function components will no longer need `forwardRef` and you can migrate your components now.
- And many others

完整新功能清單請參閱 [React 19 發布部落格](https://react.dev/blog/2024/12/05/react-19)。

#### React 編譯器

React 編譯器是建置階段工具，透過自動化記憶機制優化 React 應用。雖然開發者能手動使用 `useMemo`、`useCallback` 和 `React.memo` 等 API 避免不必要的重新計算，但也可能遺漏或誤用這些優化。React 編譯器透過解析 JavaScript 與 [React 規則](https://react.dev/reference/rules)，自動對元件與 Hook 中的值或值群組實施記憶化，解決此問題。

在此版本中，我們簡化了在 React Native 應用中啟用 React Compiler 的流程。[在先前版本](https://react.dev/learn/react-compiler#using-react-compiler-with-react-17-or-18)中，您需要安裝兩個套件：編譯器及其運行時。安裝這些套件後，還需配置 Babel 插件以透過 Metro 啟用 React Compiler。

現在，您只需安裝編譯器本身並配置 Babel 插件即可。若要了解如何啟用，可遵循我們的逐步[指南](https://react.dev/learn/react-compiler#usage-with-babel)。

若要驗證編譯器是否運作，可開啟 React Native DevTools：您應能在元件檢查器中看到被記憶化的元件標有 `Memo ✨` 標籤。

若想深入了解 React Compiler，以下資源相當實用：

- [React Compiler](https://react.dev/learn/react-compiler) 文件
- [React Compiler 深度解析](https://www.youtube.com/watch?v=uA_PVyZP7AI)

### 邁向更小更快的版本發布

我們正在更新發布流程，以便在 2025 年更頻繁地推出穩定的 React Native 版本。

這將使您更容易更新 React Native 版本，因為我們將減少引入的破壞性變更。更快的發布也意味著我們內部修復的錯誤能更早提供給您，讓您能受益於我們在 React Native 中開發的最新功能。

我們相信這種新模式將使 React Native 生態系統中的每位開發者受益，因為更少的破壞性變更意味著更穩定的框架，讓大家都能依賴。

### 選擇性啟用 Metro 中的 JavaScript 日誌

我們新增了一個選項，可透過 Metro 開發伺服器恢復 JavaScript 日誌串流，[此功能在 0.77 版中被移除](/blog/2025/01/21/version-0.77#removal-of-consolelog-streaming-in-metro)（針對 Community CLI 使用者）。這是基於使用者回饋，以及我們對當前替代方案的評估。

若要啟用，請使用新的 `--client-logs` 標記。為方便起見，也可透過 npm 腳本設定別名。

```sh
npx @react-native-community/cli start --client-logs
```

Metro 中的日誌串流功能未來仍會移除，且預設保持關閉。然而，我們打算給予開發者更長的遷移期以適應此變更。

此更新也將在即將發布的 0.77.1 次要版本中提供。

### 新增對 Android XML 繪圖資源的支援

在 React Native 0.78 中，我們推出了一種新方法，可在 Android 上以 [XML 資源](https://developer.android.com/guide/topics/resources/drawable-resource)形式載入圖標、插圖和其他圖形元素。這意味著您可以使用[向量繪圖資源](https://developer.android.com/develop/ui/views/graphics/vector-drawable-resources)來顯示任意縮放皆不失真的向量圖像，或使用[形狀繪圖資源](https://developer.android.com/guide/topics/resources/drawable-resource#Shape)來繪製更基本的裝飾元素。這些功能均由您熟悉的 `Image` 元件支援。要立即使用此功能，可像引用其他[靜態資源](/docs/next/images#static-image-resources)一樣，在 `source` 屬性中引用 XML 資源。此外，使用 XML 資源而非點陣圖還能幫助您減小應用體積，並在不同 DPI 的螢幕上實現更好的渲染效果。

```js
// via require
<Image
  source={require('./img/my_icon.xml')}
  style={{width: 40, height: 40}}
/>;

// or via import
import MyIcon from './img/my_icon.xml';
<Image source={MyIcon} style={{width: 40, height: 40}} />;
```

#### 效能與品質

[與其他所有圖片類型一樣](/docs/next/images#off-thread-decoding)，Android 的 XML 資源會在非主線程上載入和解析，因此不會丟失任何幀數。這意味著資源不保證能立即顯示，但在資源載入時也不會阻擋使用者輸入。當您需要同時渲染多個圖標時，非主線程解析尤其重要。內部應用在使用 Android 的向量繪圖資源時，已實現了顯著的效能提升。

使用向量可繪製資源（vector drawables）這類資源類型，是完美呈現不失真圖像的方式，且能縮減APK檔案大小，因為您無需為每種螢幕密度包含對應的圖片格式。此外，向量可繪製資源一旦載入後會從記憶體複製，因此若您多次渲染相同圖示，它們將能同時顯示。

#### 權衡取捨

需注意可繪製XML資源並非完美，使用時存在以下限制：

- They must be referenced at build time of your Android application. These resources are passed into a build step with the [Android Asset Packaging Tool](https://developer.android.com/tools/aapt2) (AAPT) to convert raw XML into binary XML. Android does not support loading raw XML files, [this is a known limitation](https://issuetracker.google.com/issues/62435069).
- They cannot be loaded over the network by Metro. If you change the directory or name of an XML resource, you will need to rebuild your Android application each time.
- They have no dimensions. By default, they will display with a 0x0 size and you need to provide a width and height for them to show up.
- They are not fully customizable at runtime; you can only control dimensions or the overall tint color but you can’t customize individual element attributes _inside_ the resource like stroke widths, border radius, or colors. These types of customizations require different variants of your XML resource.

:::info
Android的向量可繪製資源並非`react-native-svg`等函式庫的1:1替代方案。它們專為Android設計，無法用於iOS。若需跨平台使用相同SVG，仍需使用`react-native-svg`。向量可繪製資源僅以犧牲自訂性換取效能優勢。
:::

### iOS上的ReactNativeFactory

React Native 0.78強化了iOS平台的整合度。

此版本新增名為`RCTReactNativeFactory`的類別，讓您無需透過AppDelegate即可建立React Native實例。例如，您現在能在ViewController中建立新版本的React Native，大幅簡化與Brownfield應用的整合流程。

假設您想在應用程式的視圖控制器中顯示React Native視圖。從React Native 0.78開始，按照[本指南](/docs/next/integration-with-existing-apps?language=apple#1-set-up-directory-structure)安裝所有相依項後，只需加入以下程式碼：

```diff

+import React
+import React_RCTAppDelegate

public class ViewController {

+  var reactNativeFactory: RCTReactNativeFactory?
+  var reactNativeDelegate: ReactNativeDelegate?

  public func viewdidLoad() {
    super.viewDidLoad()
    // …
+ reactNativeDelegate = ReactNativeDelegate()
+ reactNativeFactory = RCTReactNativeFactory(delegate: reactNativeDelegate!)
+ view = reactNativeFactory.rootViewFactory.view(withModuleName: "<your module name>")
  }

}

+class ReactNativeDelegate: RCTDefaultReactNativeFactoryDelegate {

+  override func sourceURL(for bridge: RCTBridge) -> URL? {
+    self.bundleURL()
+  }
+
+  override func bundleURL() -> URL? {
+    #if DEBUG
+    RCTBundleURLProvider.sharedSettings().jsBundleURL(forBundleRoot: "index")
+    #else
+    Bundle.main.url(forResource: "main", withExtension: "jsbundle")
+    #endif
+  }
+}

```

當您導航至該視圖控制器時，React Native將立即載入。

此程式碼會建立`RCTReactNativeFactory`、指派委派對象，並要求其為React Native視圖建立`rootView`。

下方定義的委派會覆寫`sourceURL`與`bundleURL`屬性，告知React Native從何處載入JS套件至視圖中。

## 其他重大變更

### 通用

- React Native DevTools
  - 移除FuseboxClient CDP領域
- Codegen
  - 分離元件陣列類型與指令陣列類型

### Android

- 可空性變更：將`RootView`遷移至Kotlin導致參數類型從可空變更為不可空。
- 以下類別已從公開改為內部或移除，無法再存取：
  - `com.facebook.react.bridge.GuardedResultAsyncTask`
  - `com.facebook.react.uimanager.ComponentNameResolver`
  - `com.facebook.react.uimanager.FabricViewStateManager`
  - `com.facebook.react.views.text.frescosupport.FrescoBasedReactTextInlineImageViewManager`

### iOS

- 將圖片載入事件的尺寸資訊從邏輯尺寸改為像素（僅影響舊架構）

## 致謝

React Native 0.78 包含了來自 87 位貢獻者的超過 509 次提交。感謝大家的辛勤工作！

感謝所有在本版本發布文章中參與功能文檔撰寫的額外作者：

- [Dream11](https://github.com/dream-sports-labs) 團隊對 React Native 中 React 19 功能的全面測試
- [Nicola Corti](https://github.com/cortinico) 為「更快的發布」所做的貢獻
- [Alex Hunt](https://github.com/huntie) 為 Metro 日誌選擇加入功能的工作
- [Peter Abbondanzo](https://github.com/Abbondanzo) 為 Android XML Drawable 支援的工作
- [Oskar Kwaśniewski](https://github.com/okwasniewski) 為 ReactNativeFactory 的工作

## 升級至 0.78

請使用 [React Native Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) 查看現有專案在 React Native 版本之間的程式碼變更，以及升級文檔。

要建立一個新專案：

```
npx @react-native-community/cli@latest init MyProject --version latest
```

如果您使用 Expo，[React Native 0.78 將在 Expo SDK 的 canary 版本中獲得支援](https://expo.dev/changelog/react-native-78)。

:::info
0.78 現在是 React Native 的最新穩定版本，而 0.75.x 將不再獲得支援。更多資訊請參閱 [React Native 的支援政策](https://github.com/reactwg/react-native-releases/blob/main/docs/support.md)。我們計劃在不久的將來發布 0.75 的最終終止支援更新。
:::