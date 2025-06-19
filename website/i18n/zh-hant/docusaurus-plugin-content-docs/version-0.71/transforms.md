---
id: transforms
title: Transforms
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

變換(Transforms)是能幫助您使用2D或3D轉換來修改元件外觀與位置的樣式屬性。但需注意，當套用變換後，元件周圍的佈局會保持不變，這可能導致變換後的元件與鄰近元件發生重疊。您可以透過為變換元件添加邊距(margin)、調整鄰近元件間距，或為容器設置內邊距(padding)來避免此類重疊問題。

## 範例

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Transforms
import React from 'react';
import {SafeAreaView, ScrollView, StyleSheet, Text, View} from 'react-native';

const App = () => (
  <SafeAreaView style={styles.container}>
    <ScrollView contentContainerStyle={styles.scrollContentContainer}>
      <View style={styles.box}>
        <Text style={styles.text}>Original Object</Text>
      </View>

      <View
        style={[
          styles.box,
          {
            transform: [{scale: 2}],
          },
        ]}>
        <Text style={styles.text}>Scale by 2</Text>
      </View>

      <View
        style={[
          styles.box,
          {
            transform: [{scaleX: 2}],
          },
        ]}>
        <Text style={styles.text}>ScaleX by 2</Text>
      </View>

      <View
        style={[
          styles.box,
          {
            transform: [{scaleY: 2}],
          },
        ]}>
        <Text style={styles.text}>ScaleY by 2</Text>
      </View>

      <View
        style={[
          styles.box,
          {
            transform: [{rotate: '45deg'}],
          },
        ]}>
        <Text style={styles.text}>Rotate by 45 deg</Text>
      </View>

      <View
        style={[
          styles.box,
          {
            transform: [{rotateX: '45deg'}, {rotateZ: '45deg'}],
          },
        ]}>
        <Text style={styles.text}>Rotate X&Z by 45 deg</Text>
      </View>

      <View
        style={[
          styles.box,
          {
            transform: [{rotateY: '45deg'}, {rotateZ: '45deg'}],
          },
        ]}>
        <Text style={styles.text}>Rotate Y&Z by 45 deg</Text>
      </View>

      <View
        style={[
          styles.box,
          {
            transform: [{skewX: '45deg'}],
          },
        ]}>
        <Text style={styles.text}>SkewX by 45 deg</Text>
      </View>

      <View
        style={[
          styles.box,
          {
            transform: [{skewY: '45deg'}],
          },
        ]}>
        <Text style={styles.text}>SkewY by 45 deg</Text>
      </View>

      <View
        style={[
          styles.box,
          {
            transform: [{skewX: '30deg'}, {skewY: '30deg'}],
          },
        ]}>
        <Text style={styles.text}>Skew X&Y by 30 deg</Text>
      </View>

      <View
        style={[
          styles.box,
          {
            transform: [{translateX: -50}],
          },
        ]}>
        <Text style={styles.text}>TranslateX by -50 </Text>
      </View>

      <View
        style={[
          styles.box,
          {
            transform: [{translateY: 50}],
          },
        ]}>
        <Text style={styles.text}>TranslateY by 50 </Text>
      </View>
    </ScrollView>
  </SafeAreaView>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  scrollContentContainer: {
    alignItems: 'center',
    paddingBottom: 60,
  },
  box: {
    height: 100,
    width: 100,
    borderRadius: 5,
    marginVertical: 40,
    backgroundColor: '#61dafb',
    alignItems: 'center',
    justifyContent: 'center',
  },
  text: {
    fontSize: 14,
    fontWeight: 'bold',
    margin: 8,
    color: '#000',
    textAlign: 'center',
  },
});

export default App;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=Transforms
import React, {Component} from 'react';
import {SafeAreaView, ScrollView, StyleSheet, Text, View} from 'react-native';

class App extends Component {
  render() {
    return (
      <SafeAreaView style={styles.container}>
        <ScrollView contentContainerStyle={styles.scrollContentContainer}>
          <View style={styles.box}>
            <Text style={styles.text}>Original Object</Text>
          </View>

          <View
            style={[
              styles.box,
              {
                transform: [{scale: 2}],
              },
            ]}>
            <Text style={styles.text}>Scale by 2</Text>
          </View>

          <View
            style={[
              styles.box,
              {
                transform: [{scaleX: 2}],
              },
            ]}>
            <Text style={styles.text}>ScaleX by 2</Text>
          </View>

          <View
            style={[
              styles.box,
              {
                transform: [{scaleY: 2}],
              },
            ]}>
            <Text style={styles.text}>ScaleY by 2</Text>
          </View>

          <View
            style={[
              styles.box,
              {
                transform: [{rotate: '45deg'}],
              },
            ]}>
            <Text style={styles.text}>Rotate by 45 deg</Text>
          </View>

          <View
            style={[
              styles.box,
              {
                transform: [{rotateX: '45deg'}, {rotateZ: '45deg'}],
              },
            ]}>
            <Text style={styles.text}>Rotate X&Z by 45 deg</Text>
          </View>

          <View
            style={[
              styles.box,
              {
                transform: [{rotateY: '45deg'}, {rotateZ: '45deg'}],
              },
            ]}>
            <Text style={styles.text}>Rotate Y&Z by 45 deg</Text>
          </View>

          <View
            style={[
              styles.box,
              {
                transform: [{skewX: '45deg'}],
              },
            ]}>
            <Text style={styles.text}>SkewX by 45 deg</Text>
          </View>

          <View
            style={[
              styles.box,
              {
                transform: [{skewY: '45deg'}],
              },
            ]}>
            <Text style={styles.text}>SkewY by 45 deg</Text>
          </View>

          <View
            style={[
              styles.box,
              {
                transform: [{skewX: '30deg'}, {skewY: '30deg'}],
              },
            ]}>
            <Text style={styles.text}>Skew X&Y by 30 deg</Text>
          </View>

          <View
            style={[
              styles.box,
              {
                transform: [{translateX: -50}],
              },
            ]}>
            <Text style={styles.text}>TranslateX by -50</Text>
          </View>

          <View
            style={[
              styles.box,
              {
                transform: [{translateY: 50}],
              },
            ]}>
            <Text style={styles.text}>TranslateY by 50</Text>
          </View>
        </ScrollView>
      </SafeAreaView>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  scrollContentContainer: {
    alignItems: 'center',
    paddingBottom: 60,
  },
  box: {
    height: 100,
    width: 100,
    borderRadius: 5,
    marginVertical: 40,
    backgroundColor: '#61dafb',
    alignItems: 'center',
    justifyContent: 'center',
  },
  text: {
    fontSize: 14,
    fontWeight: 'bold',
    margin: 8,
    color: '#000',
    textAlign: 'center',
  },
});

export default App;
```

</TabItem>
</Tabs>

---

# 參考文獻

## 變換(Transform)

`transform` accepts an array of transformation objects or space-separated string values. Each object specifies the property that will be transformed as the key, and the value to use in the transformation. Objects should not be combined. Use a single key/value pair per object.

The rotate transformations require a string so that the transform may be expressed in degrees (deg) or radians (rad). For example:

```js
{
  transform: [{rotateX: '45deg'}, {rotateZ: '0.785398rad'}],
}
```

相同的效果亦可透過空格分隔的字串達成：

```js
{
  transform: 'rotateX(45deg) rotateZ(0.785398rad)',
}
```

傾斜(skew)變換同樣需使用字串指定度數(deg)。例如：

```js
{
  transform: [{skewX: '45deg'}],
}
```

| Type                                                                                                                                                                                                                                                                                                          | Required |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| array of objects: `{matrix: number[]}`, `{perspective: number}`, `{rotate: string}`, `{rotateX: string}`, `{rotateY: string}`, `{rotateZ: string}`, `{scale: number}`, `{scaleX: number}`, `{scaleY: number}`, `{translateX: number}`, `{translateY: number}`, `{skewX: string}`, `{skewY: string}` or string | No       |

---

### `decomposedMatrix`, `rotation`, `scaleX`, `scaleY`, `transformMatrix`, `translateX`, `translateY`

> **已棄用。** 請改用[`transform`](transforms#transform)屬性。