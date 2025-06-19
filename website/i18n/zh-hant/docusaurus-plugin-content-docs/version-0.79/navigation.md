---
id: navigation
title: Navigating Between Screens
---

行動應用程式很少由單一畫面組成。管理多個畫面的呈現與轉場過渡，通常由稱為「導航器」(navigator)的元件負責處理。

本指南涵蓋 React Native 中提供的各種導航元件。如果您剛開始接觸導航功能，可能會想使用 [React Navigation](navigation.md#react-navigation)。React Navigation 提供了直觀的導航解決方案，能在 Android 和 iOS 上實現常見的堆疊導航和分頁導航模式。

如果您要將 React Native 整合到已使用原生導航的應用程式中，或正在尋找 React Navigation 的替代方案，以下函式庫可在雙平台上提供原生導航功能：[react-native-navigation](https://github.com/wix/react-native-navigation)。

## React Navigation

這個社群開發的導航解決方案是獨立函式庫，開發者只需幾行程式碼就能設定應用程式的畫面結構。

### 安裝與設定

首先，您需要在專案中安裝這些套件：

```shell
npm install @react-navigation/native @react-navigation/native-stack
```

接著安裝必要的相依套件。根據您的專案是 Expo 託管專案還是純 React Native 專案，需執行不同的指令。

- 如果是 Expo 託管專案，請使用 `expo` 安裝相依套件：

  ```shell
  npx expo install react-native-screens react-native-safe-area-context
  ```

- 如果是純 React Native 專案，請使用 `npm` 安裝相依套件：

  ```shell
  npm install react-native-screens react-native-safe-area-context
  ```

  對於純 React Native 專案的 iOS 端，請確保已安裝 [CocoaPods](https://cocoapods.org/)。接著執行以下指令完成安裝：

  ```shell
  cd ios
  pod install
  cd ..
  ```

:::note
安裝後可能會看到與相依套件版本相關的警告。這些通常是由某些套件指定的版本範圍不正確所導致。只要應用程式能正常建置，大多數警告可安全忽略。
:::

現在您需要將整個應用程式包裹在 `NavigationContainer` 中。通常這會在入口檔案（如 `index.js` 或 `App.js`）中進行：

```tsx
import * as React from 'react';
import {NavigationContainer} from '@react-navigation/native';

const App = () => {
  return (
    <NavigationContainer>
      {/* Rest of your app code */}
    </NavigationContainer>
  );
};

export default App;
```

現在您的應用程式已準備好可在裝置/模擬器上建置並執行。

### 使用方法

現在您可以建立包含首頁畫面和个人資料畫面的應用程式：

```tsx
import * as React from 'react';
import {NavigationContainer} from '@react-navigation/native';
import {createNativeStackNavigator} from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

const MyStack = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen
          name="Home"
          component={HomeScreen}
          options={{title: 'Welcome'}}
        />
        <Stack.Screen name="Profile" component={ProfileScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};
```

In this example, there are 2 screens (`Home` and `Profile`) defined using the `Stack.Screen` component. Similarly, you can define as many screens as you like.

You can set options such as the screen title for each screen in the `options` prop of `Stack.Screen`.

每個畫面接受一個 React 元件作為 `component` 屬性。這些元件會接收到名為 `navigation` 的屬性，其中包含連結到其他畫面的各種方法。例如，您可以使用 `navigation.navigate` 跳轉到 `Profile` 畫面：

```tsx
const HomeScreen = ({navigation}) => {
  return (
    <Button
      title="Go to Jane's profile"
      onPress={() =>
        navigation.navigate('Profile', {name: 'Jane'})
      }
    />
  );
};
const ProfileScreen = ({navigation, route}) => {
  return <Text>This is {route.params.name}'s profile</Text>;
};
```

此 `native-stack` 導航器使用原生 API：iOS 上的 `UINavigationController` 和 Android 上的 `Fragment`，因此透過 `createNativeStackNavigator` 建立的導航行為將與直接基於這些 API 開發的原生應用完全相同，並具有相同的效能特性。

React Navigation 還提供適用於不同類型導航器的套件，例如分頁和抽屜式導航。您可以使用它們在應用程式中實現各種導航模式。

要完整了解 React Navigation，請參閱 [React Navigation 入門指南](https://reactnavigation.org/docs/getting-started)。