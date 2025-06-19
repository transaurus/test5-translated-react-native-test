---
id: handling-touches
title: Handling Touches
---

使用者主要透過觸控與行動應用程式互動。他們可以組合使用多種手勢，例如點擊按鈕、滾動列表或縮放地圖。React Native 提供了處理各種常見手勢的元件，以及一套完整的[手勢回應系統](gesture-responder-system.md)來實現更進階的手勢識別，但您最可能感興趣的元件是基礎的按鈕(Button)。

## 顯示基礎按鈕

[按鈕元件](button.md)提供了一個能在所有平台上美觀渲染的基礎按鈕。顯示按鈕的最簡範例如下：

```tsx
<Button
  onPress={() => {
    console.log('You tapped the button!');
  }}
  title="Press Me"
/>
```

這會在 iOS 上渲染藍色標籤，在 Android 上渲染帶淺色文字的藍色圓角矩形。按下按鈕會呼叫 "onPress" 函數，本例中會顯示警示彈窗。您可透過指定 "color" 屬性來改變按鈕顏色。

![](/docs/assets/Button.png)

請使用下方範例實際操作 `Button` 元件。您可點擊右下角切換按鈕選擇預覽平台，再點擊「Tap to Play」預覽應用程式。

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

若基礎按鈕不符合您的應用程式風格，可使用 React Native 提供的任何「可觸控(Touchable)」元件自建按鈕。這些元件能捕捉點擊手勢，並在手勢識別時顯示反饋效果，但不會提供預設樣式，因此您需要自行設計視覺效果。

根據所需反饋類型選擇「可觸控」元件：

- 通常可將[**可觸控高亮(TouchableHighlight)**](touchablehighlight.md)用於任何網頁按鈕或連結的位置。使用者按下時，元件背景會變暗。

- 在 Android 上可考慮使用[**原生觸控反饋(TouchableNativeFeedback)**](touchablenativefeedback.md)來顯示墨水漣漪效果。

- [**可觸控透明度(TouchableOpacity)**](touchableopacity.md)透過降低按鈕透明度提供反饋，讓背景在使用者按壓時顯現。

- 若需處理點擊手勢但不需要視覺反饋，請使用[**無反饋可觸控(TouchableWithoutFeedback)**](touchablewithoutfeedback.md)。

某些情況下，您可能需要偵測使用者長按元件的行為。所有「可觸控」元件都支援透過 `onLongPress` 屬性處理長按事件。

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

觸控螢幕裝置常見手勢包含滑動(swipe)和拖曳(pan)，例如滾動項目列表或翻頁瀏覽內容。請參考核心元件[滾動視圖(ScrollView)](scrollview.md)。

## 已知問題

- [react-native#29308](https://github.com/facebook/react-native/issues/29308#issuecomment-792864162)：觸控區域不會超出父視圖邊界，且 Android 不支援負邊距。