---
id: shadow-props
title: Shadow Props
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Shadow%20Props&supportedPlatforms=ios&ext=js&dependencies=@react-native-community/slider
import React, {useState} from 'react';
import {Text, View, StyleSheet} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';
import Slider from '@react-native-community/slider';

const ShadowPropSlider = ({label, value, ...props}) => {
  return (
    <>
      <Text>
        {label} ({value.toFixed(2)})
      </Text>
      <Slider step={1} value={value} {...props} />
    </>
  );
};

const App = () => {
  const [shadowOffsetWidth, setShadowOffsetWidth] = useState(0);
  const [shadowOffsetHeight, setShadowOffsetHeight] = useState(0);
  const [shadowRadius, setShadowRadius] = useState(0);
  const [shadowOpacity, setShadowOpacity] = useState(0.1);

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <View
          style={[
            styles.square,
            {
              shadowOffset: {
                width: shadowOffsetWidth,
                height: -shadowOffsetHeight,
              },
              shadowOpacity,
              shadowRadius,
            },
          ]}
        />
        <View style={styles.controls}>
          <ShadowPropSlider
            label="shadowOffset - X"
            minimumValue={-50}
            maximumValue={50}
            value={shadowOffsetWidth}
            onValueChange={setShadowOffsetWidth}
          />
          <ShadowPropSlider
            label="shadowOffset - Y"
            minimumValue={-50}
            maximumValue={50}
            value={shadowOffsetHeight}
            onValueChange={setShadowOffsetHeight}
          />
          <ShadowPropSlider
            label="shadowRadius"
            minimumValue={0}
            maximumValue={100}
            value={shadowRadius}
            onValueChange={setShadowRadius}
          />
          <ShadowPropSlider
            label="shadowOpacity"
            minimumValue={0}
            maximumValue={1}
            step={0.05}
            value={shadowOpacity}
            onValueChange={val => setShadowOpacity(val)}
          />
        </View>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'space-around',
    backgroundColor: '#ecf0f1',
    padding: 8,
  },
  square: {
    alignSelf: 'center',
    backgroundColor: 'white',
    borderRadius: 4,
    height: 150,
    shadowColor: 'black',
    width: 150,
  },
  controls: {
    paddingHorizontal: 12,
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Shadow%20Props&supportedPlatforms=ios&ext=tsx&dependencies=@react-native-community/slider
import React, {useState} from 'react';
import {Text, View, StyleSheet} from 'react-native';
import Slider, {SliderProps} from '@react-native-community/slider';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

type ShadowPropSliderProps = SliderProps & {
  label: string;
};

const ShadowPropSlider = ({label, value, ...props}: ShadowPropSliderProps) => {
  return (
    <>
      <Text>
        {label} ({value?.toFixed(2)})
      </Text>
      <Slider step={1} value={value} {...props} />
    </>
  );
};

const App = () => {
  const [shadowOffsetWidth, setShadowOffsetWidth] = useState(0);
  const [shadowOffsetHeight, setShadowOffsetHeight] = useState(0);
  const [shadowRadius, setShadowRadius] = useState(0);
  const [shadowOpacity, setShadowOpacity] = useState(0.1);

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <View
          style={[
            styles.square,
            {
              shadowOffset: {
                width: shadowOffsetWidth,
                height: -shadowOffsetHeight,
              },
              shadowOpacity,
              shadowRadius,
            },
          ]}
        />
        <View style={styles.controls}>
          <ShadowPropSlider
            label="shadowOffset - X"
            minimumValue={-50}
            maximumValue={50}
            value={shadowOffsetWidth}
            onValueChange={setShadowOffsetWidth}
          />
          <ShadowPropSlider
            label="shadowOffset - Y"
            minimumValue={-50}
            maximumValue={50}
            value={shadowOffsetHeight}
            onValueChange={setShadowOffsetHeight}
          />
          <ShadowPropSlider
            label="shadowRadius"
            minimumValue={0}
            maximumValue={100}
            value={shadowRadius}
            onValueChange={setShadowRadius}
          />
          <ShadowPropSlider
            label="shadowOpacity"
            minimumValue={0}
            maximumValue={1}
            step={0.05}
            value={shadowOpacity}
            onValueChange={val => setShadowOpacity(val)}
          />
        </View>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'space-around',
    backgroundColor: '#ecf0f1',
    padding: 8,
  },
  square: {
    alignSelf: 'center',
    backgroundColor: 'white',
    borderRadius: 4,
    height: 150,
    shadowColor: 'black',
    width: 150,
  },
  controls: {
    paddingHorizontal: 12,
  },
});

export default App;
```

</TabItem>
</Tabs>

---

# 參考文獻

React Native 提供三組陰影 API：

- `boxShadow`：一個 View 樣式屬性，完全符合 [同名網頁樣式屬性](https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow) 的規範。
- `dropShadow`：作為 [`filter`](./view-style-props#filter) View 樣式屬性的一部分，提供特定的濾鏡功能。
- 各種 `shadow` 屬性（`shadowColor`、`shadowOffset`、`shadowOpacity`、`shadowRadius`）：這些直接對應到平台層級 API 所暴露的原生功能。

The difference between `dropShadow` and `boxShadow` are as follows:

- `dropShadow` exists as part of `filter`, whereas `boxShadow` is a standalone style prop.
- `dropShadow` is an alpha mask, so only pixels with a positive alpha value will "cast" a shadow. `boxShadow` will cast around the border box of the element no matter it's contents (unless it is inset).
- `dropShadow` is only available on Android, `boxShadow` is available on iOS and Android.
- `dropShadow` cannot be inset like `boxShadow`.
- `dropShadow` does not have the `spreadDistance` argument like `boxShadow`.

Both `boxShadow` and `dropShadow` are generally more capable than the `shadow` props. The `shadow` props, however, map to native platform-level APIs, so if you only need a straightforward shadow these props are recommended. Note that only `shadowColor` works on both Android and iOS, all other `shadow` props only work on iOS.

## 屬性

### `boxShadow`

詳見 [View 樣式屬性](./view-style-props#boxshadow) 的文件。

### `dropShadow` <div class="label android">Android</div>

詳見 [View 樣式屬性](./view-style-props#filter) 的文件。

### `shadowColor`

設定陰影的顏色。

此屬性僅在 Android API 28 及以上版本中有效。對於較低版本的 Android API，請使用 [`elevation` 屬性](view-style-props#elevation-android) 來實現類似功能。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `shadowOffset` <div class="label ios">iOS</div>

設定陰影的偏移量。

| Type                                     |
| ---------------------------------------- |
| object: `{width: number,height: number}` |

---

### `shadowOpacity` <div class="label ios">iOS</div>

設定陰影的不透明度（乘以顏色的 alpha 分量）。

| Type   |
| ------ |
| number |

---

### `shadowRadius` <div class="label ios">iOS</div>

設定陰影的模糊半徑。

| Type   |
| ------ |
| number |