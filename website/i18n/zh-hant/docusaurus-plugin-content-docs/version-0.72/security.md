---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程式時，安全性往往被忽視。確實，要打造完全無法穿透的軟體是不可能的——畢竟我們尚未發明完全無法破解的鎖（銀行金庫仍然會被闖入）。然而，遭受惡意攻擊或暴露於安全漏洞的機率，與你願意投入多少努力來保護應用程式免受此類事件影響成反比。雖然普通的掛鎖可以被撬開，但它仍然比櫥櫃掛鉤難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

在本指南中，你將學習有關儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及幫助你保護應用程式的工具。這不是一份預檢清單——而是一個選項目錄，每個選項都能進一步保護你的應用程式和用戶。

## 儲存敏感資訊

切勿在應用程式代碼中儲存敏感的 API 金鑰。任何包含在代碼中的內容都可能被檢查應用程式套件的人以純文字形式存取。像 [react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv) 和 [react-native-config](https://github.com/luggit/react-native-config/) 這樣的工具非常適合添加環境特定的變數，如 API 端點，但它們不應與伺服器端的環境變數混淆，後者通常包含密鑰和 API 金鑰。

如果必須在應用程式中使用 API 金鑰或密鑰來存取某些資源，最安全的處理方式是構建一個介於應用程式和資源之間的協調層。這可以是一個無伺服器函數（例如使用 AWS Lambda 或 Google Cloud Functions），它可以轉發帶有所需 API 金鑰或密鑰的請求。伺服器端代碼中的密鑰無法像應用程式代碼中的密鑰那樣被 API 使用者存取。

**對於持久化的用戶數據，根據其敏感性選擇正確的儲存類型。** 隨著應用程式的使用，你會經常發現需要在設備上儲存數據，無論是為了支持應用程式離線使用、減少網路請求，還是在會話之間儲存用戶的存取令牌，這樣他們就不必每次使用應用程式時重新驗證。

> **持久化與非持久化** — 持久化數據被寫入設備的磁碟，這使得數據可以在應用程式啟動之間被讀取，而無需再次進行網路請求或要求用戶重新輸入。但這也可能使數據更容易被攻擊者存取。非持久化數據從未被寫入磁碟——因此沒有數據可供存取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage) 是一個由社區維護的 React Native 模組，提供了一個非同步、未加密的鍵值儲存。Async Storage 不在應用程式之間共享：每個應用程式都有自己的沙盒環境，無法存取其他應用程式的數據。

| **Do** use async storage when...              | **Don't** use async storage for... |
| --------------------------------------------- | ---------------------------------- |
| Persisting non-sensitive data across app runs | Token storage                      |
| Persisting Redux state                        | Secrets                            |
| Persisting GraphQL state                      |                                    |
| Storing global app-wide variables             |                                    |

#### 開發者注意事項

<Tabs groupId="guide" queryString defaultValue="web" values={constants.getDevNotesTabs(["web"])}>

<TabItem value="web">

> Async Storage is the React Native equivalent of Local Storage from the web

</TabItem>
</Tabs>

### 安全儲存

React Native 並未內建任何儲存敏感數據的方式。然而，Android 和 iOS 平台已有現有的解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services) 允許你安全地儲存用戶的小塊敏感資訊。這是儲存證書、令牌、密碼以及任何其他不應放在 Async Storage 中的敏感資訊的理想場所。

#### Android - 安全共享偏好設定

[共享偏好設定](https://developer.android.com/reference/android/content/SharedPreferences) 是 Android 上相當於持久化鍵值數據儲存的工具。**共享偏好設定中的數據預設未加密**，但 [加密共享偏好設定](https://developer.android.com/topic/security/data) 封裝了 Android 的 Shared Preferences 類，並自動加密鍵和值。

#### Android - 金鑰庫

The [Android Keystore](https://developer.android.com/training/articles/keystore) system lets you store cryptographic keys in a container to make it more difficult to extract from the device.

若要使用 iOS Keychain 服務或 Android Secure Shared Preferences，您可以自行編寫橋接程式碼，或使用封裝這些功能的函式庫（風險自負），這些函式庫會提供統一的 API。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-encrypted-storage](https://github.com/emeraldsanto/react-native-encrypted-storage) - 在 iOS 上使用 Keychain，在 Android 上使用 EncryptedSharedPreferences。
- [react-native-keychain](https://github.com/oblador/react-native-keychain)
- [react-native-sensitive-info](https://github.com/mCodex/react-native-sensitive-info) - 在 iOS 上是安全的，但在 Android 上使用 Shared Preferences（預設不安全）。不過有一個[分支](https://github.com/mCodex/react-native-sensitive-info/tree/keystore)使用 Android Keystore。
  - [redux-persist-sensitive-storage](https://github.com/CodingZeal/redux-persist-sensitive-storage) - 為 Redux 封裝 react-native-sensitive-info。

> **注意不要無意中儲存或暴露敏感資訊。** 這種情況可能意外發生，例如將敏感表單資料儲存在 redux state 中，並將整個 state 樹持久化到 Async Storage。或將使用者 token 和個人資訊發送到應用程式監控服務（如 Sentry 或 Crashlytics）。

## 認證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

Mobile apps have a unique vulnerability that is non-existent in the web: **deep linking**. Deep linking is a way of sending data directly to a native application from an outside source. A deep link looks like `app://` where `app` is your app scheme and anything following the // could be used internally to handle the request.

例如，如果您正在構建一個電子商務應用程式，可以使用 `app://products/1` 來深度連結到您的應用程式，並打開 id 為 1 的產品詳細頁面。您可以將這些視為類似於網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您不應在其中發送任何敏感資訊。

深度連結不安全的原因是沒有集中註冊 URL scheme 的方法。作為應用程式開發人員，您可以通過[在 Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)（iOS）或[在 Android 上添加 intent](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL scheme。

惡意應用程式可以通過註冊相同的 scheme 來劫持您的深度連結，從而獲取連結中包含的資料。發送像 `app://products/1` 這樣的內容無害，但發送 token 則是一個安全問題。

當作業系統在打開連結時有兩個或更多應用程式可選時，Android 會向用戶顯示[歧義對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog)，並詢問用戶選擇使用哪個應用程式打開連結。然而在 iOS 上，作業系統會為您做出選擇，因此用戶可能完全不知情。Apple 在後續的 iOS 版本（iOS 11）中採取了措施來解決這個問題，實施了先到先得的原則，儘管這種漏洞仍可能以其他方式被利用，您可以[在此](https://thehackernews.com/2019/07/ios-custom-url-scheme.html)閱讀更多相關資訊。使用[通用連結](https://developer.apple.com/ios/universal-links/)可以在 iOS 中安全地連結到應用程式內的內容。

### OAuth2 與重新導向

OAuth2 認證協議在當今極受歡迎，被譽為最完善且安全的協議。OpenID Connect 協議也是基於此協議。在 OAuth2 中，用戶會被要求透過第三方進行認證。成功完成後，該第三方會重定向回請求的應用程式，並附帶一個驗證碼，該驗證碼可兌換為 JWT —— 即 [JSON Web Token](https://jwt.io/introduction/)。JWT 是一種在網絡上安全傳輸資訊的開放標準。

在網絡上，這個重定向步驟是安全的，因為網絡上的 URL 保證是唯一的。但這不適用於應用程式，因為如前所述，沒有集中式的方法來註冊 URL 方案！為了解決這個安全問題，必須添加額外的檢查機制，即 PKCE。

[PKCE](https://oauth.net/2/pkce/)，發音為「Pixy」，全稱為 Proof of Key Code Exchange，是 OAuth 2 規範的擴展。它通過添加一層額外的安全性來驗證認證和令牌交換請求是否來自同一個客戶端。PKCE 使用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密哈希算法。SHA 256 會為任何大小的文本或文件創建一個唯一的「簽名」，但它具有以下特性：

- 無論輸入文件的大小如何，輸出的長度始終相同
- 保證相同的輸入始終產生相同的結果
- 單向性（即無法逆向工程以揭示原始輸入）

現在你有兩個值：

- **code_verifier** —— 客戶端生成的一個大型隨機字符串
- **code_challenge** —— code_verifier 的 SHA 256 哈希值

During the initial `/authorize` request, the client also sends the `code_challenge` for the `code_verifier` it keeps in memory. After the authorize request has returned correctly, the client also sends the `code_verifier` that was used to generate the `code_challenge`. The IDP will then calculate the `code_challenge`, see if it matches what was set on the very first `/authorize` request, and only send the access token if the values match.

這保證了只有觸發初始授權流程的應用程式才能成功將驗證碼兌換為 JWT。因此，即使惡意應用程式獲取了驗證碼，單獨使用它也毫無用處。如需查看實際運作，請參考[此範例](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

一個適用於原生 OAuth 的庫是 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth)。React-native-app-auth 是一個與 OAuth2 提供者通信的 SDK。它封裝了原生的 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 和 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 庫，並支持 PKCE。

> React-native-app-auth 只有在你的身份提供者支持 PKCE 時才能支持 PKCE。

![OAuth2 with PKCE](/docs/assets/diagram_pkce.svg)

## 網絡安全

你的 API 應始終使用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。SSL 加密可防止請求的數據在離開服務器後、到達客戶端前以明文形式被讀取。你可以通過檢查端點是否以 `https://` 開頭（而非 `http://`）來確認其安全性。

### SSL 固定

即使使用 https 端點，你的數據仍可能面臨攔截風險。在使用 https 時，客戶端僅在服務器能提供由可信證書頒發機構（CA）簽名的有效證書時才會信任該服務器，而這些 CA 證書已預裝在客戶端上。攻擊者可能會利用這一點，在用戶設備上安裝惡意的根 CA 證書，從而使客戶端信任所有由攻擊者簽名的證書。因此，僅依賴證書仍可能使你面臨[中間人攻擊](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)的風險。

**SSL 憑證釘選（SSL Pinning）** 是一種可在客戶端使用的技術，用於避免此類攻擊。其原理是在開發階段將一組受信任的憑證嵌入（或「釘選」）至客戶端，使得只有由這些受信任憑證簽署的請求才會被接受，任何自簽憑證都將被拒絕。

> 使用 SSL 憑證釘選時，需注意憑證的有效期限。憑證通常每 1-2 年會過期，屆時不僅伺服器需更新憑證，應用程式也必須同步更新。一旦伺服器憑證更新後，任何仍嵌有舊憑證的應用程式將立即失效。

## 總結

沒有任何方法能絕對保證安全性，但透過謹慎規劃與持續維護，可大幅降低應用程式遭受安全漏洞攻擊的風險。請根據應用程式儲存的資料敏感度、用戶數量，以及駭客入侵可能造成的損害程度，投入相應的安全防護資源。切記：最安全的資料，永遠是那些從未被請求過的資料。