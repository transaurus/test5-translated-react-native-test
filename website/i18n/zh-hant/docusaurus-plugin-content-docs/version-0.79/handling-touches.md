---
id: handling-touches
title: Handling Touches
---

使用者主要透過觸控與行動應用程式互動。他們可以結合多種手勢操作，例如點擊按鈕、滾動列表或縮放地圖。React Native 提供了處理各種常見手勢的元件，以及一套完整的[手勢回應系統](gesture-responder-system.md)來實現進階手勢識別，但您最可能感興趣的會是基礎的按鈕元件。

## 顯示基礎按鈕

[按鈕元件](button.md)提供了能在所有平台良好渲染的基礎按鈕。顯示按鈕的最小範例如下：

```tsx
<Button
  onPress={() => {
    console.log('You tapped the button!');
  }}
  title="Press Me"
/>
```

這會在 iOS 上渲染藍色標籤，在 Android 上渲染帶淺色文字的藍色圓角矩形。按下按鈕會呼叫 "onPress" 函數，此範例中會顯示警示彈窗。您可透過指定 "color" 屬性來變更按鈕顏色。

![](/docs/assets/Button.png)

請直接使用下方範例試玩 `Button` 元件。您可點擊右下角切換按鈕選擇預覽平台，然後點擊「Tap to Play」預覽應用程式。

```SnackPlayer name=Button%20Basics
import React from 'react';
import {Alert, Button, StyleSheet, View} from 'react-native';

const ButtonBasics = () => {
  const onPress = () => {
    Alert.alert('You tapped the button!');
  };

  return (
    <View style={styles.container}>
      <View style={styles.buttonContainer}>
        <Button onPress={onPress} title="Press Me" />
      </View>
      <View style={styles.buttonContainer}>
        <Button onPress={onPress} title="Press Me" color="#841584" />
      </View>
      <View style={styles.alternativeLayoutButtonContainer}>
        <Button onPress={onPress} title="This looks great!" />
        <Button onPress={onPress} title="OK!" color="#841584" />
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  buttonContainer: {
    margin: 20,
  },
  alternativeLayoutButtonContainer: {
    margin: 20,
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
});

export default ButtonBasics;
```

## 可觸控元件

若基礎按鈕不符合您的應用程式風格，可使用 React Native 提供的任何「可觸控」元件來自訂按鈕。這些元件能捕捉點擊手勢，並在手勢被識別時提供視覺回饋。但需注意這些元件不提供預設樣式，您需要自行設計使其在應用程式中美觀呈現。

根據您想提供的回饋類型，可選用不同的「可觸控」元件：

- 通常可於任何需要網頁按鈕或連結的場景使用[**可觸控高亮**](touchablehighlight.md)。使用者按下按鈕時，元件背景會變暗。

- 在 Android 平台上可考慮使用[**原生觸控回饋**](touchablenativefeedback.md)，以顯示墨水漣漪效果回應使用者觸控。

- [**可觸控透明度**](touchableopacity.md)透過降低按鈕透明度提供回饋效果，讓背景在使用者按壓時能透出。

- 若需處理點擊手勢但不想顯示任何視覺回饋，請使用[**無回饋可觸控**](touchablewithoutfeedback.md)。

某些情況下，您可能需要偵測使用者長按某元件的行為。所有「可觸控」元件都支援透過 `onLongPress` 屬性傳遞函數來處理長按事件。

實際操作範例：

```SnackPlayer name=Touchables
import React from 'react';
import {
  Alert,
  Platform,
  StyleSheet,
  Text,
  TouchableHighlight,
  TouchableOpacity,
  TouchableNativeFeedback,
  TouchableWithoutFeedback,
  View,
} from 'react-native';

const Touchables = () => {
  const onPressButton = () => {
    Alert.alert('You tapped the button!');
  };

  const onLongPressButton = () => {
    Alert.alert('You long-pressed the button!');
  };

  return (
    <View style={styles.container}>
      <TouchableHighlight onPress={onPressButton} underlayColor="white">
        <View style={styles.button}>
          <Text style={styles.buttonText}>TouchableHighlight</Text>
        </View>
      </TouchableHighlight>
      <TouchableOpacity onPress={onPressButton}>
        <View style={styles.button}>
          <Text style={styles.buttonText}>TouchableOpacity</Text>
        </View>
      </TouchableOpacity>
      <TouchableNativeFeedback
        onPress={onPressButton}
        background={
          Platform.OS === 'android'
            ? TouchableNativeFeedback.SelectableBackground()
            : undefined
        }>
        <View style={styles.button}>
          <Text style={styles.buttonText}>
            TouchableNativeFeedback{' '}
            {Platform.OS !== 'android' ? '(Android only)' : ''}
          </Text>
        </View>
      </TouchableNativeFeedback>
      <TouchableWithoutFeedback onPress={onPressButton}>
        <View style={styles.button}>
          <Text style={styles.buttonText}>TouchableWithoutFeedback</Text>
        </View>
      </TouchableWithoutFeedback>
      <TouchableHighlight
        onPress={onPressButton}
        onLongPress={onLongPressButton}
        underlayColor="white">
        <View style={styles.button}>
          <Text style={styles.buttonText}>Touchable with Long Press</Text>
        </View>
      </TouchableHighlight>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    paddingTop: 60,
    alignItems: 'center',
  },
  button: {
    marginBottom: 30,
    width: 260,
    alignItems: 'center',
    backgroundColor: '#2196F3',
  },
  buttonText: {
    textAlign: 'center',
    padding: 20,
    color: 'white',
  },
});

export default Touchables;
```

## 滾動與滑動

觸控螢幕裝置常見的手勢包含滑動(swipe)與拖曳(pan)，這些手勢讓使用者能滾動項目列表或滑動瀏覽內容頁面。相關功能請參考核心元件[滾動視圖](scrollview.md)。

## 已知問題

- [react-native#29308](https://github.com/facebook/react-native/issues/29308#issuecomment-792864162)：觸控區域不會延伸至父元件邊界之外，且 Android 不支援負邊距設定。