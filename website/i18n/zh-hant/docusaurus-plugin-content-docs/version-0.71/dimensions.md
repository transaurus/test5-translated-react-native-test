---
id: dimensions
title: Dimensions
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

> [`useWindowDimensions`](usewindowdimensions) is the preferred API for React components. Unlike `Dimensions`, it updates as the window's dimensions update. This works nicely with the React paradigm.

```tsx
import {Dimensions} from 'react-native';
```

你可以使用以下程式碼取得應用程式視窗的寬度和高度：

```tsx
const windowWidth = Dimensions.get('window').width;
const windowHeight = Dimensions.get('window').height;
```

> 雖然尺寸可以立即取得，但它們可能會發生變化（例如由於設備旋轉、折疊式設備等），因此任何依賴這些常數的渲染邏輯或樣式都應該在每次渲染時調用此函數，而不是快取值（例如使用內聯樣式而不是在 `StyleSheet` 中設定值）。

如果你的目標是折疊式設備或可以改變螢幕尺寸或應用視窗大小的設備，你可以使用 Dimensions 模組中的事件監聽器，如下例所示。

## 範例

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Dimensions
import React, {useState, useEffect} from 'react';
import {View, StyleSheet, Text, Dimensions} from 'react-native';

const windowDimensions = Dimensions.get('window');
const screenDimensions = Dimensions.get('screen');

const App = () => {
  const [dimensions, setDimensions] = useState({
    window: windowDimensions,
    screen: screenDimensions,
  });

  useEffect(() => {
    const subscription = Dimensions.addEventListener(
      'change',
      ({window, screen}) => {
        setDimensions({window, screen});
      },
    );
    return () => subscription?.remove();
  });

  return (
    <View style={styles.container}>
      <Text style={styles.header}>Window Dimensions</Text>
      {Object.entries(dimensions.window).map(([key, value]) => (
        <Text>
          {key} - {value}
        </Text>
      ))}
      <Text style={styles.header}>Screen Dimensions</Text>
      {Object.entries(dimensions.screen).map(([key, value]) => (
        <Text>
          {key} - {value}
        </Text>
      ))}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  header: {
    fontSize: 16,
    marginVertical: 10,
  },
});

export default App;
```

</TabItem>
<TabItem value="classical">

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Dimensions&ext=js
import React, {Component} from 'react';
import {View, StyleSheet, Text, Dimensions} from 'react-native';

const windowDimensions = Dimensions.get('window');
const screenDimensions = Dimensions.get('screen');

class App extends Component {
  state = {
    dimensions: {
      window: windowDimensions,
      screen: screenDimensions,
    },
  };

  onChange = ({window, screen}) => {
    this.setState({dimensions: {window, screen}});
  };

  componentDidMount() {
    this.dimensionsSubscription = Dimensions.addEventListener(
      'change',
      this.onChange,
    );
  }

  componentWillUnmount() {
    this.dimensionsSubscription?.remove();
  }

  render() {
    const {
      dimensions: {window, screen},
    } = this.state;

    return (
      <View style={styles.container}>
        <Text style={styles.header}>Window Dimensions</Text>
        {Object.entries(window).map(([key, value]) => (
          <Text>
            {key} - {value}
          </Text>
        ))}
        <Text style={styles.header}>Screen Dimensions</Text>
        {Object.entries(screen).map(([key, value]) => (
          <Text>
            {key} - {value}
          </Text>
        ))}
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  header: {
    fontSize: 16,
    marginVertical: 10,
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Dimensions&ext=tsx
import React, {Component} from 'react';
import {View, StyleSheet, Text, Dimensions} from 'react-native';
import type {EmitterSubscription, ScaledSize} from 'react-native';

const windowDimensions = Dimensions.get('window');
const screenDimensions = Dimensions.get('screen');

class App extends Component {
  dimensionsSubscription?: EmitterSubscription;
  state = {
    dimensions: {
      window: windowDimensions,
      screen: screenDimensions,
    },
  };

  onChange = ({window, screen}: {window: ScaledSize; screen: ScaledSize}) => {
    this.setState({dimensions: {window, screen}});
  };

  componentDidMount() {
    this.dimensionsSubscription = Dimensions.addEventListener(
      'change',
      this.onChange,
    );
  }

  componentWillUnmount() {
    this.dimensionsSubscription?.remove();
  }

  render() {
    const {
      dimensions: {window, screen},
    } = this.state;

    return (
      <View style={styles.container}>
        <Text style={styles.header}>Window Dimensions</Text>
        {Object.entries(window).map(([key, value]) => (
          <Text>
            {key} - {value}
          </Text>
        ))}
        <Text style={styles.header}>Screen Dimensions</Text>
        {Object.entries(screen).map(([key, value]) => (
          <Text>
            {key} - {value}
          </Text>
        ))}
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  header: {
    fontSize: 16,
    marginVertical: 10,
  },
});

export default App;
```

</TabItem>
</Tabs>

</TabItem>
</Tabs>

# 參考

## 方法

### `addEventListener()`

```tsx
static addEventListener(
  type: 'change',
  handler: ({
    window,
    screen,
  }: DimensionsValue) => void,
): EmitterSubscription;
```

添加事件處理程序。支援的事件：

- `change`：當 `Dimensions` 物件中的屬性發生變化時觸發。事件處理程序的參數是一個 [`DimensionsValue`](#dimensionsvalue) 類型的物件。

---

### `get()`

```tsx
static get(dim: 'window' | 'screen'): ScaledSize;
```

初始尺寸在 `runApplication` 被調用之前就已設定，因此它們應該在其他任何 require 執行之前就可取得，但可能會在之後更新。

範例：`const {height, width} = Dimensions.get('window');`

**參數：**

| Name                                                               | Type   | Description                                                                       |
| ------------------------------------------------------------------ | ------ | --------------------------------------------------------------------------------- |
| dim <div className="label basic required two-lines">Required</div> | string | Name of dimension as defined when calling `set`. Returns value for the dimension. |

> For Android the `window` dimension will exclude the size used by the `status bar` (if not translucent) and `bottom navigation bar`

---

## 類型定義

### DimensionsValue

**屬性：**

| Name   | Type                                | Description                             |
| ------ | ----------------------------------- | --------------------------------------- |
| window | [ScaledSize](dimensions#scaledsize) | Size of the visible Application window. |
| screen | [ScaledSize](dimensions#scaledsize) | Size of the device's screen.            |

### ScaledSize

| Type   |
| ------ |
| object |

**屬性：**

| Name      | Type   |
| --------- | ------ |
| width     | number |
| height    | number |
| scale     | number |
| fontScale | number |