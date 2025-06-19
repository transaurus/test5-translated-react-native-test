---
id: flexbox
title: Layout with Flexbox
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

元件可以透過 Flexbox 演算法來指定其子元素的佈局方式。Flexbox 的設計目的是在不同螢幕尺寸上提供一致的佈局。

通常你會結合使用 `flexDirection`、`alignItems` 和 `justifyContent` 來達成正確的佈局。

:::caution
Flexbox 在 React Native 中的運作方式與在網頁 CSS 中相同，但有幾個例外。
預設值不同：`flexDirection` 預設為 `column` 而非 `row`，`alignContent` 預設為 `flex-start` 而非 `stretch`，`flexShrink` 預設為 `0` 而非 `1`，且 `flex` 參數僅支援單一數字。
:::

## Flex 彈性佈局

[`flex`](layout-props#flex) will define how your items are going to **“fill”** over the available space along your main axis. Space will be divided according to each element's flex property.

在以下範例中，紅色、橙色和綠色視圖都是設定了 `flex: 1` 的容器視圖的子元素。紅色視圖使用 `flex: 1`，橙色視圖使用 `flex: 2`，綠色視圖使用 `flex: 3`。**1+2+3 = 6**，這意味著紅色視圖將獲得 `1/6` 的空間，橙色視圖獲得 `2/6` 的空間，綠色視圖獲得 `3/6` 的空間。

```SnackPlayer name=Flex%20Example
import React from 'react';
import {StyleSheet, View} from 'react-native';

const Flex = () => {
  return (
    <View
      style={[
        styles.container,
        {
          // Try setting `flexDirection` to `"row"`.
          flexDirection: 'column',
        },
      ]}>
      <View style={{flex: 1, backgroundColor: 'red'}} />
      <View style={{flex: 2, backgroundColor: 'darkorange'}} />
      <View style={{flex: 3, backgroundColor: 'green'}} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
});

export default Flex;
```

## Flex Direction 主軸方向

[`flexDirection`](layout-props#flexdirection) 控制節點子元素的排列方向。這也被稱為主軸。交叉軸是與主軸垂直的軸，或是換行線排列的軸。

- `column` (**預設值**) 從上到下排列子元素。如果啟用換行，則下一行將從容器頂部第一個項目的右側開始。
- `row` 從左到右排列子元素。如果啟用換行，則下一行將從容器左側第一個項目的下方開始。
- `column-reverse` 從下到上排列子元素。如果啟用換行，則下一行將從容器底部第一個項目的右側開始。
- `row-reverse` 從右到左排列子元素。如果啟用換行，則下一行將從容器右側第一個項目的下方開始。

你可以在此[了解更多](https://www.yogalayout.dev/docs/styling/flex-direction)。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Flex%20Direction&ext=js
import React, {useState} from 'react';
import {StyleSheet, Text, TouchableOpacity, View} from 'react-native';

const FlexDirectionBasics = () => {
  const [flexDirection, setflexDirection] = useState('column');

  return (
    <PreviewLayout
      label="flexDirection"
      values={['column', 'row', 'row-reverse', 'column-reverse']}
      selectedValue={flexDirection}
      setSelectedValue={setflexDirection}>
      <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
    </PreviewLayout>
  );
};

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default FlexDirectionBasics;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Flex%20Direction&ext=tsx
import React, {useState} from 'react';
import {StyleSheet, Text, TouchableOpacity, View} from 'react-native';
import type {PropsWithChildren} from 'react';

const FlexDirectionBasics = () => {
  const [flexDirection, setflexDirection] = useState('column');

  return (
    <PreviewLayout
      label="flexDirection"
      values={['column', 'row', 'row-reverse', 'column-reverse']}
      selectedValue={flexDirection}
      setSelectedValue={setflexDirection}>
      <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  label: string;
  values: string[];
  selectedValue: string;
  setSelectedValue: (value: string) => void;
}>;

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}: PreviewLayoutProps) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default FlexDirectionBasics;
```

</TabItem>
</Tabs>

## Layout Direction 佈局方向

佈局 [`direction`](layout-props#direction) 指定了層次結構中子元素和文字的排列方向。佈局方向也會影響 `start` 和 `end` 所指的邊緣。預設情況下，React Native 使用 LTR（從左到右）的佈局方向。在此模式下，`start` 指的是左側，`end` 指的是右側。

- `LTR` (**預設值**) 文字和子元素從左到右排列。應用於元素起始位置的 margin 和 padding 會作用於左側。
- `RTL` 文字和子元素從右到左排列。應用於元素起始位置的 margin 和 padding 會作用於右側。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Flex%20Direction&ext=js
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';

const DirectionLayout = () => {
  const [direction, setDirection] = useState('ltr');

  return (
    <PreviewLayout
      label="direction"
      selectedValue={direction}
      values={['ltr', 'rtl']}
      setSelectedValue={setDirection}>
      <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
    </PreviewLayout>
  );
};

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default DirectionLayout;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Flex%20Direction&ext=tsx
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';
import type {PropsWithChildren} from 'react';

const DirectionLayout = () => {
  const [direction, setDirection] = useState('ltr');

  return (
    <PreviewLayout
      label="direction"
      selectedValue={direction}
      values={['ltr', 'rtl']}
      setSelectedValue={setDirection}>
      <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  label: string;
  values: string[];
  selectedValue: string;
  setSelectedValue: (value: string) => void;
}>;

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}: PreviewLayoutProps) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default DirectionLayout;
```

</TabItem>
</Tabs>

## Justify Content 主軸對齊

[`justifyContent`](layout-props#justifycontent) 描述了如何在容器的主軸上對齊子元素。例如，你可以使用此屬性在 `flexDirection` 設為 `row` 的容器中水平居中子元素，或在 `flexDirection` 設為 `column` 的容器中垂直居中子元素。

- `flex-start`（**預設值**）將容器的子元素對齊到主軸的起始端。

- `flex-end` 將容器的子元素對齊到主軸的末端。

- `center` 將容器的子元素在主軸上居中對齊。

- `space-between` 在主軸上均勻分布子元素，剩餘空間分配在子元素之間。

- `space-around` 在主軸上均勻分布子元素，剩餘空間環繞在子元素周圍。與 `space-between` 相比，使用 `space-around` 會在第一個子元素之前和最後一個子元素之後也分配空間。

- `space-evenly` 在主軸上完全均勻分布子元素。每對相鄰子元素之間的間距、主軸起始端與第一個子元素的間距、主軸末端與最後一個子元素的間距都完全相同。

你可以在此[了解更多](https://www.yogalayout.dev/docs/styling/justify-content)。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Justify%20Content&ext=js
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';

const JustifyContentBasics = () => {
  const [justifyContent, setJustifyContent] = useState('flex-start');

  return (
    <PreviewLayout
      label="justifyContent"
      selectedValue={justifyContent}
      values={[
        'flex-start',
        'flex-end',
        'center',
        'space-between',
        'space-around',
        'space-evenly',
      ]}
      setSelectedValue={setJustifyContent}>
      <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
    </PreviewLayout>
  );
};

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default JustifyContentBasics;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Justify%20Content&ext=tsx
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';
import type {PropsWithChildren} from 'react';

const JustifyContentBasics = () => {
  const [justifyContent, setJustifyContent] = useState('flex-start');

  return (
    <PreviewLayout
      label="justifyContent"
      selectedValue={justifyContent}
      values={[
        'flex-start',
        'flex-end',
        'center',
        'space-between',
        'space-around',
        'space-evenly',
      ]}
      setSelectedValue={setJustifyContent}>
      <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  label: string;
  values: string[];
  selectedValue: string;
  setSelectedValue: (value: string) => void;
}>;

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}: PreviewLayoutProps) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default JustifyContentBasics;
```

</TabItem>
</Tabs>

## 對齊項目 (Align Items)

[`alignItems`](layout-props#alignitems) 描述如何沿容器的交叉軸對齊子元素。它與 `justifyContent` 非常相似，但 `alignItems` 作用於交叉軸而非主軸。

- `stretch`（**預設值**）拉伸容器的子元素以匹配容器交叉軸的 `height`。

- `flex-start` 將容器的子元素對齊到交叉軸的起始端。

- `flex-end` 將容器的子元素對齊到交叉軸的末端。

- `center` 將容器的子元素在交叉軸上居中對齊。

- `baseline` 將容器的子元素沿基準線對齊。可以為每個子元素設置其父元素的基準線參考。

:::info
For `stretch` to have an effect, children must not have a fixed dimension along the secondary axis. In the following example, setting `alignItems: stretch` does nothing until the `width: 50` is removed from the children.
:::

你可以在此[了解更多](https://www.yogalayout.dev/docs/styling/align-items-self)。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Align%20Items&ext=js
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';

const AlignItemsLayout = () => {
  const [alignItems, setAlignItems] = useState('stretch');

  return (
    <PreviewLayout
      label="alignItems"
      selectedValue={alignItems}
      values={['stretch', 'flex-start', 'flex-end', 'center', 'baseline']}
      setSelectedValue={setAlignItems}>
      <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View
        style={[
          styles.box,
          {
            backgroundColor: 'steelblue',
            width: 'auto',
            minWidth: 50,
          },
        ]}
      />
    </PreviewLayout>
  );
};

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
    minHeight: 200,
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default AlignItemsLayout;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Align%20Items&ext=tsx
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';
import type {PropsWithChildren} from 'react';

const AlignItemsLayout = () => {
  const [alignItems, setAlignItems] = useState('stretch');

  return (
    <PreviewLayout
      label="alignItems"
      selectedValue={alignItems}
      values={['stretch', 'flex-start', 'flex-end', 'center', 'baseline']}
      setSelectedValue={setAlignItems}>
      <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View
        style={[
          styles.box,
          {
            backgroundColor: 'steelblue',
            width: 'auto',
            minWidth: 50,
          },
        ]}
      />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  label: string;
  values: string[];
  selectedValue: string;
  setSelectedValue: (value: string) => void;
}>;

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}: PreviewLayoutProps) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
    minHeight: 200,
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default AlignItemsLayout;
```

</TabItem>
</Tabs>

## 自身對齊 (Align Self)

[`alignSelf`](layout-props#alignself) 的選項和效果與 `alignItems` 相同，但它作用於單個子元素，用於改變該子元素在其父容器中的對齊方式。`alignSelf` 會覆蓋父容器通過 `alignItems` 設置的任何選項。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Align%20Self&ext=js
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';

const AlignSelfLayout = () => {
  const [alignSelf, setAlignSelf] = useState('stretch');

  return (
    <PreviewLayout
      label="alignSelf"
      selectedValue={alignSelf}
      values={['stretch', 'flex-start', 'flex-end', 'center', 'baseline']}
      setSelectedValue={setAlignSelf}>
      <View
        style={[
          styles.box,
          {
            alignSelf,
            width: 'auto',
            minWidth: 50,
            backgroundColor: 'powderblue',
          },
        ]}
      />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
    </PreviewLayout>
  );
};

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={styles.container}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
    minHeight: 200,
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default AlignSelfLayout;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Align%20Self&ext=tsx
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';
import type {PropsWithChildren} from 'react';
import type {FlexAlignType} from 'react-native';

const AlignSelfLayout = () => {
  const [alignSelf, setAlignSelf] = useState<FlexAlignType>('stretch');

  return (
    <PreviewLayout
      label="alignSelf"
      selectedValue={alignSelf}
      values={['stretch', 'flex-start', 'flex-end', 'center', 'baseline']}
      setSelectedValue={setAlignSelf}>
      <View
        style={[
          styles.box,
          {
            alignSelf,
            width: 'auto',
            minWidth: 50,
            backgroundColor: 'powderblue',
          },
        ]}
      />
      <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  label: string;
  values: FlexAlignType[];
  selectedValue: string;
  setSelectedValue: (value: FlexAlignType) => void;
}>;

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}: PreviewLayoutProps) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={styles.container}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
    minHeight: 200,
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default AlignSelfLayout;
```

</TabItem>
</Tabs>

## 對齊內容 (Align Content)

[alignContent](layout-props#aligncontent) 定義了多行內容在交叉軸上的分布方式。此屬性僅在使用 `flexWrap` 將項目換行為多行時生效。

- `flex-start`（**預設值**）將換行後的內容對齊到容器交叉軸的起始端。

- `flex-end` 將換行後的內容對齊到容器交叉軸的末端。

- `stretch`（_在網頁版 Yoga 中的預設值_）拉伸換行後的內容以匹配容器交叉軸的高度。

- `center` 將換行後的內容在交叉軸上居中對齊。

- `space-between` 在交叉軸上均勻分布換行後的內容，剩餘空間分配在行與行之間。

- `space-around` 在交叉軸上均勻分布換行後的內容，剩餘空間環繞在行周圍。容器兩端的空間大小為行間空間的一半。

- `space-evenly` 在交叉軸上完全均勻分布換行後的內容，剩餘空間完全均勻分配。每段空間大小相同。

你可以在此[了解更多](https://www.yogalayout.dev/docs/styling/align-content)。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Align%20Content&ext=js
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';

const AlignContentLayout = () => {
  const [alignContent, setAlignContent] = useState('flex-start');

  return (
    <PreviewLayout
      label="alignContent"
      selectedValue={alignContent}
      values={[
        'flex-start',
        'flex-end',
        'stretch',
        'center',
        'space-between',
        'space-around',
      ]}
      setSelectedValue={setAlignContent}>
      <View style={[styles.box, {backgroundColor: 'orangered'}]} />
      <View style={[styles.box, {backgroundColor: 'orange'}]} />
      <View style={[styles.box, {backgroundColor: 'mediumseagreen'}]} />
      <View style={[styles.box, {backgroundColor: 'deepskyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'mediumturquoise'}]} />
      <View style={[styles.box, {backgroundColor: 'mediumslateblue'}]} />
      <View style={[styles.box, {backgroundColor: 'purple'}]} />
    </PreviewLayout>
  );
};

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexWrap: 'wrap',
    marginTop: 8,
    backgroundColor: 'aliceblue',
    maxHeight: 400,
  },
  box: {
    width: 50,
    height: 80,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default AlignContentLayout;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Align%20Content&ext=tsx
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';
import type {PropsWithChildren} from 'react';

const AlignContentLayout = () => {
  const [alignContent, setAlignContent] = useState('flex-start');

  return (
    <PreviewLayout
      label="alignContent"
      selectedValue={alignContent}
      values={[
        'flex-start',
        'flex-end',
        'stretch',
        'center',
        'space-between',
        'space-around',
      ]}
      setSelectedValue={setAlignContent}>
      <View style={[styles.box, {backgroundColor: 'orangered'}]} />
      <View style={[styles.box, {backgroundColor: 'orange'}]} />
      <View style={[styles.box, {backgroundColor: 'mediumseagreen'}]} />
      <View style={[styles.box, {backgroundColor: 'deepskyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'mediumturquoise'}]} />
      <View style={[styles.box, {backgroundColor: 'mediumslateblue'}]} />
      <View style={[styles.box, {backgroundColor: 'purple'}]} />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  label: string;
  values: string[];
  selectedValue: string;
  setSelectedValue: (value: string) => void;
}>;

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}: PreviewLayoutProps) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexWrap: 'wrap',
    marginTop: 8,
    backgroundColor: 'aliceblue',
    maxHeight: 400,
  },
  box: {
    width: 50,
    height: 80,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default AlignContentLayout;
```

</TabItem>
</Tabs>

## 彈性換行 (Flex Wrap)

The [`flexWrap`](layout-props#flexwrap) property is set on containers and it controls what happens when children overflow the size of the container along the main axis. By default, children are forced into a single line (which can shrink elements). If wrapping is allowed, items are wrapped into multiple lines along the main axis if needed.

當啟用換行時，可以使用 `alignContent` 來指定行在容器中的排列方式。了解更多請參閱[這裡](https://www.yogalayout.dev/docs/styling/flex-wrap)。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Flex%20Wrap&ext=js
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';

const FlexWrapLayout = () => {
  const [flexWrap, setFlexWrap] = useState('wrap');

  return (
    <PreviewLayout
      label="flexWrap"
      selectedValue={flexWrap}
      values={['wrap', 'nowrap']}
      setSelectedValue={setFlexWrap}>
      <View style={[styles.box, {backgroundColor: 'orangered'}]} />
      <View style={[styles.box, {backgroundColor: 'orange'}]} />
      <View style={[styles.box, {backgroundColor: 'mediumseagreen'}]} />
      <View style={[styles.box, {backgroundColor: 'deepskyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'mediumturquoise'}]} />
      <View style={[styles.box, {backgroundColor: 'mediumslateblue'}]} />
      <View style={[styles.box, {backgroundColor: 'purple'}]} />
    </PreviewLayout>
  );
};

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
    maxHeight: 400,
  },
  box: {
    width: 50,
    height: 80,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default FlexWrapLayout;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Flex%20Wrap&ext=tsx
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';
import type {PropsWithChildren} from 'react';

const FlexWrapLayout = () => {
  const [flexWrap, setFlexWrap] = useState('wrap');

  return (
    <PreviewLayout
      label="flexWrap"
      selectedValue={flexWrap}
      values={['wrap', 'nowrap']}
      setSelectedValue={setFlexWrap}>
      <View style={[styles.box, {backgroundColor: 'orangered'}]} />
      <View style={[styles.box, {backgroundColor: 'orange'}]} />
      <View style={[styles.box, {backgroundColor: 'mediumseagreen'}]} />
      <View style={[styles.box, {backgroundColor: 'deepskyblue'}]} />
      <View style={[styles.box, {backgroundColor: 'mediumturquoise'}]} />
      <View style={[styles.box, {backgroundColor: 'mediumslateblue'}]} />
      <View style={[styles.box, {backgroundColor: 'purple'}]} />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  label: string;
  values: string[];
  selectedValue: string;
  setSelectedValue: (value: string) => void;
}>;

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}: PreviewLayoutProps) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={[styles.container, {[label]: selectedValue}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
    maxHeight: 400,
  },
  box: {
    width: 50,
    height: 80,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default FlexWrapLayout;
```

</TabItem>
</Tabs>

## Flex Basis、Grow 與 Shrink

- [`flexBasis`](layout-props#flexbasis) is an axis-independent way of providing the default size of an item along the main axis. Setting the `flexBasis` of a child is similar to setting the `width` of that child if its parent is a container with `flexDirection: row` or setting the `height` of a child if its parent is a container with `flexDirection: column`. The `flexBasis` of an item is the default size of that item, the size of the item before any `flexGrow` and `flexShrink` calculations are performed.

- [`flexGrow`](layout-props#flexgrow) describes how much space within a container should be distributed among its children along the main axis. After laying out its children, a container will distribute any remaining space according to the flex grow values specified by its children.

  `flexGrow` accepts any floating point value >= 0, with 0 being the default value. A container will distribute any remaining space among its children weighted by the children’s `flexGrow` values.

- [`flexShrink`](layout-props#flexshrink) describes how to shrink children along the main axis in the case in which the total size of the children overflows the size of the container on the main axis. `flexShrink` is very similar to `flexGrow` and can be thought of in the same way if any overflowing size is considered to be negative remaining space. These two properties also work well together by allowing children to grow and shrink as needed.

  `flexShrink` accepts any floating point value >= 0, with 0 being the default value (on the web, the default is 1). A container will shrink its children weighted by the children’s `flexShrink` values.

了解更多請參閱[這裡](https://www.yogalayout.dev/docs/styling/flex-basis-grow-shrink)。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Flex%20Basis%2C%20Grow%2C%20and%20Shrink&ext=js
import React, {useState} from 'react';
import {View, Text, TextInput, StyleSheet} from 'react-native';

const App = () => {
  const [powderblue, setPowderblue] = useState({
    flexGrow: 0,
    flexShrink: 1,
    flexBasis: 'auto',
  });
  const [skyblue, setSkyblue] = useState({
    flexGrow: 1,
    flexShrink: 0,
    flexBasis: 100,
  });
  const [steelblue, setSteelblue] = useState({
    flexGrow: 0,
    flexShrink: 1,
    flexBasis: 200,
  });
  return (
    <View style={styles.container}>
      <View
        style={[
          styles.container,
          {
            flexDirection: 'row',
            alignContent: 'space-between',
          },
        ]}>
        <BoxInfo color="powderblue" {...powderblue} setStyle={setPowderblue} />
        <BoxInfo color="skyblue" {...skyblue} setStyle={setSkyblue} />
        <BoxInfo color="steelblue" {...steelblue} setStyle={setSteelblue} />
      </View>
      <View style={styles.previewContainer}>
        <View
          style={[
            styles.box,
            {
              flexBasis: powderblue.flexBasis,
              flexGrow: powderblue.flexGrow,
              flexShrink: powderblue.flexShrink,
              backgroundColor: 'powderblue',
            },
          ]}
        />
        <View
          style={[
            styles.box,
            {
              flexBasis: skyblue.flexBasis,
              flexGrow: skyblue.flexGrow,
              flexShrink: skyblue.flexShrink,
              backgroundColor: 'skyblue',
            },
          ]}
        />
        <View
          style={[
            styles.box,
            {
              flexBasis: steelblue.flexBasis,
              flexGrow: steelblue.flexGrow,
              flexShrink: steelblue.flexShrink,
              backgroundColor: 'steelblue',
            },
          ]}
        />
      </View>
    </View>
  );
};

const BoxInfo = ({color, flexBasis, flexShrink, setStyle, flexGrow}) => (
  <View style={[styles.row, {flexDirection: 'column'}]}>
    <View
      style={[
        styles.boxLabel,
        {
          backgroundColor: color,
        },
      ]}>
      <Text
        style={{
          color: '#fff',
          fontWeight: '500',
          textAlign: 'center',
        }}>
        Box
      </Text>
    </View>
    <Text style={styles.label}>flexBasis</Text>
    <TextInput
      value={flexBasis}
      style={styles.input}
      onChangeText={fB =>
        setStyle(value => ({
          ...value,
          flexBasis: isNaN(parseInt(fB, 10)) ? 'auto' : parseInt(fB, 10),
        }))
      }
    />
    <Text style={styles.label}>flexShrink</Text>
    <TextInput
      value={flexShrink}
      style={styles.input}
      onChangeText={fS =>
        setStyle(value => ({
          ...value,
          flexShrink: isNaN(parseInt(fS, 10)) ? undefined : parseInt(fS, 10),
        }))
      }
    />
    <Text style={styles.label}>flexGrow</Text>
    <TextInput
      value={flexGrow}
      style={styles.input}
      onChangeText={fG =>
        setStyle(value => ({
          ...value,
          flexGrow: isNaN(parseInt(fG, 10)) ? undefined : parseInt(fG, 10),
        }))
      }
    />
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingHorizontal: 10,
  },
  box: {
    flex: 1,
    height: 50,
    width: 50,
  },
  boxLabel: {
    minWidth: 80,
    padding: 8,
    borderRadius: 4,
    marginTop: 8,
  },
  label: {
    marginTop: 6,
    fontSize: 16,
    fontWeight: '100',
  },
  previewContainer: {
    flex: 1,
    flexDirection: 'row',
    backgroundColor: 'aliceblue',
  },
  row: {
    flex: 1,
    flexDirection: 'row',
    flexWrap: 'wrap',
    alignItems: 'center',
    marginBottom: 10,
  },
  input: {
    borderBottomWidth: 1,
    paddingVertical: 3,
    width: 50,
    textAlign: 'center',
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Flex%20Basis%2C%20Grow%2C%20and%20Shrink&ext=tsx
import React, {useState} from 'react';
import {View, Text, TextInput, StyleSheet} from 'react-native';
import type {ViewStyle} from 'react-native';

const App = () => {
  const [powderblue, setPowderblue] = useState<ViewStyle>({
    flexGrow: 0,
    flexShrink: 1,
    flexBasis: 'auto',
  });
  const [skyblue, setSkyblue] = useState<ViewStyle>({
    flexGrow: 1,
    flexShrink: 0,
    flexBasis: 100,
  });
  const [steelblue, setSteelblue] = useState<ViewStyle>({
    flexGrow: 0,
    flexShrink: 1,
    flexBasis: 200,
  });
  return (
    <View style={styles.container}>
      <View
        style={[
          styles.container,
          {
            flexDirection: 'row',
            alignContent: 'space-between',
          },
        ]}>
        <BoxInfo color="powderblue" {...powderblue} setStyle={setPowderblue} />
        <BoxInfo color="skyblue" {...skyblue} setStyle={setSkyblue} />
        <BoxInfo color="steelblue" {...steelblue} setStyle={setSteelblue} />
      </View>
      <View style={styles.previewContainer}>
        <View
          style={[
            styles.box,
            {
              flexBasis: powderblue.flexBasis,
              flexGrow: powderblue.flexGrow,
              flexShrink: powderblue.flexShrink,
              backgroundColor: 'powderblue',
            },
          ]}
        />
        <View
          style={[
            styles.box,
            {
              flexBasis: skyblue.flexBasis,
              flexGrow: skyblue.flexGrow,
              flexShrink: skyblue.flexShrink,
              backgroundColor: 'skyblue',
            },
          ]}
        />
        <View
          style={[
            styles.box,
            {
              flexBasis: steelblue.flexBasis,
              flexGrow: steelblue.flexGrow,
              flexShrink: steelblue.flexShrink,
              backgroundColor: 'steelblue',
            },
          ]}
        />
      </View>
    </View>
  );
};

type BoxInfoProps = ViewStyle & {
  color: string;
  setStyle: React.Dispatch<React.SetStateAction<ViewStyle>>;
};

const BoxInfo = ({
  color,
  flexBasis,
  flexShrink,
  setStyle,
  flexGrow,
}: BoxInfoProps) => (
  <View style={[styles.row, {flexDirection: 'column'}]}>
    <View
      style={[
        styles.boxLabel,
        {
          backgroundColor: color,
        },
      ]}>
      <Text
        style={{
          color: '#fff',
          fontWeight: '500',
          textAlign: 'center',
        }}>
        Box
      </Text>
    </View>
    <Text style={styles.label}>flexBasis</Text>
    <TextInput
      value={String(flexBasis)}
      style={styles.input}
      onChangeText={fB =>
        setStyle(value => ({
          ...value,
          flexBasis: isNaN(parseInt(fB, 10)) ? 'auto' : parseInt(fB, 10),
        }))
      }
    />
    <Text style={styles.label}>flexShrink</Text>
    <TextInput
      value={String(flexShrink)}
      style={styles.input}
      onChangeText={fS =>
        setStyle(value => ({
          ...value,
          flexShrink: isNaN(parseInt(fS, 10)) ? undefined : parseInt(fS, 10),
        }))
      }
    />
    <Text style={styles.label}>flexGrow</Text>
    <TextInput
      value={String(flexGrow)}
      style={styles.input}
      onChangeText={fG =>
        setStyle(value => ({
          ...value,
          flexGrow: isNaN(parseInt(fG, 10)) ? undefined : parseInt(fG, 10),
        }))
      }
    />
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingHorizontal: 10,
  },
  box: {
    flex: 1,
    height: 50,
    width: 50,
  },
  boxLabel: {
    minWidth: 80,
    padding: 8,
    borderRadius: 4,
    marginTop: 8,
  },
  label: {
    marginTop: 6,
    fontSize: 16,
    fontWeight: '100',
  },
  previewContainer: {
    flex: 1,
    flexDirection: 'row',
    backgroundColor: 'aliceblue',
  },
  row: {
    flex: 1,
    flexDirection: 'row',
    flexWrap: 'wrap',
    alignItems: 'center',
    marginBottom: 10,
  },
  input: {
    borderBottomWidth: 1,
    paddingVertical: 3,
    width: 50,
    textAlign: 'center',
  },
});

export default App;
```

</TabItem>
</Tabs>

## 行間距、列間距與間距

- [`rowGap`](layout-props#rowgap) 設定元素行與行之間的間距（溝槽）大小。
- [`columnGap`](layout-props#columngap) 設定元素列與列之間的間距（溝槽）大小。
- [`gap`](layout-props#gap) 設定行與列之間的間距（溝槽）大小。它是 `rowGap` 和 `columnGap` 的簡寫形式。

您可以結合使用 `flexWrap`、`alignContent` 和 `gap` 來為項目之間添加一致的間距。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Row%20Gap%20and%20Column%20Gap&ext=js
import React, {useState} from 'react';
import {View, Text, StyleSheet, TextInput} from 'react-native';

const RowGapAndColumnGap = () => {
  const [rowGap, setRowGap] = useState(10);
  const [columnGap, setColumnGap] = useState(10);

  return (
    <PreviewLayout
      columnGap={columnGap}
      handleColumnGapChange={setColumnGap}
      rowGap={rowGap}
      handleRowGapChange={setRowGap}>
      <View style={[styles.box, styles.box1]} />
      <View style={[styles.box, styles.box2]} />
      <View style={[styles.box, styles.box3]} />
      <View style={[styles.box, styles.box4]} />
      <View style={[styles.box, styles.box5]} />
    </PreviewLayout>
  );
};

const PreviewLayout = ({
  children,
  handleColumnGapChange,
  handleRowGapChange,
  rowGap,
  columnGap,
}) => (
  <View style={styles.previewContainer}>
    <View style={styles.inputContainer}>
      <View style={styles.itemsCenter}>
        <Text>Row Gap</Text>
        <TextInput
          style={styles.input}
          value={rowGap}
          onChangeText={v => handleRowGapChange(Number(v))}
        />
      </View>
      <View style={styles.itemsCenter}>
        <Text>Column Gap</Text>
        <TextInput
          style={styles.input}
          value={columnGap}
          onChangeText={v => handleColumnGapChange(Number(v))}
        />
      </View>
    </View>
    <View style={[styles.container, {rowGap, columnGap}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  itemsCenter: {alignItems: 'center'},
  inputContainer: {
    gap: 4,
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  previewContainer: {padding: 10, flex: 1},
  input: {
    borderBottomWidth: 1,
    paddingVertical: 3,
    width: 50,
    textAlign: 'center',
  },
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
    maxHeight: 400,
    flexWrap: 'wrap',
    alignContent: 'flex-start',
  },
  box: {
    width: 50,
    height: 80,
  },
  box1: {
    backgroundColor: 'orangered',
  },
  box2: {
    backgroundColor: 'orange',
  },
  box3: {
    backgroundColor: 'mediumseagreen',
  },
  box4: {
    backgroundColor: 'deepskyblue',
  },
  box5: {
    backgroundColor: 'mediumturquoise',
  },
});

export default RowGapAndColumnGap;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Row%20Gap%20and%20Column%20Gap&ext=tsx
import React, {useState} from 'react';
import {View, Text, StyleSheet, TextInput} from 'react-native';
import type {PropsWithChildren} from 'react';

const RowGapAndColumnGap = () => {
  const [rowGap, setRowGap] = useState(10);
  const [columnGap, setColumnGap] = useState(10);

  return (
    <PreviewLayout
      columnGap={columnGap}
      handleColumnGapChange={setColumnGap}
      rowGap={rowGap}
      handleRowGapChange={setRowGap}>
      <View style={[styles.box, styles.box1]} />
      <View style={[styles.box, styles.box2]} />
      <View style={[styles.box, styles.box3]} />
      <View style={[styles.box, styles.box4]} />
      <View style={[styles.box, styles.box5]} />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  columnGap: number;
  handleColumnGapChange: (gap: number) => void;
  rowGap: number;
  handleRowGapChange: (gap: number) => void;
}>;

const PreviewLayout = ({
  children,
  handleColumnGapChange,
  handleRowGapChange,
  rowGap,
  columnGap,
}: PreviewLayoutProps) => (
  <View style={styles.previewContainer}>
    <View style={styles.inputContainer}>
      <View style={styles.itemsCenter}>
        <Text>Row Gap</Text>
        <TextInput
          style={styles.input}
          value={String(rowGap)}
          onChangeText={v => handleRowGapChange(Number(v))}
        />
      </View>
      <View style={styles.itemsCenter}>
        <Text>Column Gap</Text>
        <TextInput
          style={styles.input}
          value={String(columnGap)}
          onChangeText={v => handleColumnGapChange(Number(v))}
        />
      </View>
    </View>
    <View style={[styles.container, {rowGap, columnGap}]}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  itemsCenter: {alignItems: 'center'},
  inputContainer: {
    gap: 4,
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  previewContainer: {padding: 10, flex: 1},
  input: {
    borderBottomWidth: 1,
    paddingVertical: 3,
    width: 50,
    textAlign: 'center',
  },
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
    maxHeight: 400,
    flexWrap: 'wrap',
    alignContent: 'flex-start',
  },
  box: {
    width: 50,
    height: 80,
  },
  box1: {
    backgroundColor: 'orangered',
  },
  box2: {
    backgroundColor: 'orange',
  },
  box3: {
    backgroundColor: 'mediumseagreen',
  },
  box4: {
    backgroundColor: 'deepskyblue',
  },
  box5: {
    backgroundColor: 'mediumturquoise',
  },
});

export default RowGapAndColumnGap;
```

</TabItem>
</Tabs>

## 寬度與高度

The `width` property specifies the width of an element's content area. Similarly, the `height` property specifies the height of an element's content area.

Both `width` and `height` can take the following values:

- `auto`（**預設值**）React Native 會根據元素的內容（無論是其他子元素、文字還是圖片）計算其寬度/高度。
- `pixels` 以絕對像素定義寬度/高度。根據組件上設置的其他樣式，這可能不會是最終節點的實際尺寸。
- `percentage` 以父元素寬度或高度的百分比定義寬度或高度。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Width%20and%20Height&ext=js
import React, {useState} from 'react';
import {
  View,
  SafeAreaView,
  TouchableOpacity,
  Text,
  StyleSheet,
} from 'react-native';

const WidthHeightBasics = () => {
  const [widthType, setWidthType] = useState('auto');
  const [heightType, setHeightType] = useState('auto');

  return (
    <PreviewLayout
      widthType={widthType}
      heightType={heightType}
      widthValues={['auto', 300, '80%']}
      heightValues={['auto', 200, '60%']}
      setWidthType={setWidthType}
      setHeightType={setHeightType}>
      <View
        style={{
          alignSelf: 'flex-start',
          backgroundColor: 'aliceblue',
          height: heightType,
          width: widthType,
          padding: 15,
        }}>
        <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
        <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
        <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
      </View>
    </PreviewLayout>
  );
};

const PreviewLayout = ({
  children,
  widthType,
  heightType,
  widthValues,
  heightValues,
  setWidthType,
  setHeightType,
}) => (
  <SafeAreaView style={{flex: 1, padding: 10}}>
    <View style={styles.row}>
      <Text style={styles.label}>width </Text>
      {widthValues.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setWidthType(value)}
          style={[styles.button, widthType === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              widthType === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={styles.row}>
      <Text style={styles.label}>height </Text>
      {heightValues.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setHeightType(value)}
          style={[styles.button, heightType === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              heightType === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    {children}
  </SafeAreaView>
);

const styles = StyleSheet.create({
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    padding: 8,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginRight: 10,
    marginBottom: 10,
  },
  selected: {
    backgroundColor: 'coral',
    shadowOpacity: 0,
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default WidthHeightBasics;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Width%20and%20Height&ext=tsx
import React, {useState} from 'react';
import {
  View,
  SafeAreaView,
  TouchableOpacity,
  Text,
  StyleSheet,
} from 'react-native';
import type {PropsWithChildren} from 'react';

type Dimension = 'auto' | `${number}%` | number;

const WidthHeightBasics = () => {
  const [widthType, setWidthType] = useState<Dimension>('auto');
  const [heightType, setHeightType] = useState<Dimension>('auto');

  return (
    <PreviewLayout
      widthType={widthType}
      heightType={heightType}
      widthValues={['auto', 300, '80%']}
      heightValues={['auto', 200, '60%']}
      setWidthType={setWidthType}
      setHeightType={setHeightType}>
      <View
        style={{
          alignSelf: 'flex-start',
          backgroundColor: 'aliceblue',
          height: heightType,
          width: widthType,
          padding: 15,
        }}>
        <View style={[styles.box, {backgroundColor: 'powderblue'}]} />
        <View style={[styles.box, {backgroundColor: 'skyblue'}]} />
        <View style={[styles.box, {backgroundColor: 'steelblue'}]} />
      </View>
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  widthType: Dimension;
  heightType: Dimension;
  widthValues: Dimension[];
  heightValues: Dimension[];
  setWidthType: (value: Dimension) => void;
  setHeightType: (value: Dimension) => void;
}>;

const PreviewLayout = ({
  children,
  widthType,
  heightType,
  widthValues,
  heightValues,
  setWidthType,
  setHeightType,
}: PreviewLayoutProps) => (
  <SafeAreaView style={{flex: 1, padding: 10}}>
    <View style={styles.row}>
      <Text style={styles.label}>width </Text>
      {widthValues.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setWidthType(value)}
          style={[styles.button, widthType === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              widthType === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={styles.row}>
      <Text style={styles.label}>height </Text>
      {heightValues.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setHeightType(value)}
          style={[styles.button, heightType === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              heightType === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    {children}
  </SafeAreaView>
);

const styles = StyleSheet.create({
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    padding: 8,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginRight: 10,
    marginBottom: 10,
  },
  selected: {
    backgroundColor: 'coral',
    shadowOpacity: 0,
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default WidthHeightBasics;
```

</TabItem>
</Tabs>

## 絕對與相對佈局

元素的 `position` 類型定義了其在父元素內的定位方式。

- `relative`（**預設值**）預設情況下，元素會以相對方式定位。這表示元素會根據佈局的正常流動進行定位，然後根據 `top`、`right`、`bottom` 和 `left` 的值相對於該位置進行偏移。此偏移不會影響任何兄弟元素或父元素的位置。

- `absolute` 當元素以絕對方式定位時，它不會參與正常的佈局流動。相反，它會獨立於其兄弟元素進行佈局。其位置是根據 `top`、`right`、`bottom` 和 `left` 的值來決定的。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Absolute%20%26%20Relative%20Layout&ext=js
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';

const PositionLayout = () => {
  const [position, setPosition] = useState('relative');

  return (
    <PreviewLayout
      label="position"
      selectedValue={position}
      values={['relative', 'absolute']}
      setSelectedValue={setPosition}>
      <View
        style={[
          styles.box,
          {
            top: 25,
            left: 25,
            position,
            backgroundColor: 'powderblue',
          },
        ]}
      />
      <View
        style={[
          styles.box,
          {
            top: 50,
            left: 50,
            position,
            backgroundColor: 'skyblue',
          },
        ]}
      />
      <View
        style={[
          styles.box,
          {
            top: 75,
            left: 75,
            position,
            backgroundColor: 'steelblue',
          },
        ]}
      />
    </PreviewLayout>
  );
};

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={styles.container}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
    minHeight: 200,
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default PositionLayout;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Absolute%20%26%20Relative%20Layout&ext=tsx
import React, {useState} from 'react';
import {View, TouchableOpacity, Text, StyleSheet} from 'react-native';
import type {PropsWithChildren} from 'react';

const PositionLayout = () => {
  const [position, setPosition] = useState<'relative' | 'absolute'>('relative');

  return (
    <PreviewLayout
      label="position"
      selectedValue={position}
      values={['relative', 'absolute']}
      setSelectedValue={setPosition}>
      <View
        style={[
          styles.box,
          {
            top: 25,
            left: 25,
            position,
            backgroundColor: 'powderblue',
          },
        ]}
      />
      <View
        style={[
          styles.box,
          {
            top: 50,
            left: 50,
            position,
            backgroundColor: 'skyblue',
          },
        ]}
      />
      <View
        style={[
          styles.box,
          {
            top: 75,
            left: 75,
            position,
            backgroundColor: 'steelblue',
          },
        ]}
      />
    </PreviewLayout>
  );
};

type PreviewLayoutProps = PropsWithChildren<{
  label: string;
  values: Array<'relative' | 'absolute'>;
  selectedValue: string;
  setSelectedValue: (value: 'relative' | 'absolute') => void;
}>;

const PreviewLayout = ({
  label,
  children,
  values,
  selectedValue,
  setSelectedValue,
}: PreviewLayoutProps) => (
  <View style={{padding: 10, flex: 1}}>
    <Text style={styles.label}>{label}</Text>
    <View style={styles.row}>
      {values.map(value => (
        <TouchableOpacity
          key={value}
          onPress={() => setSelectedValue(value)}
          style={[styles.button, selectedValue === value && styles.selected]}>
          <Text
            style={[
              styles.buttonLabel,
              selectedValue === value && styles.selectedLabel,
            ]}>
            {value}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
    <View style={styles.container}>{children}</View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 8,
    backgroundColor: 'aliceblue',
    minHeight: 200,
  },
  box: {
    width: 50,
    height: 50,
  },
  row: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  button: {
    paddingHorizontal: 8,
    paddingVertical: 6,
    borderRadius: 4,
    backgroundColor: 'oldlace',
    alignSelf: 'flex-start',
    marginHorizontal: '1%',
    marginBottom: 6,
    minWidth: '48%',
    textAlign: 'center',
  },
  selected: {
    backgroundColor: 'coral',
    borderWidth: 0,
  },
  buttonLabel: {
    fontSize: 12,
    fontWeight: '500',
    color: 'coral',
  },
  selectedLabel: {
    color: 'white',
  },
  label: {
    textAlign: 'center',
    marginBottom: 10,
    fontSize: 24,
  },
});

export default PositionLayout;
```

</TabItem>
</Tabs>

## 深入探討

查看互動式的 [yoga playground](https://www.yogalayout.dev/playground)，它可以幫助你更好地理解 flexbox。

我們已經涵蓋了基礎知識，但還有許多其他你可能需要用於佈局的樣式。控制佈局的完整屬性列表記錄在[這裡](./layout-props.md)。

此外，你也可以參考 [Wix Engineers](https://medium.com/wix-engineering/the-full-react-native-layout-cheat-sheet-a4147802405c) 提供的一些範例。