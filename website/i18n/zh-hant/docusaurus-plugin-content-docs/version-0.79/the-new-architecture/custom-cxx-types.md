import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

# 進階：自訂 C++ 型別

:::note
本指南假設您已熟悉[**純 C++ Turbo 原生模組**](pure-cxx-modules.md)指南。本文將在此基礎上進一步延伸。
:::

C++ Turbo Native Modules support [bridging functionality](https://github.com/facebook/react-native/tree/main/packages/react-native/ReactCommon/react/bridging) for most `std::` standard types. You can use most of those types in your modules without any additional code required.

若需在應用程式或函式庫中新增自訂型別支援，您必須提供對應的 `bridging` 標頭檔案。

## 新增自訂型別：Int64

由於 JavaScript 不支援大於 2^53 的數字，C++ Turbo 原生模組目前尚未支援 `int64_t` 數值型別。我們可以在 JS 中使用 `string` 型別來表示大數，並自動轉換為 C++ 的 `int64_t`。

### 1. 建立橋接標頭檔案

The first step to support a new custom type is to define the bridging header that takes care of converting the type **from** the JS representation to the C++ representation, and from the C++ representation **to** the JS one.

1. In the `shared` folder, add a new file called `Int64.h`
2. Add the following code to that file:

```cpp title="Int64.h"
#pragma once

#include <react/bridging/Bridging.h>

namespace facebook::react {

template <>
struct Bridging<int64_t> {
  // Converts from the JS representation to the C++ representation
  static int64_t fromJs(jsi::Runtime &rt, const jsi::String &value) {
    try {
      size_t pos;
      auto str = value.utf8(rt);
      auto num = std::stoll(str, &pos);
      if (pos != str.size()) {
        throw std::invalid_argument("Invalid number"); // don't support alphanumeric strings
      }
      return num;
    } catch (const std::logic_error &e) {
      throw jsi::JSError(rt, e.what());
    }
  }

  // Converts from the C++ representation to the JS representation
  static jsi::String toJs(jsi::Runtime &rt, int64_t value) {
    return bridging::toJs(rt, std::to_string(value));
  }
};

}
```

自訂橋接標頭的關鍵組件包括：

- Explicit specialization of the `Bridging` struct for your custom type. In this case, the template specify the `int64_t` type.
- A `fromJs` function to convert from the JS representation to the C++ representation
- A `toJs` function to convert from the C++ representation to the JS representation

:::note
在 iOS 平台上，請記得將 `Int64.h` 檔案加入 Xcode 專案。
:::

### 2. 修改 JS 規格

接著可修改 JS 規格，新增使用此型別的方法。與往常一樣，我們可以使用 Flow 或 TypeScript 來編寫規格。

1. 開啟 `specs/NativeSampleTurbomodule`
2. 按以下方式修改規格：

<Tabs groupId="custom-int64" queryString defaultValue={constants.defaultJavaScriptSpecLanguages} values={constants.javaScriptSpecLanguages}>
<TabItem value="typescript">

```diff title="NativeSampleModule.ts"
import {TurboModule, TurboModuleRegistry} from 'react-native';

export interface Spec extends TurboModule {
  readonly reverseString: (input: string) => string;
+  readonly cubicRoot: (input: string) => number;
}

export default TurboModuleRegistry.getEnforcing<Spec>(
  'NativeSampleModule',
);
```

</TabItem>
<TabItem value="flow">

```diff title="NativeSampleModule.js"
// @flow
import type {TurboModule} from 'react-native';
import { TurboModuleRegistry } from "react-native";

export interface Spec extends TurboModule {
  +reverseString: (input: string) => string;
+  +cubicRoot: (input: string) => number;
}

export default (TurboModuleRegistry.getEnforcing<Spec>(
  "NativeSampleModule"
): Spec);
```

</TabItem>
</Tabs>

此檔案中我們定義了需要在 C++ 中實現的函式。

### 3. 實作原生程式碼

現在需要實作在 JS 規格中宣告的函式。

1. 開啟 `specs/NativeSampleModule.h` 檔案並進行以下修改：

```diff title="NativeSampleModule.h"
#pragma once

#include <AppSpecsJSI.h>
#include <memory>
#include <string>

+ #include "Int64.h"

namespace facebook::react {

class NativeSampleModule : public NativeSampleModuleCxxSpec<NativeSampleModule> {
public:
  NativeSampleModule(std::shared_ptr<CallInvoker> jsInvoker);

  std::string reverseString(jsi::Runtime& rt, std::string input);
+ int32_t cubicRoot(jsi::Runtime& rt, int64_t input);
};

} // namespace facebook::react

```

2. 開啟 `specs/NativeSampleModule.cpp` 檔案並實作新函式：

```diff title="NativeSampleModule.cpp"
#include "NativeSampleModule.h"
+ #include <cmath>

namespace facebook::react {

NativeSampleModule::NativeSampleModule(std::shared_ptr<CallInvoker> jsInvoker)
    : NativeSampleModuleCxxSpec(std::move(jsInvoker)) {}

std::string NativeSampleModule::reverseString(jsi::Runtime& rt, std::string input) {
  return std::string(input.rbegin(), input.rend());
}

+int32_t NativeSampleModule::cubicRoot(jsi::Runtime& rt, int64_t input) {
+    return std::cbrt(input);
+}

} // namespace facebook::react
```

The implementation imports the `<cmath>` C++ library to perform mathematical operations, then it implements the `cubicRoot` function using the `cbrt` primitive from the `<cmath>` module.

### 4. 在應用程式中測試程式碼

現在可以在應用程式中測試程式碼。

首先需更新 `App.tsx` 檔案以使用 TurboModule 的新方法，接著可建置 Android 和 iOS 應用程式。

1. 開啟 `App.tsx` 程式碼並進行以下修改：

```diff title="App.tsx"
// ...
+ const [cubicSource, setCubicSource] = React.useState('')
+ const [cubicRoot, setCubicRoot] = React.useState(0)
  return (
    <SafeAreaView style={styles.container}>
      <View>
        <Text style={styles.title}>
          Welcome to C++ Turbo Native Module Example
        </Text>
        <Text>Write down here the text you want to revert</Text>
        <TextInput
          style={styles.textInput}
          placeholder="Write your text here"
          onChangeText={setValue}
          value={value}
        />
        <Button title="Reverse" onPress={onPress} />
        <Text>Reversed text: {reversedValue}</Text>
+        <Text>For which number do you want to compute the Cubic Root?</Text>
+        <TextInput
+          style={styles.textInput}
+          placeholder="Write your text here"
+          onChangeText={setCubicSource}
+          value={cubicSource}
+        />
+        <Button title="Get Cubic Root" onPress={() => setCubicRoot(SampleTurboModule.cubicRoot(cubicSource))} />
+        <Text>The cubic root is: {cubicRoot}</Text>
      </View>
    </SafeAreaView>
  );
}
//...
```

2. To test the app on Android, run `yarn android` from the root folder of your project.
3. To test the app on iOS, run `yarn ios` from the root folder of your project.

## 新增結構化自訂型別：Address

上述方法可通用於任何類型。對於結構化類型，React Native 提供了一些輔助函數，能更簡便地實現 JS 與 C++ 之間的橋接。

假設我們需要橋接一個自定義的 `Address` 類型，其屬性如下：

```ts
interface Address {
  street: string;
  num: number;
  isInUS: boolean;
}
```

### 1. 在規格中定義類型

首先在 JS 規格中定義新的自定義類型，讓 Codegen 能自動生成所有支援代碼，這樣就不需手動編寫。

1. 打開 `specs/NativeSampleModule` 文件並添加以下修改。

<Tabs groupId="custom-int64" queryString defaultValue={constants.defaultJavaScriptSpecLanguages} values={constants.javaScriptSpecLanguages}>
<TabItem value="typescript">

```diff title="NativeSampleModule (Add Address type and validateAddress function)"
import {TurboModule, TurboModuleRegistry} from 'react-native';

+export type Address = {
+  street: string,
+  num: number,
+  isInUS: boolean,
+};

export interface Spec extends TurboModule {
  readonly reverseString: (input: string) => string;
+ readonly validateAddress: (input: Address) => boolean;
}

export default TurboModuleRegistry.getEnforcing<Spec>(
  'NativeSampleModule',
);
```

</TabItem>
<TabItem value="flow">

```diff title="NativeSampleModule (Add Address type and validateAddress function)"

// @flow
import type {TurboModule} from 'react-native';
import { TurboModuleRegistry } from "react-native";

+export type Address = {
+  street: string,
+  num: number,
+  isInUS: boolean,
+};


export interface Spec extends TurboModule {
  +reverseString: (input: string) => string;
+ +validateAddress: (input: Address) => boolean;
}

export default (TurboModuleRegistry.getEnforcing<Spec>(
  "NativeSampleModule"
): Spec);
```

</TabItem>
</Tabs>

這段代碼定義了新的 `Address` 類型，並為 Turbo Native Module 新增了 `validateAddress` 函數。注意 `validateFunction` 需要一個 `Address` 物件作為參數。

也可以定義返回自定義類型的函數。

### 2. 定義橋接代碼

根據規格中定義的 `Address` 類型，Codegen 會生成兩個輔助類型：`NativeSampleModuleAddress` 和 `NativeSampleModuleAddressBridging`。

The first type is the definition of the `Address`. The second type contains all the infrastructure to bridge the custom type from JS to C++ and vice versa. The only extra step we need to add is to define the `Bridging` structure that extends the `NativeSampleModuleAddressBridging` type.

1. 打開 `shared/NativeSampleModule.h` 文件
2. 在文件中添加以下代碼：

```diff title="NativeSampleModule.h (Bridging the Address type)"
#include "Int64.h"
#include <memory>
#include <string>

namespace facebook::react {
+  using Address = NativeSampleModuleAddress<std::string, int32_t, bool>;

+  template <>
+  struct Bridging<Address>
+      : NativeSampleModuleAddressBridging<Address> {};
  // ...
}
```

This code defines an `Address` typealias for the generic type `NativeSampleModuleAddress`. **The order of the generics matters**: the first template argument refers to the first data type of the struct, the second refers to the second, and so forth.

Then, the code adds the `Bridging` specialization for the new `Address` type, by extending `NativeSampleModuleAddressBridging` that is generated by Codegen.

:::note
生成這些類型時遵循以下命名慣例：

- 名稱的第一部分始終是模組類型。本例中為 `NativeSampleModule`。
- 名稱的第二部分始終是規格中定義的 JS 類型名稱。本例中為 `Address`。
  :::

### 3. 實現原生代碼

現在需要在 C++ 中實現 `validateAddress` 函數。首先在 `.h` 文件中添加函數聲明，然後在 `.cpp` 文件中實現。

1. 打開 `shared/NativeSampleModule.h` 文件並添加函數定義

```diff title="NativeSampleModule.h (validateAddress function prototype)"
  std::string reverseString(jsi::Runtime& rt, std::string input);

+  bool validateAddress(jsi::Runtime &rt, jsi::Object input);
};

} // namespace facebook::react
```

2. 打開 `shared/NativeSampleModule.cpp` 文件並添加函數實現

```c++ title="NativeSampleModule.cpp (validateAddress implementation)"
bool NativeSampleModule::validateAddress(jsi::Runtime &rt, jsi::Object input) {
  std::string street = input.getProperty(rt, "street").asString(rt).utf8(rt);
  int32_t number = input.getProperty(rt, "num").asNumber();

  return !street.empty() && number > 0;
}
```

在實現中，代表 `Address` 的物件是一個 `jsi::Object`。要從該物件提取值，需使用 `JSI` 提供的訪問器：

- `getProperty()` 按名稱從物件獲取屬性。
- `asString()` 將屬性轉換為 `jsi::String`。
- `utf8()` 將 `jsi::String` 轉換為 `std::string`。
- `asNumber()` 將屬性轉換為 `double`。

手動解析物件後，即可實現所需邏輯。

:::note
If you want to learn more about `JSI` and how it works, have a look at this [great talk](https://youtu.be/oLmGInjKU2U?feature=shared) from App.JS 2024
:::

### 4. 在應用中測試代碼

要在應用程式中測試這段程式碼，我們需要修改 `App.tsx` 檔案。

1. 開啟 `App.tsx` 檔案，移除 `App()` 函式的內容。
2. 將 `App()` 函式的主體替換為以下程式碼：

```ts title="App.tsx (App function body replacement)"
const [street, setStreet] = React.useState('');
const [num, setNum] = React.useState('');
const [isValidAddress, setIsValidAddress] = React.useState<
  boolean | null
>(null);

const onPress = () => {
  let houseNum = parseInt(num, 10);
  if (isNaN(houseNum)) {
    houseNum = -1;
  }
  const address = {
    street,
    num: houseNum,
    isInUS: false,
  };
  const result = SampleTurboModule.validateAddress(address);
  setIsValidAddress(result);
};

return (
  <SafeAreaView style={styles.container}>
    <View>
      <Text style={styles.title}>
        Welcome to C Turbo Native Module Example
      </Text>
      <Text>Address:</Text>
      <TextInput
        style={styles.textInput}
        placeholder="Write your address here"
        onChangeText={setStreet}
        value={street}
      />
      <Text>Number:</Text>
      <TextInput
        style={styles.textInput}
        placeholder="Write your address here"
        onChangeText={setNum}
        value={num}
      />
      <Button title="Validate" onPress={onPress} />
      {isValidAddress != null && (
        <Text>
          Your address is {isValidAddress ? 'valid' : 'not valid'}
        </Text>
      )}
    </View>
  </SafeAreaView>
);
```

恭喜！🎉

你已成功將第一個類型從 JS 橋接到 C++。