---
id: props
title: Props
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

Most components can be customized when they are created, with different parameters. These created parameters are called `props`, short for properties.

舉例來說，React Native 的基本元件之一為 `Image`。當您創建圖片時，可使用名為 `source` 的 prop 來控制顯示的圖片。

```SnackPlayer name=Props
import React from 'react';
import {Image} from 'react-native';

const Bananas = () => {
  let pic = {
    uri: 'https://upload.wikimedia.org/wikipedia/commons/d/de/Bananavarieties.jpg',
  };
  return (
    <Image source={pic} style={{width: 193, height: 110, marginTop: 50}} />
  );
};

export default Bananas;
```

請注意包圍 `{pic}` 的大括號——這會將變數 `pic` 嵌入 JSX 中。您可以在 JSX 的大括號內放入任何 JavaScript 表達式。

Your own components can also use `props`. This lets you make a single component that is used in many different places in your app, with slightly different properties in each place by referring to `props` in your `render` function. Here's an example:

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Props&ext=js
import React from 'react';
import {Text, View} from 'react-native';

const Greeting = props => {
  return (
    <View style={{alignItems: 'center'}}>
      <Text>Hello {props.name}!</Text>
    </View>
  );
};

const LotsOfGreetings = () => {
  return (
    <View style={{alignItems: 'center', top: 50}}>
      <Greeting name="Rexxar" />
      <Greeting name="Jaina" />
      <Greeting name="Valeera" />
    </View>
  );
};

export default LotsOfGreetings;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Props&ext=tsx
import React from 'react';
import {Text, View} from 'react-native';

type GreetingProps = {
  name: string;
};

const Greeting = (props: GreetingProps) => {
  return (
    <View style={{alignItems: 'center'}}>
      <Text>Hello {props.name}!</Text>
    </View>
  );
};

const LotsOfGreetings = () => {
  return (
    <View style={{alignItems: 'center', top: 50}}>
      <Greeting name="Rexxar" />
      <Greeting name="Jaina" />
      <Greeting name="Valeera" />
    </View>
  );
};

export default LotsOfGreetings;
```

</TabItem>
</Tabs>

將 `name` 作為 prop 使用，讓我們能客製化 `Greeting` 元件，從而重複利用該元件於各個問候語。此範例亦在 JSX 中使用 `Greeting` 元件，類似於[核心元件](intro-react-native-components)。這正是 React 的強大之處——若您希望使用不同的 UI 原始元件，您可以自行創建新的元件。

此處另一個新概念是 [`View`](view.md) 元件。[`View`](view.md) 作為其他元件的容器非常實用，有助於控制樣式與佈局。

透過 `props` 與基礎的 [`Text`](text.md)、[`Image`](image.md) 和 [`View`](view.md) 元件，您能建構多種靜態畫面。要讓應用程式隨時間變化，您需要[學習 State](state.md)。