---
id: communication-android
title: Communication between native and React Native
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在[整合至現有應用程式指南](integration-with-existing-apps)和[原生UI元件指南](legacy/native-components-android)中，我們學習了如何將React Native嵌入原生元件，反之亦然。當我們混合使用原生與React Native元件時，最終會需要在這兩個世界之間建立溝通管道。其他指南已提及部分實現方法，本文將統整現有技術方案。

## 簡介

React Native的設計靈感源自React，因此資訊流的基本概念相似。React中的資料流是單向的——我們維護元件層級結構，每個元件僅依賴其父元件與自身內部狀態。透過屬性(properties)實現：資料以自上而下(top-down)的方式從父元件傳遞給子元件。若祖先元件需依賴後代元件的狀態，則應向下傳遞回調函數(callback)供後代元件更新祖先狀態。

此概念同樣適用於React Native。只要我們完全在框架內建構應用程式，就能透過屬性和回調驅動應用。但當混合使用React Native與原生元件時，就需要特定的跨語言機制來實現資訊傳遞。

## 屬性

屬性是最直接的跨元件通訊方式。因此我們需要實現雙向傳遞：從原生到React Native，以及從React Native到原生。

### 從原生端傳遞屬性至React Native

您可透過在主活動(main activity)中自訂`ReactActivityDelegate`實現來傳遞屬性至React Native應用。此實現應覆寫`getLaunchOptions`方法，返回包含所需屬性的`Bundle`物件。

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>

<TabItem value="java">

```java
public class MainActivity extends ReactActivity {
  @Override
  protected ReactActivityDelegate createReactActivityDelegate() {
    return new ReactActivityDelegate(this, getMainComponentName()) {
      @Override
      protected Bundle getLaunchOptions() {
        Bundle initialProperties = new Bundle();
        ArrayList<String> imageList = new ArrayList<String>(Arrays.asList(
                "https://dummyimage.com/600x400/ffffff/000000.png",
                "https://dummyimage.com/600x400/000000/ffffff.png"
        ));
        initialProperties.putStringArrayList("images", imageList);
        return initialProperties;
      }
    };
  }
}
```

</TabItem>

<TabItem value="kotlin">

```kotlin
class MainActivity : ReactActivity() {
    override fun createReactActivityDelegate(): ReactActivityDelegate {
        return object : ReactActivityDelegate(this, mainComponentName) {
            override fun getLaunchOptions(): Bundle {
                val imageList = arrayListOf("https://dummyimage.com/600x400/ffffff/000000.png", "https://dummyimage.com/600x400/000000/ffffff.png")
                val initialProperties = Bundle().apply { putStringArrayList("images", imageList) }
                return initialProperties
            }
        }
    }
}
```

</TabItem>
</Tabs>

```tsx
import React from 'react';
import {View, Image} from 'react-native';

export default class ImageBrowserApp extends React.Component {
  renderImage(imgURI) {
    return <Image source={{uri: imgURI}} />;
  }
  render() {
    return <View>{this.props.images.map(this.renderImage)}</View>;
  }
}
```

`ReactRootView` provides a read-write property `appProperties`. After `appProperties` is set, the React Native app is re-rendered with new properties. The update is only performed when the new updated properties differ from the previous ones.

<Tabs groupId="android-language" queryString defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>

<TabItem value="java">

```java
Bundle updatedProps = mReactRootView.getAppProperties();
ArrayList<String> imageList = new ArrayList<String>(Arrays.asList(
        "https://dummyimage.com/600x400/ff0000/000000.png",
        "https://dummyimage.com/600x400/ffffff/ff0000.png"
));
updatedProps.putStringArrayList("images", imageList);

mReactRootView.setAppProperties(updatedProps);
```

</TabItem>

<TabItem value="kotlin">

```kotlin
var updatedProps: Bundle = reactRootView.getAppProperties()
var imageList = arrayListOf("https://dummyimage.com/600x400/ff0000/000000.png", "https://dummyimage.com/600x400/ffffff/ff0000.png")
```

</TabItem>

</Tabs>

隨時更新屬性皆可，但更新操作必須在主執行緒執行。而讀取(getter)操作可在任何執行緒進行。

目前無法僅更新部分屬性，建議您自行建立封裝層處理此需求。

> **_注意：_** 目前頂層RN元件的JS函數`componentWillUpdateProps`在屬性更新後不會被呼叫。但您可透過`componentDidMount`函數存取新屬性。

### 從React Native傳遞屬性至原生端

The problem exposing properties of native components is covered in detail in [this article](legacy/native-components-android#3-expose-view-property-setters-using-reactprop-or-reactpropgroup-annotation). In short, properties that are to be reflected in JavaScript needs to be exposed as setter method annotated with `@ReactProp`, then use them in React Native as if the component was an ordinary React Native component.

### 屬性的限制

跨語言屬性的主要缺點是不支援回調機制，這使我們無法實現自下而上(bottom-up)的資料綁定。例如當您希望透過JS操作移除原生父視圖中的RN子視圖時，純屬性機制無法實現此需求，因為資訊需逆向傳遞。

雖然存在跨語言回調的變體方案([參見此處](legacy/native-modules-android#callbacks))，但這些回調並非萬能解方。主要問題在於它們並非設計用來作為屬性傳遞，而是提供從JS觸發原生操作後，在JS端處理操作結果的機制。

## 其他跨語言互動方式（事件與原生模組）

如前一章所述，使用屬性存在一些限制。有時屬性不足以驅動應用程式的邏輯，我們需要一個提供更大靈活性的解決方案。本章涵蓋了 React Native 中可用的其他通訊技術。這些技術既可用於內部通訊（RN 中 JS 與原生層之間的互動），也可用於外部通訊（RN 與應用程式中「純原生」部分之間的互動）。

React Native 允許您執行跨語言函數呼叫。您可以從 JS 執行自訂原生程式碼，反之亦然。遺憾的是，根據我們所處的環境，實現相同目標的方式有所不同。對於原生端，我們使用事件機制來安排在 JS 中執行處理函數；而對於 React Native 端，我們則直接呼叫由原生模組匯出的方法。

### 從原生端呼叫 React Native 函數（事件）

事件在[這篇文章](legacy/native-components-android#events)中有詳細描述。請注意，使用事件無法保證執行時間，因為事件是在單獨的執行緒中處理的。

事件非常強大，因為它們允許我們無需持有 React Native 元件的參考即可對其進行修改。然而，在使用時可能會遇到一些陷阱：

- 由於事件可以從任何地方發送，它們可能會在專案中引入義大利麵條式的依賴關係。
- 事件共享命名空間，這意味著您可能會遇到名稱衝突。這些衝突無法靜態檢測，因此難以除錯。
- 如果您使用多個相同 React Native 元件的實例，並希望從事件的角度區分它們，則可能需要引入識別符並隨事件一起傳遞（可以使用原生視圖的 `reactTag` 作為識別符）。

### 從 React Native 呼叫原生函數（原生模組）

原生模組是可在 JS 中使用的 Java/Kotlin 類別。通常每個 JS 橋接器會為每個模組建立一個實例。它們可以向 React Native 匯出任意函數和常數。[這篇文章](legacy/native-modules-android)對此有詳細說明。

> **_警告_**：所有原生模組共享相同的命名空間。在建立新模組時請注意名稱衝突。