---
id: intro-react
title: React Fundamentals
description: To understand React Native fully, you need a solid foundation in React. This short introduction to React can help you get started or get refreshed.
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

React Native 基於 [React](https://react.dev/) 運行，這是一個使用 JavaScript 構建用戶界面的熱門開源庫。要充分發揮 React Native 的優勢，理解 React 本身會有所幫助。本節內容可作為入門指南或複習課程。

我們將介紹 React 的核心概念：

- 組件
- JSX
- props
- state

若想深入學習，建議查閱 [React 官方文檔](https://react.dev/learn)。

## 你的第一個組件

本 React 入門指南的後續示例將以貓為主題：這些友好親切的生物需要名字和工作的咖啡館。以下是你的第一個 Cat 組件：

```SnackPlayer name=Your%20Cat
import React from 'react';
import {Text} from 'react-native';

const Cat = () => {
  return <Text>Hello, I am your cat!</Text>;
};

export default Cat;
```

具體操作如下：要定義 `Cat` 組件，首先使用 JavaScript 的 [`import`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) 導入 React 和 React Native 的 [`Text`](/docs/next/text) 核心組件：

```tsx
import React from 'react';
import {Text} from 'react-native';
```

你的組件以函數形式開始：

```tsx
const Cat = () => {};
```

可將組件視為藍圖。函數組件返回的內容會被渲染為 **React 元素**。React 元素用於描述你希望在屏幕上看到的內容。

此處 `Cat` 組件將渲染一個 `<Text>` 元素：

```tsx
const Cat = () => {
  return <Text>Hello, I am your cat!</Text>;
};
```

你可以使用 JavaScript 的 [`export default`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) 導出函數組件，以便在整個應用中使用：

```tsx
const Cat = () => {
  return <Text>Hello, I am your cat!</Text>;
};

export default Cat;
```

> 這是導出組件的多種方式之一。此類導出方式與 Snack Player 兼容良好。但根據應用的文件結構，可能需要採用其他約定。這份 [JavaScript 導入導出速查表](https://medium.com/dailyjs/javascript-module-cheatsheet-7bd474f1d829) 會有所幫助。

現在仔細觀察這個 `return` 語句。`<Text>Hello, I am your cat!</Text>` 使用了一種使元素編寫更便捷的 JavaScript 語法：JSX。

## JSX

React 和 React Native 使用 **JSX**，這種語法允許你在 JavaScript 中直接編寫元素：`<Text>Hello, I am your cat!</Text>`。React 文檔提供了 [完整的 JSX 指南](https://react.dev/learn/writing-markup-with-jsx) 供深入學習。由於 JSX 就是 JavaScript，你可以在其中使用變量。此處聲明了貓的名字 `name`，並用花括號將其嵌入 `<Text>`。

```SnackPlayer name=Curly%20Braces
import React from 'react';
import {Text} from 'react-native';

const Cat = () => {
  const name = 'Maru';
  return <Text>Hello, I am {name}!</Text>;
};

export default Cat;
```

花括號內可包含任何 JavaScript 表達式，包括函數調用如 `{getFullName("Rum", "Tum", "Tugger")}`：

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Curly%20Braces&ext=js
import React from 'react';
import {Text} from 'react-native';

const getFullName = (firstName, secondName, thirdName) => {
  return firstName + ' ' + secondName + ' ' + thirdName;
};

const Cat = () => {
  return <Text>Hello, I am {getFullName('Rum', 'Tum', 'Tugger')}!</Text>;
};

export default Cat;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Curly%20Braces&ext=tsx
import React from 'react';
import {Text} from 'react-native';

const getFullName = (
  firstName: string,
  secondName: string,
  thirdName: string,
) => {
  return firstName + ' ' + secondName + ' ' + thirdName;
};

const Cat = () => {
  return <Text>Hello, I am {getFullName('Rum', 'Tum', 'Tugger')}!</Text>;
};

export default Cat;
```

</TabItem>
</Tabs>

你可以將花括號視為通往 JSX 中 JavaScript 功能的傳送門！

> 由於 JSX 包含在 React 庫中，若文件頂部沒有 `import React from 'react'`，它將無法工作！

## 自定義組件

你已接觸過 [React Native 核心組件](intro-react-native-components)。React 允許你將這些組件相互嵌套以創建新組件。這些可嵌套、可複用的組件是 React 範式的核心。

例如，你可以將 [`Text`](text) 和 [`TextInput`](textinput) 嵌套在 [`View`](view) 中，React Native 會將它們一併渲染：

```SnackPlayer name=Custom%20Components
import React from 'react';
import {Text, TextInput, View} from 'react-native';

const Cat = () => {
  return (
    <View>
      <Text>Hello, I am...</Text>
      <TextInput
        style={{
          height: 40,
          borderColor: 'gray',
          borderWidth: 1,
        }}
        defaultValue="Name me!"
      />
    </View>
  );
};

export default Cat;
```

#### 開發者須知

<Tabs groupId="guide" queryString defaultValue="web" values={constants.getDevNotesTabs(["android", "web"])}>

<TabItem value="web">

> If you’re familiar with web development, `<View>` and `<Text>` might remind you of HTML! You can think of them as the `<div>` and `<p>` tags of application development.

</TabItem>
<TabItem value="android">

> On Android, you usually put your views inside `LinearLayout`, `FrameLayout`, `RelativeLayout`, etc. to define how the view’s children will be arranged on the screen. In React Native, `View` uses Flexbox for its children’s layout. You can learn more in [our guide to layout with Flexbox](flexbox).

</TabItem>
</Tabs>

你可以透過使用 `<Cat>` 來多次渲染這個元件，且無需重複程式碼：

```SnackPlayer name=Multiple%20Components
import React from 'react';
import {Text, View} from 'react-native';

const Cat = () => {
  return (
    <View>
      <Text>I am also a cat!</Text>
    </View>
  );
};

const Cafe = () => {
  return (
    <View>
      <Text>Welcome!</Text>
      <Cat />
      <Cat />
      <Cat />
    </View>
  );
};

export default Cafe;
```

Any component that renders other components is a **parent component.** Here, `Cafe` is the parent component and each `Cat` is a **child component.**

你可以在咖啡館中放入任意數量的貓咪。每個 `<Cat>` 都會渲染一個獨特的元素——你可以透過 props 來自訂它們。

## Props

**Props** 是「屬性」(properties) 的簡稱。Props 讓你能夠自訂 React 元件。例如，這裡你為每個 `<Cat>` 傳遞不同的 `name`，讓 `Cat` 渲染：

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Multiple%20Props&ext=js
import React from 'react';
import {Text, View} from 'react-native';

const Cat = props => {
  return (
    <View>
      <Text>Hello, I am {props.name}!</Text>
    </View>
  );
};

const Cafe = () => {
  return (
    <View>
      <Cat name="Maru" />
      <Cat name="Jellylorum" />
      <Cat name="Spot" />
    </View>
  );
};

export default Cafe;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Multiple%20Props&ext=tsx
import React from 'react';
import {Text, View} from 'react-native';

type CatProps = {
  name: string;
};

const Cat = (props: CatProps) => {
  return (
    <View>
      <Text>Hello, I am {props.name}!</Text>
    </View>
  );
};

const Cafe = () => {
  return (
    <View>
      <Cat name="Maru" />
      <Cat name="Jellylorum" />
      <Cat name="Spot" />
    </View>
  );
};

export default Cafe;
```

</TabItem>
</Tabs>

React Native 的大多數核心元件也可以透過 props 來自訂。例如，使用 [`Image`](image) 時，你可以傳遞一個名為 [`source`](image#source) 的 prop 來定義它顯示的圖片：

```SnackPlayer name=Props
import React from 'react';
import {Text, View, Image} from 'react-native';

const CatApp = () => {
  return (
    <View>
      <Image
        source={{
          uri: 'https://reactnative.dev/docs/assets/p_cat1.png',
        }}
        style={{width: 200, height: 200}}
      />
      <Text>Hello, I am your cat!</Text>
    </View>
  );
};

export default CatApp;
```

`Image` 有[許多不同的 props](image#props)，包括 [`style`](image#style)，它接受一個包含設計和佈局相關屬性值對的 JS 物件。

> Notice the double curly braces `{{ }}` surrounding `style`‘s width and height. In JSX, JavaScript values are referenced with `{}`. This is handy if you are passing something other than a string as props, like an array or number: `<Cat food={["fish", "kibble"]} age={2} />`. However, JS objects are **_also_** denoted with curly braces: `{width: 200, height: 200}`. Therefore, to pass a JS object in JSX, you must wrap the object in **another pair** of curly braces: `{{width: 200, height: 200}}`

你可以透過 props 和核心元件 [`Text`](text)、[`Image`](image) 和 [`View`](view) 來建構許多東西！但要建構互動式的內容，你需要狀態。

## State

你可以將 props 視為用來設定元件渲染方式的參數，而**state** 則是元件的個人資料儲存空間。state 適用於處理隨時間變化的資料或來自使用者互動的資料。state 賦予你的元件記憶力！

> 一般來說，當元件渲染時，使用 props 來設定元件。使用 state 來追蹤任何你預期會隨時間變化的元件資料。

以下範例發生在一家貓咪咖啡館，兩隻飢餓的貓咪等待被餵食。牠們的飢餓狀態（我們預期會隨時間變化，不像牠們的名字）被儲存為 state。要餵食貓咪，請按下牠們的按鈕——這將更新牠們的 state。

You can add state to a component by calling [React’s `useState` Hook](https://react.dev/learn/state-a-components-memory). A Hook is a kind of function that lets you “hook into” React features. For example, `useState` is a Hook that lets you add state to function components. You can learn more about [other kinds of Hooks in the React documentation.](https://react.dev/reference/react)

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=State&ext=js
import React, {useState} from 'react';
import {Button, Text, View} from 'react-native';

const Cat = props => {
  const [isHungry, setIsHungry] = useState(true);

  return (
    <View>
      <Text>
        I am {props.name}, and I am {isHungry ? 'hungry' : 'full'}!
      </Text>
      <Button
        onPress={() => {
          setIsHungry(false);
        }}
        disabled={!isHungry}
        title={isHungry ? 'Give me some food, please!' : 'Thank you!'}
      />
    </View>
  );
};

const Cafe = () => {
  return (
    <>
      <Cat name="Munkustrap" />
      <Cat name="Spot" />
    </>
  );
};

export default Cafe;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=State&ext=tsx
import React, {useState} from 'react';
import {Button, Text, View} from 'react-native';

type CatProps = {
  name: string;
};

const Cat = (props: CatProps) => {
  const [isHungry, setIsHungry] = useState(true);

  return (
    <View>
      <Text>
        I am {props.name}, and I am {isHungry ? 'hungry' : 'full'}!
      </Text>
      <Button
        onPress={() => {
          setIsHungry(false);
        }}
        disabled={!isHungry}
        title={isHungry ? 'Give me some food, please!' : 'Thank you!'}
      />
    </View>
  );
};

const Cafe = () => {
  return (
    <>
      <Cat name="Munkustrap" />
      <Cat name="Spot" />
    </>
  );
};

export default Cafe;
```

</TabItem>
</Tabs>

首先，你需要從 React 導入 `useState`：

```tsx
import React, {useState} from 'react';
```

然後，你可以在元件的函數內部呼叫 `useState` 來宣告元件的 state。在此範例中，`useState` 創建了一個 `isHungry` state 變數：

```tsx
const Cat = (props: CatProps) => {
  const [isHungry, setIsHungry] = useState(true);
  // ...
};
```

> 你可以使用 `useState` 來追蹤任何類型的資料：字串、數字、布林值、陣列、物件。例如，你可以用 `const [timesPetted, setTimesPetted] = useState(0)` 來追蹤貓咪被撫摸的次數！

呼叫 `useState` 會做兩件事：

- 它創建一個帶有初始值的「state 變數」——在此範例中，state 變數是 `isHungry`，其初始值為 `true`
- 它創建一個函數來設定該 state 變數的值——`setIsHungry`

命名方式並不重要，但將模式理解為 `[<getter>, <setter>] = useState(<initialValue>)` 會很有幫助。

接著加入 [`Button`](button) 核心元件並賦予它一個 `onPress` 屬性：

```tsx
<Button
  onPress={() => {
    setIsHungry(false);
  }}
  //..
/>
```

現在，當有人按下按鈕時，`onPress` 會觸發並呼叫 `setIsHungry(false)`。這會將狀態變數 `isHungry` 設為 `false`。當 `isHungry` 為 false 時，`Button` 的 `disabled` 屬性會被設為 `true`，其 `title` 也會改變：

```tsx
<Button
  //..
  disabled={!isHungry}
  title={isHungry ? 'Give me some food, please!' : 'Thank you!'}
/>
```

> 你可能注意到，儘管 `isHungry` 是一個 [const](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/const)，它看似可以被重新賦值！實際情況是，當像 `setIsHungry` 這樣的狀態設定函數被呼叫時，其元件會重新渲染。在這個例子中，`Cat` 函數會再次執行——而這次，`useState` 會給我們 `isHungry` 的下一個值。

最後，將你的貓咪放入 `Cafe` 元件中：

```tsx
const Cafe = () => {
  return (
    <>
      <Cat name="Munkustrap" />
      <Cat name="Spot" />
    </>
  );
};
```

> 看到上面的 `<>` 和 `</>` 了嗎？這些 JSX 片段稱為 [fragments](https://react.dev/reference/react/Fragment)。相鄰的 JSX 元素必須被包裹在一個封閉標籤中。Fragments 讓你可以做到這一點，而無需嵌套一個額外的、不必要的包裹元素如 `View`。

---

現在你已經涵蓋了 React 和 React Native 的核心元件，讓我們更深入探討其中一些核心元件，看看如何[處理 `<TextInput>`](handling-text-input)。