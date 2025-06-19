---
id: drawerlayoutandroid
title: DrawerLayoutAndroid
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

封裝平台原生 `DrawerLayout` 的 React 元件（僅限 Android）。抽屜式導航欄透過 `renderNavigationView` 渲染，而直接子元件則作為主視圖（放置內容區域）。導航視圖初始狀態隱藏於螢幕側邊，可透過 `drawerPosition` 屬性指定滑入方向，並以 `drawerWidth` 屬性設定其寬度。

## 範例

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=DrawerLayoutAndroid%20Component%20Example&supportedPlatforms=android&ext=js
import React, {useRef, useState} from 'react';
import {
  Button,
  DrawerLayoutAndroid,
  Text,
  StyleSheet,
  View,
} from 'react-native';

const App = () => {
  const drawer = useRef(null);
  const [drawerPosition, setDrawerPosition] = useState('left');
  const changeDrawerPosition = () => {
    if (drawerPosition === 'left') {
      setDrawerPosition('right');
    } else {
      setDrawerPosition('left');
    }
  };

  const navigationView = () => (
    <View style={[styles.container, styles.navigationContainer]}>
      <Text style={styles.paragraph}>I'm in the Drawer!</Text>
      <Button
        title="Close drawer"
        onPress={() => drawer.current.closeDrawer()}
      />
    </View>
  );

  return (
    <DrawerLayoutAndroid
      ref={drawer}
      drawerWidth={300}
      drawerPosition={drawerPosition}
      renderNavigationView={navigationView}>
      <View style={styles.container}>
        <Text style={styles.paragraph}>Drawer on the {drawerPosition}!</Text>
        <Button
          title="Change Drawer Position"
          onPress={() => changeDrawerPosition()}
        />
        <Text style={styles.paragraph}>
          Swipe from the side or press button below to see it!
        </Text>
        <Button
          title="Open drawer"
          onPress={() => drawer.current.openDrawer()}
        />
      </View>
    </DrawerLayoutAndroid>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: 16,
  },
  navigationContainer: {
    backgroundColor: '#ecf0f1',
  },
  paragraph: {
    padding: 16,
    fontSize: 15,
    textAlign: 'center',
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=DrawerLayoutAndroid%20Component%20Example&supportedPlatforms=android&ext=tsx
import React, {useRef, useState} from 'react';
import {
  Button,
  DrawerLayoutAndroid,
  Text,
  StyleSheet,
  View,
} from 'react-native';

const App = () => {
  const drawer = useRef<DrawerLayoutAndroid>(null);
  const [drawerPosition, setDrawerPosition] = useState<'left' | 'right'>(
    'left',
  );
  const changeDrawerPosition = () => {
    if (drawerPosition === 'left') {
      setDrawerPosition('right');
    } else {
      setDrawerPosition('left');
    }
  };

  const navigationView = () => (
    <View style={[styles.container, styles.navigationContainer]}>
      <Text style={styles.paragraph}>I'm in the Drawer!</Text>
      <Button
        title="Close drawer"
        onPress={() => drawer.current?.closeDrawer()}
      />
    </View>
  );

  return (
    <DrawerLayoutAndroid
      ref={drawer}
      drawerWidth={300}
      drawerPosition={drawerPosition}
      renderNavigationView={navigationView}>
      <View style={styles.container}>
        <Text style={styles.paragraph}>Drawer on the {drawerPosition}!</Text>
        <Button
          title="Change Drawer Position"
          onPress={() => changeDrawerPosition()}
        />
        <Text style={styles.paragraph}>
          Swipe from the side or press button below to see it!
        </Text>
        <Button
          title="Open drawer"
          onPress={() => drawer.current?.openDrawer()}
        />
      </View>
    </DrawerLayoutAndroid>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: 16,
  },
  navigationContainer: {
    backgroundColor: '#ecf0f1',
  },
  paragraph: {
    padding: 16,
    fontSize: 15,
    textAlign: 'center',
  },
});

export default App;
```

</TabItem>
</Tabs>

---

# 參考文件

## 屬性

### [View 屬性](view.md#props)

繼承 [View 元件所有屬性](view.md#props)。

---

### `drawerBackgroundColor`

設定抽屜背景色，預設值為 `white`。若需設定透明度，請使用 rgba 格式。範例：

```tsx
return (
  <DrawerLayoutAndroid drawerBackgroundColor="rgba(0,0,0,0.5)" />
);
```

| Type               | Required |
| ------------------ | -------- |
| [color](colors.md) | No       |

---

### `drawerLockMode`

設定抽屜鎖定模式，共有三種狀態：

- unlocked（預設值）：抽屜會響應觸控手勢（開啟/關閉）
- locked-closed：抽屜保持關閉且不響應手勢
- locked-open：抽屜保持開啟且不響應手勢（仍可透過程式呼叫 `openDrawer`/`closeDrawer` 控制）

| Type                                             | Required |
| ------------------------------------------------ | -------- |
| enum('unlocked', 'locked-closed', 'locked-open') | No       |

---

### `drawerPosition`

指定抽屜滑入的螢幕側邊，預設值為 `left`。

| Type                  | Required |
| --------------------- | -------- |
| enum('left', 'right') | No       |

---

### `drawerWidth`

設定抽屜寬度，即從視窗邊緣拉出的視圖寬度。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `keyboardDismissMode`

決定拖曳時是否關閉鍵盤：

- 'none'（預設值）：拖曳時不關閉鍵盤
- 'on-drag'：開始拖曳時立即關閉鍵盤

| Type                    | Required |
| ----------------------- | -------- |
| enum('none', 'on-drag') | No       |

---

### `onDrawerClose`

當導航視圖關閉時觸發的函式。

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `onDrawerOpen`

當導航視圖開啟時觸發的函式。

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `onDrawerSlide`

當與導航視圖發生互動時觸發的函式。

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `onDrawerStateChanged`

當抽屜狀態改變時觸發的函式，可能狀態包括：

- idle：當前無任何導航視圖互動
- dragging：正在與導航視圖互動
- settling：導航視圖互動結束，正在完成開啟或關閉的動畫

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `renderNavigationView`

渲染於螢幕側邊、可被拉出的導航視圖元件。

| Type     | Required |
| -------- | -------- |
| function | Yes      |

---

### `statusBarBackgroundColor`

讓抽屜佔據整個屏幕並繪製狀態欄的背景，使其能夠在狀態欄上方開啟。此屬性僅在 API 21+ 以上版本有效。

| Type               | Required |
| ------------------ | -------- |
| [color](colors.md) | No       |

## 方法

### `closeDrawer()`

```tsx
closeDrawer();
```

關閉抽屜。

---

### `openDrawer()`

```tsx
openDrawer();
```

開啟抽屜。