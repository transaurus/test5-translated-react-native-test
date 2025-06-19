---
id: network
title: Networking
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

許多行動應用程式需要從遠端 URL 載入資源。您可能想向 REST API 發送 POST 請求，或者需要從其他伺服器獲取靜態內容片段。

## 使用 Fetch

React Native 提供了 [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) 來滿足您的網路請求需求。如果您曾使用過 `XMLHttpRequest` 或其他網路 API，Fetch 會讓您感到熟悉。您可以參考 MDN 的 [使用 Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) 指南以獲取更多資訊。

### 發送請求

要從任意 URL 獲取內容，您可以將 URL 傳遞給 fetch：

```tsx
fetch('https://mywebsite.com/mydata.json');
```

Fetch 還接受一個可選的第二參數，允許您自訂 HTTP 請求。您可能需要指定額外的標頭，或發送 POST 請求：

```tsx
fetch('https://mywebsite.com/endpoint/', {
  method: 'POST',
  headers: {
    Accept: 'application/json',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    firstParam: 'yourValue',
    secondParam: 'yourOtherValue',
  }),
});
```

請查閱 [Fetch Request 文檔](https://developer.mozilla.org/en-US/docs/Web/API/Request) 以獲取完整的屬性列表。

### 處理回應

上述範例展示了如何發送請求。在許多情況下，您會希望對回應進行處理。

網路請求本質上是非同步操作。Fetch 方法會返回一個 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)，讓您可以輕鬆編寫非同步程式碼：

```tsx
const getMoviesFromApi = () => {
  return fetch('https://reactnative.dev/movies.json')
    .then(response => response.json())
    .then(json => {
      return json.movies;
    })
    .catch(error => {
      console.error(error);
    });
};
```

您也可以在 React Native 應用程式中使用 `async` / `await` 語法：

```tsx
const getMoviesFromApiAsync = async () => {
  try {
    const response = await fetch(
      'https://reactnative.dev/movies.json',
    );
    const json = await response.json();
    return json.movies;
  } catch (error) {
    console.error(error);
  }
};
```

別忘了捕獲 `fetch` 可能拋出的錯誤，否則這些錯誤會被無聲地忽略。

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Fetch%20Example&ext=js
import React, {useEffect, useState} from 'react';
import {ActivityIndicator, FlatList, Text, View} from 'react-native';

const App = () => {
  const [isLoading, setLoading] = useState(true);
  const [data, setData] = useState([]);

  const getMovies = async () => {
    try {
      const response = await fetch('https://reactnative.dev/movies.json');
      const json = await response.json();
      setData(json.movies);
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    getMovies();
  }, []);

  return (
    <View style={{flex: 1, padding: 24}}>
      {isLoading ? (
        <ActivityIndicator />
      ) : (
        <FlatList
          data={data}
          keyExtractor={({id}) => id}
          renderItem={({item}) => (
            <Text>
              {item.title}, {item.releaseYear}
            </Text>
          )}
        />
      )}
    </View>
  );
};

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Fetch%20Example&ext=tsx
import React, {useEffect, useState} from 'react';
import {ActivityIndicator, FlatList, Text, View} from 'react-native';

type Movie = {
  id: string;
  title: string;
  releaseYear: string;
};

const App = () => {
  const [isLoading, setLoading] = useState(true);
  const [data, setData] = useState<Movie[]>([]);

  const getMovies = async () => {
    try {
      const response = await fetch('https://reactnative.dev/movies.json');
      const json = await response.json();
      setData(json.movies);
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    getMovies();
  }, []);

  return (
    <View style={{flex: 1, padding: 24}}>
      {isLoading ? (
        <ActivityIndicator />
      ) : (
        <FlatList
          data={data}
          keyExtractor={({id}) => id}
          renderItem={({item}) => (
            <Text>
              {item.title}, {item.releaseYear}
            </Text>
          )}
        />
      )}
    </View>
  );
};

export default App;
```

</TabItem>
</Tabs>

</TabItem>
<TabItem value="classical">

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Fetch%20Example&ext=js
import React, {Component} from 'react';
import {ActivityIndicator, FlatList, Text, View} from 'react-native';

export default class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      data: [],
      isLoading: true,
    };
  }

  async getMovies() {
    try {
      const response = await fetch('https://reactnative.dev/movies.json');
      const json = await response.json();
      this.setState({data: json.movies});
    } catch (error) {
      console.log(error);
    } finally {
      this.setState({isLoading: false});
    }
  }

  componentDidMount() {
    this.getMovies();
  }

  render() {
    const {data, isLoading} = this.state;

    return (
      <View style={{flex: 1, padding: 24}}>
        {isLoading ? (
          <ActivityIndicator />
        ) : (
          <FlatList
            data={data}
            keyExtractor={({id}) => id}
            renderItem={({item}) => (
              <Text>
                {item.title}, {item.releaseYear}
              </Text>
            )}
          />
        )}
      </View>
    );
  }
}
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Fetch%20Example&ext=tsx
import React, {Component} from 'react';
import {ActivityIndicator, FlatList, Text, View} from 'react-native';

type Movie = {
  id: string;
  title: string;
  releaseYear: string;
};

type AppState = {
  data: Movie[];
  isLoading: boolean;
};

export default class App extends Component {
  state: AppState = {
    data: [],
    isLoading: true,
  };

  async getMovies() {
    try {
      const response = await fetch('https://reactnative.dev/movies.json');
      const json = await response.json();
      this.setState({data: json.movies});
    } catch (error) {
      console.log(error);
    } finally {
      this.setState({isLoading: false});
    }
  }

  componentDidMount() {
    this.getMovies();
  }

  render() {
    const {data, isLoading} = this.state;

    return (
      <View style={{flex: 1, padding: 24}}>
        {isLoading ? (
          <ActivityIndicator />
        ) : (
          <FlatList
            data={data}
            keyExtractor={({id}) => id}
            renderItem={({item}) => (
              <Text>
                {item.title}, {item.releaseYear}
              </Text>
            )}
          />
        )}
      </View>
    );
  }
}
```

</TabItem>
</Tabs>

</TabItem>
</Tabs>

> 預設情況下，iOS 會阻止任何未使用 [SSL](https://hosting.review/web-hosting-glossary/#12) 加密的請求。如果您需要從明文 URL（以 `http` 開頭的 URL）獲取資料，您首先需要 [添加 App Transport Security 例外](integration-with-existing-apps.md#test-your-integration)。如果您事先知道需要訪問哪些網域，僅為這些網域添加例外會更安全；如果網域在運行時才知曉，您可以 [完全禁用 ATS](publishing-to-app-store.md#1-enable-app-transport-security)。但請注意，自 2017 年 1 月起，[Apple 的 App Store 審核會要求對禁用 ATS 提供合理理由](https://forums.developer.apple.com/thread/48979)。詳情請參閱 [Apple 的文檔](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW33)。

> 在 Android 上，自 API 等級 28 起，預設也會阻止明文流量。您可以通過在應用程式清單文件中設置 [`android:usesCleartextTraffic`](https://developer.android.com/guide/topics/manifest/application-element#usesCleartextTraffic) 來覆蓋此行為。

## 使用其他網路庫

The [XMLHttpRequest API](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) is built into React Native. This means that you can use third party libraries such as [frisbee](https://github.com/niftylettuce/frisbee) or [axios](https://github.com/mzabriskie/axios) that depend on it, or you can use the XMLHttpRequest API directly if you prefer.

```tsx
var request = new XMLHttpRequest();
request.onreadystatechange = e => {
  if (request.readyState !== 4) {
    return;
  }

  if (request.status === 200) {
    console.log('success', request.responseText);
  } else {
    console.warn('error');
  }
};

request.open('GET', 'https://mywebsite.com/endpoint/');
request.send();
```

> XMLHttpRequest 的安全模型與網頁不同，因為原生應用程式中沒有 [CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) 的概念。

## WebSocket 支援

React Native 也支援 [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)，這是一種透過單一 TCP 連線提供全雙工通訊通道的協定。

```tsx
var ws = new WebSocket('ws://host.com/path');

ws.onopen = () => {
  // connection opened
  ws.send('something'); // send a message
};

ws.onmessage = e => {
  // a message was received
  console.log(e.data);
};

ws.onerror = e => {
  // an error occurred
  console.log(e.message);
};

ws.onclose = e => {
  // connection closed
  console.log(e.code, e.reason);
};
```

## 已知的 `fetch` 與基於 cookie 驗證的問題

The following options are currently not working with `fetch`

- `redirect:manual`
- `credentials:omit`

* Having same name headers on Android will result in only the latest one being present. A temporary solution can be found here: https://github.com/facebook/react-native/issues/18837#issuecomment-398779994.
* Cookie based authentication is currently unstable. You can view some of the issues raised here: https://github.com/facebook/react-native/issues/23185
* As a minimum on iOS, when redirected through a `302`, if a `Set-Cookie` header is present, the cookie is not set properly. Since the redirect cannot be handled manually this might cause a scenario where infinite requests occur if the redirect is the result of an expired session.

## 在 iOS 上配置 NSURLSession

For some applications it may be appropriate to provide a custom `NSURLSessionConfiguration` for the underlying `NSURLSession` that is used for network requests in a React Native application running on iOS. For instance, one may need to set a custom user agent string for all network requests coming from the app or supply `NSURLSession` with an emphemeral `NSURLSessionConfiguration`. The function `RCTSetCustomNSURLSessionConfigurationProvider` allows for such customization. Remember to add the following import to the file in which `RCTSetCustomNSURLSessionConfigurationProvider` will be called:

```objectivec
#import <React/RCTHTTPRequestHandler.h>
```

`RCTSetCustomNSURLSessionConfigurationProvider` should be called early in the application life cycle such that it is readily available when needed by React, for instance:

```objectivec
-(void)application:(__unused UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

  // set RCTSetCustomNSURLSessionConfigurationProvider
  RCTSetCustomNSURLSessionConfigurationProvider(^NSURLSessionConfiguration *{
     NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
     // configure the session
     return configuration;
  });

  // set up React
  _bridge = [[RCTBridge alloc] initWithDelegate:self launchOptions:launchOptions];
}
```