# 附錄

## I. 術語解釋

- **Spec** - 用於描述 Turbo Native 模組或 Fabric Native 元件的 API 的 TypeScript 或 Flow 代碼。由 **Codegen** 用於生成樣板代碼。

- **Native Modules (原生模組)** - 不包含用戶界面(UI)的原生函式庫。例如持久存儲、通知、網絡事件等。這些可通過函數和對象在您的 JavaScript 應用代碼中訪問。
- **Native Component (原生元件)** - 可通過 React 元件在應用 JavaScript 代碼中使用的原生平台視圖。

- **Legacy Native Components (舊版原生元件)** - 運行在舊版 React Native 架構上的元件。
- **Legacy Native Modules (舊版原生模組)** - 運行在舊版 React Native 架構上的模組。

## II. Codegen 類型對照

您可參考以下表格了解支援的類型及其在各平台的對應關係：

| Flow                                                                       | TypeScript                                          | Flow Nullable Support                                   | TypeScript Nullable Support                          | Android (Java)                       | iOS (ObjC)                                                     |
| -------------------------------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------- | ---------------------------------------------------- | ------------------------------------ | -------------------------------------------------------------- |
| `string`                                                                   | `string`                                            | `?string`                                               | <code>string &#124; null</code>                      | `string`                             | `NSString`                                                     |
| `boolean`                                                                  | `boolean`                                           | `?boolean`                                              | <code>boolean &#124; null</code>                     | `Boolean`                            | `NSNumber`                                                     |
| Object Literal<br /><code>&#123;&#124; foo: string, ...&#124;&#125;</code> | <code>&#123; foo: string, ...&#125; as const</code> | <code>?&#123;&#124; foo: string, ...&#124;&#125;</code> | <code>?&#123; foo: string, ...&#125; as const</code> | \-                                   | \-                                                             |
| Object [[1](#notes)]                                                       | Object [[1](#notes)]                                | `?Object`                                               | <code>Object &#124; null</code>                      | `ReadableMap`                        | `@` (untyped dictionary)                                       |
| <code>Array&lt;T&gt;</code>                                                | <code>Array&lt;T&gt;</code>                         | <code>?Array&lt;T&gt;</code>                            | <code>Array&lt;T&gt; &#124; null</code>              | `ReadableArray`                      | `NSArray` (or `RCTConvertVecToArray` when used inside objects) |
| `Function`                                                                 | `Function`                                          | `?Function`                                             | <code>Function &#124; null</code>                    | \-                                   | \-                                                             |
| <code>Promise&lt;T&gt;</code>                                              | <code>Promise&lt;T&gt;</code>                       | <code>?Promise&lt;T&gt;</code>                          | <code>Promise&lt;T&gt; &#124; null</code>            | `com.facebook.react.bridge.Promise`  | `RCTPromiseResolve` and `RCTPromiseRejectBlock`                |
| Type Unions<br /><code>'SUCCESS'&#124;'FAIL'</code>                        | Type Unions<br /><code>'SUCCESS'&#124;'FAIL'</code> | Only as callbacks                                       |                                                      | \-                                   | \-                                                             |
| Callbacks<br />`() =>`                                                     | Callbacks<br />`() =>`                              | Yes                                                     |                                                      | `com.facebook.react.bridge.Callback` | `RCTResponseSenderBlock`                                       |
| `number`                                                                   | `number`                                            | No                                                      |                                                      | `double`                             | `NSNumber`                                                     |

### 注意事項：

<b>[1]</b> 我們強烈建議使用物件字面量(Object literals)而非物件(Objects)。

:::info
您也可能會發現參考 React Native 核心模組的 JavaScript 規範很有幫助。這些規範位於 React Native 代碼庫中的 `Libraries/` 目錄內。
:::