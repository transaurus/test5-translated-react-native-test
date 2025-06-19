---
id: intro-react
title: React Fundamentals
description: To understand React Native fully, you need a solid foundation in React. This short introduction to React can help you get started or get refreshed.
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

React Native 基於 [React](https://react.dev/) 運行，這是一個用 JavaScript 構建用戶界面的熱門開源庫。要充分發揮 React Native 的優勢，理解 React 本身很有幫助。本節內容可作為入門指南或複習課程。

我們將介紹 React 的核心概念：

- 組件
- JSX
- props
- state

若想深入學習，建議查閱 [React 官方文檔](https://react.dev/learn)。

## 你的第一個組件

本篇 React 入門指南使用貓咪作為範例：這些友好親人的小動物需要名字和工作的咖啡館。以下是你的第一個 Cat 組件：

```SnackPlayer name=Your%20Cat
import React from 'react';
import {Text} from 'react-native';

const Cat = () => {
  return <Text>Hello, I am your cat!</Text>;
};

export default Cat;
```

具體操作如下：要定義你的 `Cat` 組件，首先使用 JavaScript 的 [`import`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) 語句導入 React 和 React Native 的 [`Text`](/docs/next/text) 核心組件：

```tsx
import React from 'react';
import {Text} from 'react-native';
```

你的組件以函數形式開始：

```tsx
const Cat = () => {};
```

你可以將組件視為藍圖。函數組件返回的內容會被渲染為 **React 元素**，這些元素用於描述屏幕上要顯示的內容。

這裡的 `Cat` 組件將渲染一個 `<Text>` 元素：

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

> 這是導出組件的多種方式之一。此類導出方式與 Snack Player 兼容良好。但根據應用的文件結構，可能需要採用其他約定方式。[JavaScript 導入導出速查表](https://medium.com/dailyjs/javascript-module-cheatsheet-7bd474f1d829)可提供幫助。

現在仔細觀察這個 `return` 語句。`<Text>Hello, I am your cat!</Text>` 使用了一種讓元素編寫更便捷的 JavaScript 語法：JSX。

## JSX

React 和 React Native 使用 **JSX** 語法，允許你在 JavaScript 中直接編寫元素：`<Text>Hello, I am your cat!</Text>`。React 文檔提供 [完整的 JSX 指南](https://react.dev/learn/writing-markup-with-jsx)可供深入學習。由於 JSX 就是 JavaScript，你可以在其中使用變量。這裡我們聲明了貓咪的名字 `name`，並用花括號將其嵌入 `<Text>`。

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

> 由於 JSX 包含在 React 庫中，若文件頂部沒有 `import React from 'react'` 語句，JSX 將無法工作！

## 自定義組件

你已經認識了 [React Native 核心組件](intro-react-native-components)。React 允許你將這些組件相互嵌套來創建新組件，這種可嵌套、可複用的組件正是 React 範式的核心。

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

你可以在咖啡館中放入任意數量的貓咪。每個 `<Cat>` 都會渲染一個獨特的元素——你可以透過 props 來客製化它們。

## Props

**Props** 是「屬性」(properties) 的簡稱。Props 讓你能夠客製化 React 元件。例如，這裡你為每個 `<Cat>` 傳遞不同的 `name`，讓 `Cat` 渲染時使用：

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

React Native 的大多數核心元件也可以透過 props 來客製化。舉例來說，使用 [`Image`](image) 時，你可以傳遞一個名為 [`source`](image#source) 的 prop 來定義它顯示的圖片：

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

`Image` 有[許多不同的 props](image#props)，包括 [`style`](image#style)，它接受一個包含設計與佈局相關屬性值對的 JavaScript 物件。

> Notice the double curly braces `{{ }}` surrounding `style`‘s width and height. In JSX, JavaScript values are referenced with `{}`. This is handy if you are passing something other than a string as props, like an array or number: `<Cat food={["fish", "kibble"]} age={2} />`. However, JS objects are **_also_** denoted with curly braces: `{width: 200, height: 200}`. Therefore, to pass a JS object in JSX, you must wrap the object in **another pair** of curly braces: `{{width: 200, height: 200}}`

你可以透過 props 和核心元件 [`Text`](text)、[`Image`](image) 和 [`View`](view) 來建構許多東西！但要建立互動式功能，你需要狀態。

## State

你可以將 props 視為用來配置元件渲染方式的參數，而**state** 則像是元件的個人資料儲存空間。State 適用於處理隨時間變化的資料或來自使用者互動的資料。State 賦予你的元件記憶力！

> 一般來說，當元件渲染時，使用 props 來配置它。使用 state 來追蹤任何你預期會隨時間變化的元件資料。

以下範例發生在一間貓咪咖啡館，兩隻飢餓的貓咪正等待餵食。牠們的飢餓狀態（我們預期會隨時間變化，與牠們的名字不同）被儲存為 state。要餵食貓咪，請按下牠們的按鈕——這將更新牠們的狀態。

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

首先，你需要從 React 中導入 `useState`，如下所示：

```tsx
import React, {useState} from 'react';
```

然後，在元件的函數內部呼叫 `useState` 來宣告元件的 state。在此範例中，`useState` 創建了一個 `isHungry` 狀態變數：

```tsx
const Cat = (props: CatProps) => {
  const [isHungry, setIsHungry] = useState(true);
  // ...
};
```

> 你可以使用 `useState` 來追蹤任何類型的資料：字串、數字、布林值、陣列、物件。例如，你可以用 `const [timesPetted, setTimesPetted] = useState(0)` 來追蹤貓咪被撫摸的次數！

呼叫 `useState` 會做兩件事：

- 它創建一個帶有初始值的「狀態變數」——在此範例中，狀態變數是 `isHungry`，其初始值為 `true`
- 它創建一個用來設定該狀態變數值的函數——`setIsHungry`

變數命名並不重要。但將模式理解為 `[<getter>, <setter>] = useState(<initialValue>)` 會很有幫助。

接著加入 [`Button`](button) 核心元件並賦予它 `onPress` 屬性：

```tsx
<Button
  onPress={() => {
    setIsHungry(false);
  }}
  //..
/>
```

現在當按鈕被點擊時，`onPress` 會觸發並呼叫 `setIsHungry(false)`。這會將狀態變數 `isHungry` 設為 `false`。當 `isHungry` 為 false 時，`Button` 的 `disabled` 屬性會被設為 `true`，其 `title` 也會隨之改變：

```tsx
<Button
  //..
  disabled={!isHungry}
  title={isHungry ? 'Give me some food, please!' : 'Thank you!'}
/>
```

> 你可能注意到雖然 `isHungry` 是 [const](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/const)，但它似乎可以被重新賦值！實際上當像 `setIsHungry` 這樣的狀態設置函數被呼叫時，其所屬元件會重新渲染。此時 `Cat` 函數會再次執行——而這次 `useState` 會回傳 `isHungry` 的下一個值。

最後，將你的貓咪們放進 `Cafe` 元件中：

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

> 看到上面的 `<>` 和 `</>` 了嗎？這些 JSX 片段稱為 [fragments](https://react.dev/reference/react/Fragment)。相鄰的 JSX 元素必須被包裹在一個封閉標籤中。Fragments 讓你可以做到這點，而無需嵌套額外不必要的包裹元素如 `View`。

---

現在你已經了解 React 和 React Native 的核心元件，讓我們更深入探討其中一些核心元件，請參閱 [處理 `<TextInput>`](handling-text-input)。