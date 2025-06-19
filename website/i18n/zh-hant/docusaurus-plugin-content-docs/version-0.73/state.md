---
id: state
title: State
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

控制元件的資料有兩種類型：`props` 和 `state`。`props` 由父元件設定，且在元件的生命週期中固定不變。對於會變動的資料，我們必須使用 `state`。

一般來說，你應該在建構子中初始化 `state`，然後在需要變更時呼叫 `setState`。

舉例來說，假設我們想製作一個持續閃爍的文字。文字本身在閃爍元件建立時就被設定一次，因此文字本身是一個 `prop`。而「文字當前是顯示還是隱藏」的狀態會隨時間改變，這應該保存在 `state` 中。

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=State&ext=js
import React, {useState, useEffect} from 'react';
import {Text, View} from 'react-native';

const Blink = props => {
  const [isShowingText, setIsShowingText] = useState(true);

  useEffect(() => {
    const toggle = setInterval(() => {
      setIsShowingText(!isShowingText);
    }, 1000);

    return () => clearInterval(toggle);
  });

  if (!isShowingText) {
    return null;
  }

  return <Text>{props.text}</Text>;
};

const BlinkApp = () => {
  return (
    <View style={{marginTop: 50}}>
      <Blink text="I love to blink" />
      <Blink text="Yes blinking is so great" />
      <Blink text="Why did they ever take this out of HTML" />
      <Blink text="Look at me look at me look at me" />
    </View>
  );
};

export default BlinkApp;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=State&ext=tsx
import React, {useState, useEffect} from 'react';
import {Text, View} from 'react-native';

type BlinkProps = {
  text: string;
};

const Blink = (props: BlinkProps) => {
  const [isShowingText, setIsShowingText] = useState(true);

  useEffect(() => {
    const toggle = setInterval(() => {
      setIsShowingText(!isShowingText);
    }, 1000);

    return () => clearInterval(toggle);
  });

  if (!isShowingText) {
    return null;
  }

  return <Text>{props.text}</Text>;
};

const BlinkApp = () => {
  return (
    <View style={{marginTop: 50}}>
      <Blink text="I love to blink" />
      <Blink text="Yes blinking is so great" />
      <Blink text="Why did they ever take this out of HTML" />
      <Blink text="Look at me look at me look at me" />
    </View>
  );
};

export default BlinkApp;
```

</TabItem>
</Tabs>

在實際應用中，你可能不會用計時器來設定狀態。狀態的變更可能來自伺服器的新資料，或是使用者的輸入。你也可以使用狀態管理工具如 [Redux](https://redux.js.org/) 或 [MobX](https://mobx.js.org/) 來控制資料流。在這種情況下，你會使用 Redux 或 MobX 來修改狀態，而不是直接呼叫 `setState`。

當呼叫 setState 時，BlinkApp 會重新渲染其元件。透過在計時器內呼叫 setState，元件會在每次計時器觸發時重新渲染。

State 的工作原理與 React 中相同，因此關於處理狀態的更多細節，你可以參考 [React.Component API](https://react.dev/reference/react/Component#setstate)。此時你可能已經注意到，我們大多數範例都使用預設的文字顏色。要自訂文字顏色，你需要[學習樣式](style.md)。