---
id: refreshcontrol
title: RefreshControl
---

此元件用於 ScrollView 或 ListView 內部，以新增下拉刷新功能。當 ScrollView 處於 `scrollY: 0` 時，向下滑動會觸發 `onRefresh` 事件。

## 範例

```SnackPlayer name=RefreshControl&supportedPlatforms=ios,android
import React from 'react';
import {
  RefreshControl,
  SafeAreaView,
  ScrollView,
  StyleSheet,
  Text,
} from 'react-native';

const App = () => {
  const [refreshing, setRefreshing] = React.useState(false);

  const onRefresh = React.useCallback(() => {
    setRefreshing(true);
    setTimeout(() => {
      setRefreshing(false);
    }, 2000);
  }, []);

  return (
    <SafeAreaView style={styles.container}>
      <ScrollView
        contentContainerStyle={styles.scrollView}
        refreshControl={
          <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
        }>
        <Text>Pull down to see RefreshControl indicator</Text>
      </ScrollView>
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  scrollView: {
    flex: 1,
    backgroundColor: 'pink',
    alignItems: 'center',
    justifyContent: 'center',
  },
});

export default App;
```

> Note: `refreshing` is a controlled prop, this is why it needs to be set to `true` in the `onRefresh` function otherwise the refresh indicator will stop immediately.

---

# 參考

## 屬性

### [View 屬性](view.md#props)

繼承 [View 屬性](view.md#props)。

---

### <div class="label required basic">Required</div>**`refreshing`**

是否應顯示正在刷新的指示。

| Type    |
| ------- |
| boolean |

---

### `colors` <div class="label android">Android</div>

用於繪製刷新指示器的顏色（至少一個）。

| Type                         |
| ---------------------------- |
| array of [colors](colors.md) |

---

### `enabled` <div class="label android">Android</div>

是否啟用下拉刷新功能。

| Type    | Default |
| ------- | ------- |
| boolean | `true`  |

---

### `onRefresh`

當視圖開始刷新時呼叫。

| Type     |
| -------- |
| function |

---

### `progressBackgroundColor` <div class="label android">Android</div>

刷新指示器的背景顏色。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `progressViewOffset`

進度視圖的頂部偏移量。

| Type   | Default |
| ------ | ------- |
| number | `0`     |

---

### `size` <div class="label android">Android</div>

刷新指示器的大小。

| Type                         | Default     |
| ---------------------------- | ----------- |
| enum(`'default'`, `'large'`) | `'default'` |

---

### `tintColor` <div class="label ios">iOS</div>

刷新指示器的顏色。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `title` <div class="label ios">iOS</div>

顯示在刷新指示器下方的標題。

| Type   |
| ------ |
| string |

---

### `titleColor` <div class="label ios">iOS</div>

刷新指示器標題的顏色。

| Type               |
| ------------------ |
| [color](colors.md) |