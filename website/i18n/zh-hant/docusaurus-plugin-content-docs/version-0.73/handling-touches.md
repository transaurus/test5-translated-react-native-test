---
id: handling-touches
title: Handling Touches
---

使用者主要透過觸控與行動應用程式互動。他們可以結合多種手勢操作，例如點擊按鈕、滾動列表或縮放地圖。React Native 提供了處理各種常見手勢的元件，以及一套完整的[手勢回應系統](gesture-responder-system.md)來實現進階手勢識別，但開發者最常使用的基礎元件仍是按鈕(Button)。

## 顯示基礎按鈕

[Button](button.md)元件提供了能在所有平台完美渲染的基礎按鈕。最簡範例如下：

```tsx
<Button
  onPress={() => {
    console.log('You tapped the button!');
  }}
  title="Press Me"
/>
```

在iOS上會渲染藍色文字標籤，在Android上則會顯示藍色圓角矩形與淺色文字。點擊按鈕將觸發"onPress"函數，本例中會顯示警示彈窗。您可透過"color"屬性來自訂按鈕顏色。

![](/docs/assets/Button.png)

您可透過下方範例實際操作`Button`元件。點擊右下角切換鈕選擇預覽平台，再點擊"Tap to Play"即可預覽應用效果。

```SnackPlayer name=Button%20Basics
import React, {Component} from 'react';
import {Alert, Button, StyleSheet, View} from 'react-native';

export default class ButtonBasics extends Component {
  _onPressButton() {
    Alert.alert('You tapped the button!');
  }

  render() {
    return (
      <View style={styles.container}>
        <View style={styles.buttonContainer}>
          <Button onPress={this._onPressButton} title="Press Me" />
        </View>
        <View style={styles.buttonContainer}>
          <Button
            onPress={this._onPressButton}
            title="Press Me"
            color="#841584"
          />
        </View>
        <View style={styles.alternativeLayoutButtonContainer}>
          <Button onPress={this._onPressButton} title="This looks great!" />
          <Button onPress={this._onPressButton} title="OK!" color="#841584" />
        </View>
      </View>
    );
  }
}

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
```

## 可觸控元件

若基礎按鈕樣式不符需求，可使用React Native提供的"Touchable"系列元件自訂按鈕。這些元件能捕捉點擊手勢並在識別時提供視覺回饋，但需自行設計樣式。

根據所需回饋類型選擇元件：

- Generally, you can use [**TouchableHighlight**](touchablehighlight.md) anywhere you would use a button or link on web. The view's background will be darkened when the user presses down on the button.

- You may consider using [**TouchableNativeFeedback**](touchablenativefeedback.md) on Android to display ink surface reaction ripples that respond to the user's touch.

- [**TouchableOpacity**](touchableopacity.md) can be used to provide feedback by reducing the opacity of the button, allowing the background to be seen through while the user is pressing down.

- If you need to handle a tap gesture but you don't want any feedback to be displayed, use [**TouchableWithoutFeedback**](touchablewithoutfeedback.md).

長按手勢可透過各Touchable元件的`onLongPress`屬性處理：

實際操作範例：

```SnackPlayer name=Touchables
import React, {Component} from 'react';
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

export default class Touchables extends Component {
  _onPressButton() {
    Alert.alert('You tapped the button!');
  }

  _onLongPressButton() {
    Alert.alert('You long-pressed the button!');
  }

  render() {
    return (
      <View style={styles.container}>
        <TouchableHighlight onPress={this._onPressButton} underlayColor="white">
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableHighlight</Text>
          </View>
        </TouchableHighlight>
        <TouchableOpacity onPress={this._onPressButton}>
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableOpacity</Text>
          </View>
        </TouchableOpacity>
        <TouchableNativeFeedback
          onPress={this._onPressButton}
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
        <TouchableWithoutFeedback onPress={this._onPressButton}>
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableWithoutFeedback</Text>
          </View>
        </TouchableWithoutFeedback>
        <TouchableHighlight
          onPress={this._onPressButton}
          onLongPress={this._onLongPressButton}
          underlayColor="white">
          <View style={styles.button}>
            <Text style={styles.buttonText}>Touchable with Long Press</Text>
          </View>
        </TouchableHighlight>
      </View>
    );
  }
}

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
```

## 滾動與滑動

觸控裝置常見手勢包含滑動(swipe)與拖曳(pan)，用於滾動列表或切換頁面，請參考核心元件[ScrollView](scrollview.md)。

## 已知問題

- [react-native#29308](https://github.com/facebook/react-native/issues/29308#issuecomment-792864162)：觸控區域不會超出父視窗邊界，且Android不支援負邊距