---
id: stylesheet
title: StyleSheet
---

StyleSheet 是一個類似於 CSS 樣式表的抽象層

```SnackPlayer name=StyleSheet
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={styles.container}>
    <Text style={styles.title}>React Native</Text>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
    backgroundColor: '#eaeaea',
  },
  title: {
    marginTop: 16,
    paddingVertical: 8,
    borderWidth: 4,
    borderColor: '#20232a',
    borderRadius: 6,
    backgroundColor: '#61dafb',
    color: '#20232a',
    textAlign: 'center',
    fontSize: 30,
    fontWeight: 'bold',
  },
});

export default App;
```

程式碼品質建議：

- 將樣式移出渲染函數，能使程式碼更易於理解。
- 為樣式命名有助於為渲染函數中的底層元件賦予意義。

---

# 參考文獻

## 方法

### `compose()`

```tsx
static compose(style1: Object, style2: Object): Object | Object[];
```

合併兩個樣式，使 `style2` 會覆蓋 `style1` 中的任何樣式。如果任一樣式為假值，則直接返回另一個樣式而不分配陣列，節省記憶體分配並保持 PureComponent 檢查的引用相等性。

```SnackPlayer name=Compose
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={container}>
    <Text style={text}>React Native</Text>
  </View>
);

const page = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
    backgroundColor: '#fff',
  },
  text: {
    fontSize: 30,
    color: '#000',
  },
});

const lists = StyleSheet.create({
  listContainer: {
    flex: 1,
    backgroundColor: '#61dafb',
  },
  listItem: {
    fontStyle: 'italic',
    fontWeight: 'bold',
  },
});

const container = StyleSheet.compose(page.container, lists.listContainer);
const text = StyleSheet.compose(page.text, lists.listItem);

export default App;
```

---

### `create()`

```tsx
static create(styles: Object): Object;
```

從給定的物件創建一個 StyleSheet 樣式引用。

---

### `flatten()`

```tsx
static flatten(style: Object[]): Object;
```

將樣式物件陣列扁平化為一個聚合的樣式物件。此方法也可用於查閱由 `StyleSheet.register` 返回的 ID。

> **注意：** 請謹慎使用，濫用此方法可能會影響優化效果。ID 能透過橋接器和記憶體實現優化。直接引用樣式物件將使您無法利用這些優化。

```SnackPlayer name=Flatten
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={page.container}>
    <Text style={flattenStyle}>React Native</Text>
    <Text>Flatten Style</Text>
    <Text style={page.code}>{JSON.stringify(flattenStyle, null, 2)}</Text>
  </View>
);

const page = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
    alignItems: 'center',
  },
  text: {
    color: '#000',
    fontSize: 14,
    fontWeight: 'bold',
  },
  code: {
    marginTop: 12,
    padding: 12,
    borderRadius: 8,
    color: '#666',
    backgroundColor: '#eaeaea',
  },
});

const typography = StyleSheet.create({
  header: {
    color: '#61dafb',
    fontSize: 30,
    marginBottom: 36,
  },
});

const flattenStyle = StyleSheet.flatten([page.text, typography.header]);

export default App;
```

此方法內部使用 `StyleSheetRegistry.getStyleByID(style)` 來解析由 ID 表示的樣式物件。因此，樣式物件陣列（`StyleSheet.create()` 的實例）會被逐一解析為各自的物件，合併為一個後返回。這也解釋了其替代用途。

---

### `setStyleAttributePreprocessor()`

> **警告：實驗性功能。** 可能會頻繁發生破壞性變更且不另行通知。整個功能可能會被刪除，誰知道呢？使用風險自負。

```tsx
static setStyleAttributePreprocessor(
  property: string,
  process: (propValue: any) => any,
);
```

設置一個函數用於預處理樣式屬性值。此功能內部用於處理顏色和變換值。除非您確切知道自己在做什麼且已嘗試其他選項，否則不應使用此方法。

## 屬性

---

### `absoluteFill`

一種非常常見的模式是創建具有絕對定位和零位置（`position: 'absolute', left: 0, right: 0, top: 0, bottom: 0`）的疊加層，因此可使用 `absoluteFill` 來簡化操作並減少這些重複樣式的冗餘。如果需要，absoluteFill 可用於在 StyleSheet 中創建自定義條目，例如：

```SnackPlayer name=absoluteFill
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={styles.container}>
    <View style={styles.box1}>
      <Text style={styles.text}>1</Text>
    </View>
    <View style={[styles.box2, StyleSheet.absoluteFill]}>
      <Text style={styles.text}>2</Text>
    </View>
    <View style={styles.box3}>
      <Text style={styles.text}>3</Text>
    </View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  box1: {
    position: 'absolute',
    top: 40,
    left: 40,
    width: 100,
    height: 100,
    backgroundColor: 'red',
  },
  box2: {
    width: 100,
    height: 100,
    backgroundColor: 'blue',
  },
  box3: {
    position: 'absolute',
    top: 120,
    left: 120,
    width: 100,
    height: 100,
    backgroundColor: 'green',
  },
  text: {
    color: '#FFF',
    fontSize: 80,
  },
});

export default App;
```

---

### `absoluteFillObject`

有時您可能需要 `absoluteFill` 但稍作調整 - `absoluteFillObject` 可用於在 `StyleSheet` 中創建自定義條目，例如：

```SnackPlayer name=absoluteFillObject
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={styles.container}>
    <View style={styles.box1}>
      <Text style={styles.text}>1</Text>
    </View>
    <View style={styles.box2}>
      <Text style={styles.text}>2</Text>
    </View>
    <View style={styles.box3}>
      <Text style={styles.text}>3</Text>
    </View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  box1: {
    position: 'absolute',
    top: 40,
    left: 40,
    width: 100,
    height: 100,
    backgroundColor: 'red',
  },
  box2: {
    ...StyleSheet.absoluteFillObject,
    top: 120,
    left: 50,
    width: 100,
    height: 100,
    backgroundColor: 'blue',
  },
  box3: {
    ...StyleSheet.absoluteFillObject,
    top: 120,
    left: 120,
    width: 100,
    height: 100,
    backgroundColor: 'green',
  },
  text: {
    color: '#FFF',
    fontSize: 80,
  },
});

export default App;
```

---

### `hairlineWidth`

此屬性定義為平台上細線的寬度。可用作邊框的厚度或兩個元素之間的分隔線。範例：

```SnackPlayer name=hairlineWidth
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={styles.container}>
    <Text style={styles.row}>React</Text>
    <Text style={styles.row}>Native</Text>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
  },
  row: {
    padding: 4,
    borderBottomColor: 'red',
    borderBottomWidth: StyleSheet.hairlineWidth,
  },
});

export default App;
```

此常數始終為像素的整數（因此由其定義的線條看起來清晰），並會嘗試匹配底層平台上細線的標準寬度。然而，您不應依賴其為固定大小，因為在不同平台和螢幕密度下，其值可能計算方式不同。

如果您的模擬器縮小比例，則具有 hairline 寬度的線條可能不可見。