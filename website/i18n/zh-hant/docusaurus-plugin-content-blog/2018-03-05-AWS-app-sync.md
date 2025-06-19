---
title: Using AWS with React Native
author: Richard Threlkeld
authorTitle: Senior Technical Product Manager at AWS Mobile
authorURL: 'https://twitter.com/undef_obj'
authorImageURL: 'https://pbs.twimg.com/profile_images/811239086581227520/APX1eZwF_400x400.jpg'
authorTwitter: undef_obj
tags: [engineering]
---

AWS 在科技產業中以雲端服務供應商聞名，其服務範疇涵蓋運算、儲存及資料庫技術，同時也提供全託管的無伺服器方案。AWS Mobile 團隊持續與客戶及 JavaScript 生態系成員密切合作，致力於提升雲端連線的行動與網頁應用程式在安全性、擴展性以及開發部署便利性。我們最初推出了一套[完整入門套件](https://github.com/awslabs/aws-mobile-react-native-starter)，而後續更有幾項新進展。

本篇部落格文章將探討 React 與 React Native 開發者感興趣的內容：

- [**AWS Amplify**](https://github.com/aws/aws-amplify)：一套用於雲端服務的 JavaScript 應用程式宣告式函式庫
- [**AWS AppSync**](https://aws.amazon.com/appsync/)：具備離線與即時功能的全託管 GraphQL 服務

## AWS Amplify

雖然透過 Create React Native App 和 Expo 等工具能快速初始化 React Native 應用程式，但當您嘗試將應用程式與雲端服務對接時，如何根據使用情境選擇基礎架構服務往往是一大挑戰。舉例來說，若您的 React Native 應用程式需要上傳照片，這些照片是否需要以使用者為單位進行保護？這可能意味著您需要某種註冊或登入流程。您希望建立專屬使用者目錄，還是直接整合社群媒體登入？或許您的應用程式還需要在使用者登入後，呼叫具有自訂商業邏輯的 API。

為協助 JavaScript 開發者解決這些問題，我們發布了名為 AWS Amplify 的函式庫。其設計核心是將功能劃分為不同「類別」，而非直接對應 AWS 的實作方式。例如，若您需要實現使用者註冊、登入後上傳私人照片的功能，只需在應用程式中引入 `Auth` 與 `Storage` 兩個類別：

```
import { Auth } from 'aws-amplify';

Auth.signIn(username, password)
    .then(user => console.log(user))
    .catch(err => console.log(err));

Auth.confirmSignIn(user, code)
    .then(data => console.log(data))
    .catch(err => console.log(err));
```

上述程式碼展示了 Amplify 協助處理的常見任務，例如透過電子郵件或簡訊發送多重要素驗證（MFA）碼。目前支援的主要類別包括：

- **Auth**：提供憑證自動化管理。開箱即用的實作採用 AWS 憑證進行簽署，並使用 [Amazon Cognito](https://aws.amazon.com/cognito/) 的 OIDC JWT 權杖。常見功能如 MFA 皆受支援。
- **Analytics**：僅需一行程式碼即可為已驗證或未驗證使用者啟用 [Amazon Pinpoint](https://aws.amazon.com/pinpoint/) 追蹤功能，並可依需求擴展自訂指標或屬性。
- **API**：透過 [AWS Signature Version 4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) 安全機制與 RESTful API 互動，特別適合 [Amazon API Gateway](https://aws.amazon.com/api-gateway/) 的無伺服器架構。
- **Storage**：簡化上傳、下載及列出 [Amazon S3](https://aws.amazon.com/s3/) 內容的操作指令，並能輕鬆以使用者為單位區分公開或私有資料。
- **Caching**：跨網頁應用與 React Native 的 LRU 快取介面，採用實作特定的持久化機制。
- **i18n and Logging**：提供國際化、本地化功能，以及除錯與日誌記錄能力。

Amplify 的優勢之一在於其針對特定程式環境內建了「最佳實踐」設計。例如我們發現，客戶與 React Native 開發者在開發過程中為求快速實現功能而採取的捷徑，往往會直接進入生產環境。這些做法可能影響擴展性或安全性，最終迫使團隊必須重構基礎架構與程式碼。

我們協助開發者避免此問題的一個實例是[《使用 AWS Lambda 的無伺服器參考架構》](https://www.allthingsdistributed.com/2016/06/aws-lambda-serverless-reference-architectures.html)。這些資源展示了在建置後端時，結合使用 Amazon API Gateway 和 AWS Lambda 的最佳實踐。此模式已被編碼至 Amplify 的 `API` 類別中。您可利用此模式與多個 REST 端點互動，並將標頭一路傳遞至 Lambda 函數以實現自訂業務邏輯。我們還發布了 [AWS Mobile CLI](https://docs.aws.amazon.com/aws-mobile/latest/developerguide/react-native-getting-started.html)，用於為新舊 React Native 專案引導這些功能。要開始使用，只需透過 **npm** 安裝並遵循配置提示：

```
npm install --global awsmobile-cli
awsmobile configure
```

另一個針對移動生態系統編碼的最佳實踐範例是密碼安全性。預設的 `Auth` 類別實現利用 Amazon Cognito 使用者池進行使用者註冊和登入。該服務採用[安全遠端密碼協議（Secure Remote Password protocol）](https://srp.stanford.edu)來保護使用者驗證過程。若您有意研究[該協議的數學原理](https://srp.stanford.edu/ndss.html#SECTION00032200000000000000)，會注意到在計算密碼驗證器時，必須使用一個大質數作為原始根的群組生成。在 React Native 環境中，[JIT 被禁用](/docs/javascript-environment)，這使得此類安全操作的大整數計算效能較低。為此，我們發布了 Android 和 iOS 的原生橋接，您可將其連結至專案中：

```
npm install --save aws-amplify-react-native
react-native link amazon-cognito-identity-js
```

我們也很高興看到 Expo 團隊已將此功能[納入其最新 SDK](https://blog.expo.io/expo-sdk-v25-0-0-is-now-available-714d10a8c3f7)，讓您無需退出即可使用 Amplify。

最後，針對 React Native（及 React）開發，Amplify 提供[高階組件（HOCs）](https://reactjs.org/docs/higher-order-components.html)，便於封裝功能，例如應用程式的註冊和登入：

```
import Amplify, { withAuthenticator } from 'aws-amplify-react-native';
import aws_exports from './aws-exports';

Amplify.configure(aws_exports);

class App extends React.Component {
...
}

export default withAuthenticator(App);
```

底層組件也以 `<Authenticator />` 形式提供，讓您能完全自訂 UI。它還提供管理使用者狀態的屬性，例如是否已登入或等待 MFA 確認，以及狀態變更時可觸發的回調函數。

同樣地，您會發現適用於不同用例的通用 React 組件。可根據需求自訂，例如在 `Storage` 模組中顯示 Amazon S3 的所有私有圖片：

```
<S3Album
  level="private"
  path={path}
  filter={(item) => /jpg/i.test(item.path)}/>
```

如先前所示，可透過 props 控制許多組件功能，例如公開或私有儲存選項。甚至能在使用者與特定 UI 組件互動時自動收集分析數據：

```
return <S3Album track/>
```

AWS Amplify 傾向於「約定優於配置」的開發風格，提供全域初始化程序或類別層級的初始化。最快速入門方式是使用 [aws-exports 檔案](https://aws.amazon.com/blogs/mobile/enhanced-javascript-development-with-aws-mobile-hub/)，但開發者亦可獨立使用該函式庫與現有資源整合。

For a deep dive into the philosophy and to see a full demo, check out the video from [AWS re\:Invent](https://www.youtube.com/watch?v=vAjf3lyjf8c).

## AWS AppSync

在 AWS Amplify 發布後不久，我們也推出了 [AWS AppSync](https://aws.amazon.com/appsync/)。這是一項全託管的 GraphQL 服務，具備離線和即時功能。雖然您可在不同客戶端程式語言（包括原生 Android 和 iOS）中使用 GraphQL，但它特別受 React Native 開發者歡迎，因為其資料模型能完美契合單向資料流和組件層級結構。

AWS AppSync 讓您能夠連接到自己 AWS 帳戶中的資源，這意味著您擁有並控制自己的數據。這是通過使用數據源來實現的，該服務支持 [Amazon DynamoDB](https://aws.amazon.com/dynamodb/)、[Amazon Elasticsearch](https://aws.amazon.com/elasticsearch-service/) 和 [AWS Lambda](https://aws.amazon.com/lambda/)。這使您能夠將功能（例如 NoSQL 和全文搜索）結合在單一的 GraphQL API 中作為一個架構。這讓您可以混合搭配數據源。AppSync 服務還可以 [從架構進行配置](https://docs.aws.amazon.com/appsync/latest/devguide/provision-from-schema.html)，因此如果您不熟悉 AWS 服務，您可以編寫 GraphQL SDL，點擊一個按鈕，然後自動啟動並運行。

AWS AppSync 中的實時功能是通過 [GraphQL 訂閱與一個眾所周知、基於事件的模式](https://graphql.org/blog/subscriptions-in-graphql-and-relay/) 來控制的。由於 AWS AppSync 中的訂閱是 [在架構上通過 GraphQL 指令控制的](https://docs.aws.amazon.com/appsync/latest/devguide/real-time-data.html)，並且架構可以使用任何數據源，這意味著您可以通過 Amazon DynamoDB 和 Amazon Elasticsearch Service 的數據庫操作觸發通知，或者通過 AWS Lambda 從基礎設施的其他部分觸發通知。

與 AWS Amplify 類似，您可以在 AWS AppSync 上使用 [企業安全功能](https://docs.aws.amazon.com/appsync/latest/devguide/security.html) 來保護您的 GraphQL API。該服務讓您可以快速開始使用 API 密鑰。然而，當您進入生產環境時，它可以轉換為使用 AWS Identity and Access Management (IAM) 或來自 Amazon Cognito 用戶池的 OIDC 令牌。您可以在解析器級別通過類型策略控制訪問。您甚至可以使用邏輯檢查來進行 [細粒度訪問控制](https://docs.aws.amazon.com/appsync/latest/devguide/security.html#fine-grained-access-control) 的運行時檢查，例如檢測用戶是否是特定數據庫資源的所有者。還有能力檢查組成員身份以執行解析器或個別數據庫記錄訪問。

為了幫助 React Native 開發者了解更多關於這些技術的資訊，AWS AppSync 控制台主頁上有一個 [內置的 GraphQL 示例架構](https://docs.aws.amazon.com/appsync/latest/devguide/quickstart.html) 可供啟動。此示例部署了一個 GraphQL 架構，配置了數據庫表，並自動為您連接查詢、變更和訂閱。還有一個功能完整的 [AWS AppSync 的 React Native 示例](https://github.com/aws-samples/aws-mobile-appsync-events-starter-react-native)，它利用了這個內置架構（以及一個 [React 示例](https://github.com/aws-samples/aws-mobile-appsync-events-starter-react)），讓您可以在幾分鐘內啟動客戶端和雲端組件。

使用 `AWSAppSyncClient` 開始非常簡單，它插入了 [Apollo Client](https://github.com/apollographql/apollo-client)。`AWSAppSyncClient` 處理您的 GraphQL API 的安全和簽名、離線功能以及訂閱握手和協商過程：

```
import AWSAppSyncClient from "aws-appsync";
import { Rehydrated } from 'aws-appsync-react';
import { AUTH_TYPE } from "aws-appsync/lib/link/auth-link";

const client = new AWSAppSyncClient({
  url: awsconfig.graphqlEndpoint,
  region: awsconfig.region,
  auth: {type: AUTH_TYPE.API_KEY, apiKey: awsconfig.apiKey}
});
```

AppSync 控制台提供了一個可供下載的配置文件，其中包含您的 GraphQL 端點、AWS 區域和 API 密鑰。然後，您可以將客戶端與 [React Apollo](https://github.com/apollographql/react-apollo) 一起使用：

```
const WithProvider = () => (
  <ApolloProvider client={client}>
      <Rehydrated>
          <App />
      </Rehydrated>
  </ApolloProvider>
);
```

此時，您可以使用標準的 GraphQL 查詢：

```
query ListEvents {
    listEvents{
      items{
        __typename
        id
        name
        where
        when
        description
        comments{
          __typename
          items{
            __typename
            eventId
            commentId
            content
            createdAt
          }
          nextToken
        }
      }
    }
}
```

上面的示例展示了與 AppSync 提供的示例應用架構的查詢。它不僅展示了與 DynamoDB 的交互，還包括數據的分頁（包括加密令牌）以及 `Events` 和 `Comments` 之間的類型關係。由於應用程序配置了 `AWSAppSyncClient`，數據會自動持久化到離線狀態，並在設備重新連接時同步。

您可以通過 [這段視頻](https://www.youtube.com/watch?v=FtkVlIal_m0) 深入了解背後的客戶端技術和一個 React Native 演示。

<iframe
  width="560"
  height="315"
  src="https://www.youtube-nocookie.com/embed/FtkVlIal_m0?rel=0"
  frameborder="0"
  allow="autoplay; encrypted-media"
  allowfullscreen></iframe>

## 反饋

這些函式庫的開發團隊非常期待聽到這些函式庫與服務對您的實際運作情況。他們也想知道我們還能做些什麼來讓您在使用雲端服務開發React和React Native時更加輕鬆。歡迎透過GitHub聯繫AWS Mobile團隊，針對[AWS Amplify](https://github.com/aws/aws-amplify)或[AWS AppSync](https://github.com/aws-samples/aws-mobile-appsync-events-starter-react-native)提供反饋。