---
id: handling-touches
title: Handling Touches
---

使用者主要透過觸控與行動應用程式互動。他們可以結合多種手勢操作，例如點擊按鈕、滾動列表或縮放地圖。React Native 提供了處理各種常見手勢的元件，以及一套完整的[手勢回應系統](gesture-responder-system.md)來實現進階手勢識別，但您最可能感興趣的會是基礎的 Button 元件。

## 顯示基礎按鈕

[Button](button.md) 元件提供了能在所有平台完美渲染的基礎按鈕。顯示按鈕的最小範例如下：

```tsx
<Button
  onPress={() => {
    console.log('You tapped the button!');
  }}
  title="Press Me"
/>
```

在 iOS 上會渲染藍色標籤，在 Android 上則會顯示帶有淺色文字的藍色圓角矩形。按下按鈕會觸發 "onPress" 函式，此範例中會顯示警示彈窗。您可透過 "color" 屬性來自訂按鈕顏色。

![](/docs/assets/Button.png)

請直接透過下方範例試用 `Button` 元件。您可點擊右下角切換按鈕選擇預覽平台，再點擊「Tap to Play」預覽應用程式。

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

若基礎按鈕樣式不符合需求，您可以使用 React Native 提供的任何「Touchable」元件來自訂按鈕。這些元件能捕捉點擊手勢，並在手勢識別時提供視覺回饋，但不會預設任何樣式，因此需要自行設計外觀。

根據所需回饋類型選擇「Touchable」元件：

- 通常可將 [**TouchableHighlight**](touchablehighlight.md) 用於任何需要網頁按鈕或連結的情境。使用者按下時會使視圖背景變暗。

- 在 Android 平台上可考慮使用 [**TouchableNativeFeedback**](touchablenativefeedback.md)，能顯示墨水漣漪擴散效果來回應觸控。

- [**TouchableOpacity**](touchableopacity.md) 透過降低按鈕不透明度來提供回饋，讓使用者按壓時能透出背景內容。

- 若需處理點擊手勢但不需要視覺回饋，請使用 [**TouchableWithoutFeedback**](touchablewithoutfeedback.md)。

某些情況下，您可能需要偵測使用者長按視圖的行為。所有「Touchable」元件都可透過 `onLongPress` 屬性傳遞函式來處理長按事件。

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

觸控裝置常見手勢包含滑動(swipe)與拖曳(pan)，可用來滾動項目列表或切換內容頁面。請參考核心元件 [ScrollView](scrollview.md)。

## 已知問題

- [react-native#29308](https://github.com/facebook/react-native/issues/29308#issuecomment-792864162)：觸控區域不會超出父視圖邊界，且 Android 不支援負邊距設定。