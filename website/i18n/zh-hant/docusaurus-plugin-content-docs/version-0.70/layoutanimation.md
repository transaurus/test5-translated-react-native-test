---
id: layoutanimation
title: LayoutAnimation
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

當下一次佈局發生時，自動將視圖動畫到它們的新位置。

使用此API的常見方式是在函數式組件中更新狀態鉤子之前調用它，或在類組件中調用`setState`之前調用它。

請注意，為了讓此功能在**Android**上正常工作，您需要通過`UIManager`設置以下標誌：

```js
if (Platform.OS === 'android') {
  if (UIManager.setLayoutAnimationEnabledExperimental) {
    UIManager.setLayoutAnimationEnabledExperimental(true);
  }
}
```

## 範例

```SnackPlayer name=LayoutAnimation&supportedPlatforms=android,ios
import React, { useState } from "react";
import { LayoutAnimation, Platform, StyleSheet, Text, TouchableOpacity, UIManager, View } from "react-native";

if (
  Platform.OS === "android" &&
  UIManager.setLayoutAnimationEnabledExperimental
) {
  UIManager.setLayoutAnimationEnabledExperimental(true);
}
const App = () => {
  const [expanded, setExpanded] = useState(false);

  return (
    <View style={style.container}>
      <TouchableOpacity
        onPress={() => {
          LayoutAnimation.configureNext(LayoutAnimation.Presets.spring);
          setExpanded(!expanded);
        }}
      >
        <Text>Press me to {expanded ? "collapse" : "expand"}!</Text>
      </TouchableOpacity>
      {expanded && (
        <View style={style.tile}>
          <Text>I disappear sometimes!</Text>
        </View>
      )}
    </View>
  );
};

const style = StyleSheet.create({
  tile: {
    backgroundColor: "lightgrey",
    borderWidth: 0.5,
    borderColor: "#d6d7da"
  },
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
    overflow: "hidden"
  }
});

export default App;
```

---

# 參考

## 方法

### `configureNext()`

```jsx
static configureNext(config, onAnimationDidEnd?, onAnimationDidFail?)
```

安排在下次佈局時執行動畫。

#### 參數：

| Name               | Type     | Required | Description                         |
| ------------------ | -------- | -------- | ----------------------------------- |
| config             | object   | Yes      | See config description below.       |
| onAnimationDidEnd  | function | No       | Called when the animation finished. |
| onAnimationDidFail | function | No       | Called when the animation failed.   |

The `config` parameter is an object with the keys below. [`create`](layoutanimation.md#create) returns a valid object for `config`, and the [`Presets`](layoutanimation.md#presets) objects can also all be passed as the `config`.

- `duration` 以毫秒為單位
- `create`，用於動畫新視圖的可選配置
- `update`，用於動畫已更新視圖的可選配置
- `delete`，用於動畫移除視圖的可選配置

傳遞給`create`、`update`或`delete`的配置具有以下鍵：

- `type`, the [animation type](layoutanimation.md#types) to use
- `property`, the [layout property](layoutanimation.md#properties) to animate (optional, but recommended for `create` and `delete`)
- `springDamping` (number, optional and only for use with `type: Type.spring`)
- `initialVelocity` (number, optional)
- `delay` (number, optional)
- `duration` (number, optional)

---

### `create()`

```jsx
static create(duration, type, creationProp)
```

創建一個對象（帶有`create`、`update`和`delete`字段）的輔助函數，以傳遞給[`configureNext`](layoutanimation.md#configurenext)。`type`參數是一個[動畫類型](layoutanimation.md#types)，`creationProp`參數是一個[佈局屬性](layoutanimation.md#properties)。

**範例：**

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=LayoutAnimation&supportedPlatforms=android,ios
import React, { useState } from "react";
import {
  View,
  Platform,
  UIManager,
  LayoutAnimation,
  StyleSheet,
  Button
} from "react-native";

if (
  Platform.OS === "android" &&
  UIManager.setLayoutAnimationEnabledExperimental
) {
  UIManager.setLayoutAnimationEnabledExperimental(true);
}

const App = () => {
  const [boxPosition, setBoxPosition] = useState("left");

  const toggleBox = () => {
    LayoutAnimation.configureNext({
      duration: 500,
      create: { type: "linear", property: "opacity" },
      update: { type: "spring", springDamping: 0.4 },
      delete: { type: "linear", property: "opacity" }
    });
    setBoxPosition(boxPosition === "left" ? "right" : "left");
  };

  return (
    <View style={styles.container}>
      <View style={styles.buttonContainer}>
        <Button title="Toggle Layout" onPress={toggleBox} />
      </View>
      <View
        style={[styles.box, boxPosition === "left" ? null : styles.moveRight]}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "flex-start",
    justifyContent: "center"
  },
  box: {
    height: 100,
    width: 100,
    borderRadius: 5,
    margin: 8,
    backgroundColor: "blue"
  },
  moveRight: {
    alignSelf: "flex-end",
    height: 200,
    width: 200
  },
  buttonContainer: {
    alignSelf: "center"
  }
});

export default App;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=LayoutAnimation&supportedPlatforms=android,ios
import React, { Component } from "react";
import {
  View,
  Platform,
  UIManager,
  LayoutAnimation,
  StyleSheet,
  Button
} from "react-native";

if (
  Platform.OS === "android" &&
  UIManager.setLayoutAnimationEnabledExperimental
) {
  UIManager.setLayoutAnimationEnabledExperimental(true);
}

class App extends Component {
  state = {
    boxPosition: "left"
  };

  toggleBox = () => {
    LayoutAnimation.configureNext({
      duration: 500,
      create: { type: "linear", property: "opacity" },
      update: { type: "spring", springDamping: 0.4 },
      delete: { type: "linear", property: "opacity" }
    });
    this.setState({
      boxPosition: this.state.boxPosition === "left" ? "right" : "left"
    });
  };

  render() {
    return (
      <View style={styles.container}>
        <View style={styles.buttonContainer}>
          <Button title="Toggle Layout" onPress={this.toggleBox} />
        </View>
        <View
          style={[
            styles.box,
            this.state.boxPosition === "left" ? null : styles.moveRight
          ]}
        />
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "flex-start",
    justifyContent: "center"
  },
  box: {
    height: 100,
    width: 100,
    borderRadius: 5,
    margin: 8,
    backgroundColor: "blue"
  },
  moveRight: {
    alignSelf: "flex-end",
    height: 200,
    width: 200
  },
  buttonContainer: {
    alignSelf: "center"
  }
});

export default App;
```

</TabItem>
</Tabs>

## 屬性

### 類型

An enumeration of animation types to be used in the [`create`](layoutanimation.md#create) method, or in the `create`/`update`/`delete` configs for [`configureNext`](layoutanimation.md#configurenext). (example usage: `LayoutAnimation.Types.easeIn`)

| Types         |
| ------------- |
| spring        |
| linear        |
| easeInEaseOut |
| easeIn        |
| easeOut       |
| keyboard      |

---

### 屬性

An enumeration of layout properties to be animated to be used in the [`create`](layoutanimation.md#create) method, or in the `create`/`update`/`delete` configs for [`configureNext`](layoutanimation.md#configurenext). (example usage: `LayoutAnimation.Properties.opacity`)

| Properties |
| ---------- |
| opacity    |
| scaleX     |
| scaleY     |
| scaleXY    |

---

### 預設

一組預定義的動畫配置，傳遞給[`configureNext`](layoutanimation.md#configurenext)。

| Presets       | Value                                                                                                                                                                 |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| easeInEaseOut | `create(300, 'easeInEaseOut', 'opacity')`                                                                                                                             |
| linear        | `create(500, 'linear', 'opacity')`                                                                                                                                    |
| spring        | `{ duration: 700, create: { type: 'linear', property: 'opacity' }, update: { type: 'spring', springDamping: 0.4 }, delete: { type: 'linear', property: 'opacity' } }` |

---

### `easeInEaseOut`

Calls `configureNext()` with `Presets.easeInEaseOut`.

---

### `linear`

Calls `configureNext()` with `Presets.linear`.

---

### `spring`

Calls `configureNext()` with `Presets.spring`.

**範例：**

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=LayoutAnimation&supportedPlatforms=android,ios
import React, { useState } from "react";
import {
  View,
  Platform,
  UIManager,
  LayoutAnimation,
  StyleSheet,
  Button
} from "react-native";

if (
  Platform.OS === "android" &&
  UIManager.setLayoutAnimationEnabledExperimental
) {
  UIManager.setLayoutAnimationEnabledExperimental(true);
}

const App = () => {
  const [firstBoxPosition, setFirstBoxPosition] = useState("left");
  const [secondBoxPosition, setSecondBoxPosition] = useState("left");
  const [thirdBoxPosition, setThirdBoxPosition] = useState("left");

  const toggleFirstBox = () => {
    LayoutAnimation.configureNext(LayoutAnimation.Presets.easeInEaseOut);
    setFirstBoxPosition(firstBoxPosition === "left" ? "right" : "left");
  };

  const toggleSecondBox = () => {
    LayoutAnimation.configureNext(LayoutAnimation.Presets.linear);
    setSecondBoxPosition(secondBoxPosition === "left" ? "right" : "left");
  };

  const toggleThirdBox = () => {
    LayoutAnimation.configureNext(LayoutAnimation.Presets.spring);
    setThirdBoxPosition(thirdBoxPosition === "left" ? "right" : "left");
  };

  return (
    <View style={styles.container}>
      <View style={styles.buttonContainer}>
        <Button title="EaseInEaseOut" onPress={toggleFirstBox} />
      </View>
      <View
        style={[
          styles.box,
          firstBoxPosition === "left" ? null : styles.moveRight
        ]}
      />
      <View style={styles.buttonContainer}>
        <Button title="Linear" onPress={toggleSecondBox} />
      </View>
      <View
        style={[
          styles.box,
          secondBoxPosition === "left" ? null : styles.moveRight
        ]}
      />
      <View style={styles.buttonContainer}>
        <Button title="Spring" onPress={toggleThirdBox} />
      </View>
      <View
        style={[
          styles.box,
          thirdBoxPosition === "left" ? null : styles.moveRight
        ]}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "flex-start",
    justifyContent: "center"
  },
  box: {
    height: 100,
    width: 100,
    borderRadius: 5,
    margin: 8,
    backgroundColor: "blue"
  },
  moveRight: {
    alignSelf: "flex-end"
  },
  buttonContainer: {
    alignSelf: "center"
  }
});

export default App;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=LayoutAnimation&supportedPlatforms=android,ios
import React, { Component } from "react";
import {
  View,
  Platform,
  UIManager,
  LayoutAnimation,
  StyleSheet,
  Button
} from "react-native";

if (
  Platform.OS === "android" &&
  UIManager.setLayoutAnimationEnabledExperimental
) {
  UIManager.setLayoutAnimationEnabledExperimental(true);
}

class App extends Component {
  state = {
    firstBoxPosition: "left",
    secondBoxPosition: "left",
    thirdBoxPosition: "left"
  };

  toggleFirstBox = () => {
    LayoutAnimation.configureNext(LayoutAnimation.Presets.easeInEaseOut);
    this.setState({
      firstBoxPosition:
        this.state.firstBoxPosition === "left" ? "right" : "left"
    });
  };

  toggleSecondBox = () => {
    LayoutAnimation.configureNext(LayoutAnimation.Presets.linear);
    this.setState({
      secondBoxPosition:
        this.state.secondBoxPosition === "left" ? "right" : "left"
    });
  };

  toggleThirdBox = () => {
    LayoutAnimation.configureNext(LayoutAnimation.Presets.spring);
    this.setState({
      thirdBoxPosition:
        this.state.thirdBoxPosition === "left" ? "right" : "left"
    });
  };

  render() {
    return (
      <View style={styles.container}>
        <View style={styles.buttonContainer}>
          <Button title="EaseInEaseOut" onPress={this.toggleFirstBox} />
        </View>
        <View
          style={[
            styles.box,
            this.state.firstBoxPosition === "left" ? null : styles.moveRight
          ]}
        />
        <View style={styles.buttonContainer}>
          <Button title="Linear" onPress={this.toggleSecondBox} />
        </View>
        <View
          style={[
            styles.box,
            this.state.secondBoxPosition === "left" ? null : styles.moveRight
          ]}
        />
        <View style={styles.buttonContainer}>
          <Button title="Spring" onPress={this.toggleThirdBox} />
        </View>
        <View
          style={[
            styles.box,
            this.state.thirdBoxPosition === "left" ? null : styles.moveRight
          ]}
        />
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "flex-start",
    justifyContent: "center"
  },
  box: {
    height: 100,
    width: 100,
    borderRadius: 5,
    margin: 8,
    backgroundColor: "blue"
  },
  moveRight: {
    alignSelf: "flex-end"
  },
  buttonContainer: {
    alignSelf: "center"
  }
});

export default App;
```

</TabItem>
</Tabs>