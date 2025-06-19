---
id: strict-typescript-api
title: Strict TypeScript API (opt in)
---

嚴格 TypeScript API 是我們未來穩定版 React Native JavaScript API 的預覽版本。

具體而言，這是針對 `react-native` npm 套件的一套全新 TypeScript 類型定義，自 0.80 版本起提供。這些類型能提供更強健且更具未來性的類型準確性，讓我們能更有信心地將 React Native API 演進至穩定形態。採用嚴格 TypeScript API 會帶來一些結構性類型差異，因此這是一次性的破壞性變更。

新類型具有以下特點：

1. **直接從原始碼生成** — 提升覆蓋率與正確性，您可預期更強的相容性保證。
2. **限制於 `react-native` 索引檔案** — 更嚴格定義我們的公開 API，意味著進行內部檔案變更時不會破壞 API。

當社群準備就緒時，嚴格 TypeScript API 將成為我們未來的預設 API — 並與深層導入移除同步實施。

## 啟用方式

我們將這些新類型與現有類型一同發布，您可自行選擇遷移時機。我們鼓勵早期採用者與新建立的應用程式透過 `tsconfig.json` 檔案啟用此功能。

啟用嚴格類型屬於**破壞性變更**，因為部分新類型已更新名稱與結構，不過許多應用程式可能不受影響。您可在下一節了解各項破壞性變更的詳細資訊。

```json title="tsconfig.json"
{
  "extends": "@react-native/typescript-config",
  "compilerOptions": {
    ...
    "customConditions": ["react-native-strict-api"]
  }
}
```

:::note[Under the hood]

This will instruct TypeScript to resolve `react-native` types from our new [`types_generated/`](https://www.npmjs.com/package/react-native?activeTab=code) dir, instead of the previous [`types/`](https://www.npmjs.com/package/react-native?activeTab=code) dir (manually maintained). No restart of TypeScript or your editor is required.

:::

嚴格 TypeScript API 遵循我們移除 React Native 深層導入的 [RFC](https://github.com/react-native-community/discussions-and-proposals/pull/894) 規範。因此部分 API 不再從根目錄匯出，這是為了刻意減少 React Native API 的整體表面積。

:::tip[API 意見回饋]

**提交意見**：我們將與社群合作，在（至少）未來兩個 React Native 版本中確定最終要匯出的 API。請在我們的[意見討論串](https://github.com/react-native-community/discussions-and-proposals/discussions/893)分享您的想法。

另請參閱我們的[公告部落格文章](/blog/2025/06/12/moving-towards-a-stable-javascript-api)了解我們的動機與時程規劃。

:::

## 遷移指南

### 程式碼生成類型現在應從 `react-native` 套件導入

用於程式碼生成的類型（如 `Int32`、`Double`、`WithDefault` 等）現在統一歸類於 `CodegenTypes` 命名空間下。同樣地，`codegenNativeComponent` 和 `codegenNativeCommands` 現在也改為從 react-native 套件導入，而非使用深層導入路徑。

命名空間化的 `CodegenTypes` 以及 `codegenNativeCommands` 和 `codegenNativeComponent` 在未啟用嚴格 API 時仍可從 `react-native` 套件取得，以方便第三方函式庫進行適配。

**變更前**

```ts title=""
import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';
import type {
  Int32,
  WithDefault,
} from 'react-native/Libraries/Types/CodegenTypes';

interface NativeProps extends ViewProps {
  enabled?: WithDefault<boolean, true>;
  size?: Int32;
}

export default codegenNativeComponent<NativeProps>(
  'RNCustomComponent',
);
```

**變更後**

```ts title=""
import {CodegenTypes, codegenNativeComponent} from 'react-native';

interface NativeProps extends ViewProps {
  enabled?: CodegenTypes.WithDefault<boolean, true>;
  size?: CodegenTypes.Int32;
}

export default codegenNativeComponent<NativeProps>(
  'RNCustomComponent',
);
```

### 移除 `*Static` 類型

**變更前**

```tsx title=""
import {Linking, LinkingStatic} from 'react-native';

function foo(linking: LinkingStatic) {}
foo(Linking);
```

**變更後**

```tsx title=""
import {Linking} from 'react-native';

function foo(linking: Linking) {}
foo(Linking);
```

以下 API 先前以 `*Static` 命名並搭配該類型的變數宣告。多數情況下會存在別名，使值與類型能以相同識別符號匯出，但部分 API 缺少此別名。

(For example there was an `AlertStatic` type, `Alert` variable of type `AlertStatic` and type `Alert` which was an alias for `AlertStatic`. But in the case of `PixelRatio` there was a `PixelRatioStatic` type and a `PixelRatio` variable of that type without additional type aliases.)

**受影響的 API**

- `AlertStatic`
- `ActionSheetIOSStatic`
- `ToastAndroidStatic`
- `InteractionManagerStatic`（此案例中沒有對應的 `InteractionManager` 類型別名）
- `UIManagerStatic`
- `PlatformStatic`
- `SectionListStatic`
- `PixelRatioStatic`（此案例中沒有對應的 `PixelRatio` 類型別名）
- `AppStateStatic`
- `AccessibilityInfoStatic`
- `ImageResizeModeStatic`
- `BackHandlerStatic`
- `DevMenuStatic`（此案例中沒有對應的 `DevMenu` 類型別名）
- `ClipboardStatic`
- `PermissionsAndroidStatic`
- `ShareStatic`
- `DeviceEventEmitterStatic`
- `LayoutAnimationStatic`
- `KeyboardStatic`（此案例中沒有對應的 `Keyboard` 類型別名）
- `DevSettingsStatic`（此案例中沒有對應的 `DevSettings` 類型別名）
- `I18nManagerStatic`
- `EasingStatic`
- `PanResponderStatic`
- `NativeModulesStatic`（此案例中沒有對應的 `NativeModules` 類型別名）
- `LogBoxStatic`
- `PushNotificationIOSStatic`
- `SettingsStatic`
- `VibrationStatic`

### 部分核心元件現在改為函式元件而非類別元件

- `View`
- `Image`
- `TextInput`
- `Modal`
- `Text`
- `TouchableWithoutFeedback`
- `Switch`
- `ActivityIndicator`
- `ProgressBarAndroid`
- `InputAccessoryView`
- `Button`
- `SafeAreaView`

由於此變更，存取這些視圖的 ref 類型需改用 `React.ComponentRef<typeof View>` 模式，此模式無論對類別元件或函式元件皆適用，例如：

```ts title=""
const ref = useRef<React.ComponentRef<typeof View>>(null);
```

## 其他重大變更

### Animated 類型的調整

Animated 節點原先會根據插值輸出結果定義為泛型類型，現在改為非泛型類型並搭配泛型 `interpolate` 方法。

`Animated.LegacyRef` 已被移除。

### 統一選擇性屬性的類型

在新類型系統中，所有選擇性屬性都會被定義為 `type | undefined`。

### 移除部分已棄用類型

所有列在 [`DeprecatedPropertiesAlias.d.ts`](https://github.com/facebook/react-native/blob/0.80-stable/packages/react-native/types/public/DeprecatedPropertiesAlias.d.ts) 的類型在 Strict API 下皆無法存取。

### 移除殘留的元件屬性

Some properties that were defined in type definitions but were not used by the component or were lacking a definition were removed (for example: `lineBreakMode` on `Text`, `scrollWithoutAnimationTo` on `ScrollView`, transform styles defined outside of transform array).

### 原先可存取的私有類型輔助工具可能已被移除

由於舊版類型定義的配置，所有定義的類型都能從 `react-native` 套件中存取。這包含未明確導出的類型以及僅供內部使用的輔助類型。

較顯著的例子包含與 StyleSheet 相關的類型（如 `RecursiveArray`、`RegisteredStyle` 和 `Falsy`）以及 Animated 相關的類型（如 `WithAnimatedArray` 和 `WithAnimatedObject`）。