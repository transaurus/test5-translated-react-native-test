---
id: handling-touches
title: Handling Touches
---

使用者主要透過觸控與行動應用程式互動，例如點擊按鈕、滾動列表或縮放地圖等手勢組合。React Native 提供處理各種常見手勢的元件，以及一套完整的[手勢回應系統](gesture-responder-system.md)來實現進階手勢識別，但開發者最常使用的基礎元件仍是按鈕(Button)。

## 顯示基礎按鈕

[按鈕元件](button.md)提供跨平台渲染良好的基礎按鈕，最簡範例如下：

```jsx
<Button
  onPress={() => {
    alert('You tapped the button!');
  }}
  title="Press Me"
/>
```

在iOS上會渲染藍色文字按鈕，Android則顯示藍色圓角矩形淺色文字。點擊按鈕會觸發"onPress"函數，此範例中會顯示警示彈窗。您可透過"color"屬性來自訂按鈕顏色。

![](/docs/assets/Button.png)

Go ahead and play around with the `Button` component using the example below. You can select which platform your app is previewed in by clicking on the toggle in the bottom right and then clicking on "Tap to Play" to preview the app.

```SnackPlayer name=Button%20Basics
import React, { Component } from 'react';
import { Button, StyleSheet, View } from 'react-native';

export default class ButtonBasics extends Component {
  _onPressButton() {
    alert('You tapped the button!')
  }

  render() {
    return (
      <View style={styles.container}>
        <View style={styles.buttonContainer}>
          <Button
            onPress={this._onPressButton}
            title="Press Me"
          />
        </View>
        <View style={styles.buttonContainer}>
          <Button
            onPress={this._onPressButton}
            title="Press Me"
            color="#841584"
          />
        </View>
        <View style={styles.alternativeLayoutButtonContainer}>
          <Button
            onPress={this._onPressButton}
            title="This looks great!"
          />
          <Button
            onPress={this._onPressButton}
            title="OK!"
            color="#841584"
          />
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
    margin: 20
  },
  alternativeLayoutButtonContainer: {
    margin: 20,
    flexDirection: 'row',
    justifyContent: 'space-between'
  }
});
```

## 觸控元件

若基礎按鈕樣式不符需求，可使用React Native提供的"Touchable"系列元件自訂按鈕。這些元件能捕捉點擊手勢並提供觸控回饋，但需自行設計樣式。

根據所需回饋類型選擇觸控元件：

- Generally, you can use [**TouchableHighlight**](touchablehighlight.md) anywhere you would use a button or link on web. The view's background will be darkened when the user presses down on the button.

- You may consider using [**TouchableNativeFeedback**](touchablenativefeedback.md) on Android to display ink surface reaction ripples that respond to the user's touch.

- [**TouchableOpacity**](touchableopacity.md) can be used to provide feedback by reducing the opacity of the button, allowing the background to be seen through while the user is pressing down.

- If you need to handle a tap gesture but you don't want any feedback to be displayed, use [**TouchableWithoutFeedback**](touchablewithoutfeedback.md).

長按手勢可透過各觸控元件的`onLongPress`屬性來處理，需傳入對應回調函數。

實際操作範例：

```SnackPlayer name=Touchables
import React, { Component } from 'react';
import { Platform, StyleSheet, Text, TouchableHighlight, TouchableOpacity, TouchableNativeFeedback, TouchableWithoutFeedback, View } from 'react-native';

export default class Touchables extends Component {
  _onPressButton() {
    alert('You tapped the button!')
  }

  _onLongPressButton() {
    alert('You long-pressed the button!')
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
            background={Platform.OS === 'android' ? TouchableNativeFeedback.SelectableBackground() : ''}>
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableNativeFeedback {Platform.OS !== 'android' ? '(Android only)' : ''}</Text>
          </View>
        </TouchableNativeFeedback>
        <TouchableWithoutFeedback
            onPress={this._onPressButton}
            >
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableWithoutFeedback</Text>
          </View>
        </TouchableWithoutFeedback>
        <TouchableHighlight onPress={this._onPressButton} onLongPress={this._onLongPressButton} underlayColor="white">
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
    alignItems: 'center'
  },
  button: {
    marginBottom: 30,
    width: 260,
    alignItems: 'center',
    backgroundColor: '#2196F3'
  },
  buttonText: {
    textAlign: 'center',
    padding: 20,
    color: 'white'
  }
});
```

## 滾動與滑動手勢

觸控裝置常見的滑動(swipe)與拖曳(pan)手勢，可透過[ScrollView](scrollview.md)核心元件實現內容滾動或頁面切換效果。

## 已知問題

- [react-native#29308](https://github.com/facebook/react-native/issues/29308#issuecomment-792864162)：觸控區域不會超出父視圖邊界，且Android不支援負邊距