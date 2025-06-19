---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程式時，安全性往往被忽視。雖然我們無法打造完全無法攻破的軟體（畢竟就連銀行金庫也可能被入侵），但遭受惡意攻擊或暴露安全漏洞的機率，與你願意投入多少心力來保護應用程式成反比。即使普通掛鎖能被撬開，它仍比櫥櫃掛鉤難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及協助強化應用程式安全的工具。這並非檢查清單，而是各種選項的目錄，每項都能為你的應用程式和用戶提供更進一步的保護。

## 儲存敏感資訊

切勿將敏感 API 金鑰儲存在應用程式代碼中。任何包含在代碼中的內容，都可能被檢查應用程式套件的人以純文字形式存取。雖然 [react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv) 和 [react-native-config](https://github.com/luggit/react-native-config/) 等工具適合用於添加環境變數（如 API 端點），但它們與伺服器端的環境變數不同，後者常包含機密和金鑰。

若必須在應用程式中使用 API 金鑰或密鑰來存取資源，最安全的做法是在應用程式與資源之間建立協調層。這可以是無伺服器函數（例如使用 AWS Lambda 或 Google Cloud Functions），它能轉發請求並附上所需的 API 金鑰或密鑰。伺服器端代碼中的密鑰無法像應用程式代碼中的密鑰那樣被 API 使用者存取。

**對於需持久化的用戶數據，請根據其敏感性選擇適當的儲存類型。** 隨著應用程式的使用，你常需要將數據儲存在裝置上，無論是為了支援離線使用、減少網路請求，或是儲存用戶的存取權杖，讓他們不必每次使用應用程式時都重新驗證。

> **持久化 vs 非持久化** — 持久化數據會寫入裝置的磁碟，讓應用程式能在多次啟動間讀取數據，無需再次發送網路請求或要求用戶重新輸入。但這也使得數據更容易被攻擊者存取。非持久化數據則從不寫入磁碟，因此無數據可存取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage) 是 React Native 的社群維護模組，提供非同步、未加密的鍵值儲存。Async Storage 不會在應用程式間共享：每個應用程式都有自己的沙箱環境，無法存取其他應用程式的數據。

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

React Native 並未內建儲存敏感數據的方式，但 Android 和 iOS 平台已有現成的解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services) 可安全儲存用戶的小型敏感資訊，是存放憑證、權杖、密碼等不應置於 Async Storage 之敏感數據的理想位置。

#### Android - 加密共享偏好設定

[共享偏好設定](https://developer.android.com/reference/android/content/SharedPreferences) 是 Android 上等效的持久化鍵值數據儲存。**共享偏好設定中的數據預設未加密**，但 [加密共享偏好設定](https://developer.android.com/topic/security/data) 封裝了 Shared Preferences 類別，會自動加密鍵與值。

#### Android - 金鑰庫

The [Android Keystore](https://developer.android.com/training/articles/keystore) system lets you store cryptographic keys in a container to make it more difficult to extract from the device.

若要使用 iOS Keychain 服務或 Android Secure Shared Preferences，您可以自行編寫橋接程式碼，或使用封裝這些功能的函式庫（風險自負），這些函式庫會提供統一的 API。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-encrypted-storage](https://github.com/emeraldsanto/react-native-encrypted-storage) - 在 iOS 上使用 Keychain，在 Android 上使用 EncryptedSharedPreferences。
- [react-native-keychain](https://github.com/oblador/react-native-keychain)
- [react-native-sensitive-info](https://github.com/mCodex/react-native-sensitive-info) - 在 iOS 上是安全的，但在 Android 上使用 Shared Preferences（預設不安全）。不過有一個[分支](https://github.com/mCodex/react-native-sensitive-info/tree/keystore)使用 Android Keystore。
  - [redux-persist-sensitive-storage](https://github.com/CodingZeal/redux-persist-sensitive-storage) - 封裝 react-native-sensitive-info 供 Redux 使用。

> **注意不要無意中儲存或暴露敏感資訊。** 這種情況可能意外發生，例如將敏感表單數據保存在 redux state 中，並將整個 state 樹持久化到 Async Storage。或將用戶令牌和個人資訊發送到應用監控服務（如 Sentry 或 Crashlytics）。

## 身份驗證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

Mobile apps have a unique vulnerability that is non-existent in the web: **deep linking**. Deep linking is a way of sending data directly to a native application from an outside source. A deep link looks like `app://` where `app` is your app scheme and anything following the // could be used internally to handle the request.

例如，如果您正在構建一個電子商務應用，可以使用 `app://products/1` 來深度連結到您的應用，並打開 id 為 1 的產品詳情頁面。您可以將其視為類似於網頁上的 URL，但有一個關鍵區別：

深度連結不安全，您不應在其中發送任何敏感資訊。

深度連結不安全的原因是沒有集中註冊 URL 方案的方法。作為應用開發者，您可以通過在 iOS 的 [Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app) 或在 Android 上 [添加 intent](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL 方案。

惡意應用可以通過註冊相同的方案來劫持您的深度連結，從而獲取連結中包含的數據。發送像 `app://products/1` 這樣的內容無害，但發送令牌則是一個安全問題。

當操作系統在打開連結時有兩個或更多應用可供選擇時，Android 會向用戶顯示 [歧義對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog) 並詢問他們選擇使用哪個應用來打開連結。然而在 iOS 上，操作系統會為您做出選擇，因此用戶可能完全不知情。Apple 在後續的 iOS 版本（iOS 11）中採取了措施來解決這個問題，實施了先到先得的原則，儘管這種漏洞仍可以通過其他方式利用，您可以在此[閱讀更多](https://thehackernews.com/2019/07/ios-custom-url-scheme.html)。使用 [通用連結](https://developer.apple.com/ios/universal-links/) 可以在 iOS 中安全地連結到應用內的內容。

### OAuth2 與重定向

OAuth2 認證協議在當今極為流行，被譽為最完善且安全的協議。OpenID Connect 協議亦基於此構建。在 OAuth2 流程中，使用者需透過第三方進行身份驗證。成功完成後，該第三方會重定向回請求應用程式，並附帶一個可兌換為 JWT（[JSON Web Token](https://jwt.io/introduction/)）的驗證碼。JWT 是一種用於在網路各方間安全傳輸資訊的開放標準。

在網頁環境中，此重定向步驟是安全的，因為網頁 URL 具有唯一性保證。但對應用程式而言則不然，如前所述，由於缺乏集中式的 URL 方案註冊機制！為解決此安全隱患，必須透過 PKCE 機制增加額外驗證層。

[PKCE](https://oauth.net/2/pkce/)（發音為 "Pixy"）全稱為 Proof of Key Code Exchange，是 OAuth 2 規範的擴展。該機制透過增加安全層來驗證認證請求與令牌交換請求是否來自同一客戶端。PKCE 採用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密雜湊演算法。SHA 256 能為任意大小的文字或檔案生成唯一「簽章」，其特性如下：

- 無論輸入檔案大小，輸出長度恆定
- 相同輸入必定產生相同結果
- 具單向性（無法逆向推導原始輸入）

此時您將持有兩個數值：

- **code_verifier**：由客戶端生成的大型隨機字串
- **code_challenge**：code_verifier 的 SHA 256 雜湊值

During the initial `/authorize` request, the client also sends the `code_challenge` for the `code_verifier` it keeps in memory. After the authorize request has returned correctly, the client also sends the `code_verifier` that was used to generate the `code_challenge`. The IDP will then calculate the `code_challenge`, see if it matches what was set on the very first `/authorize` request, and only send the access token if the values match.

此機制確保僅有觸發初始授權流程的應用程式能成功兌換驗證碼為 JWT。即使惡意應用程式取得驗證碼，該碼單獨也毫無用處。實際運作範例可參考[此示範](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

適用於原生 OAuth 的推薦函式庫為 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth)。該 SDK 封裝了原生 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 與 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 庫，能與 OAuth2 供應商通訊並支援 PKCE。

> 注意：react-native-app-auth 僅在您的身份供應商支援時方可啟用 PKCE 功能。

![OAuth2 搭配 PKCE 流程圖](/docs/assets/diagram_pkce.svg)

## 網路安全

您的 API 應始終採用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。SSL 加密能防止資料在伺服器傳輸至客戶端過程中被以明文竊取。安全端點可透過 `https://` 開頭（而非 `http://`）識別。

### SSL 憑證綁定

即使使用 https 端點，資料仍有遭攔截風險。在 https 架構下，客戶端僅信任由預裝可信憑證機構（CA）簽署的有效憑證。攻擊者可能透過在使用者裝置安裝惡意根 CA 憑證，使客戶端信任攻擊者簽署的所有憑證。因此，單純依賴憑證驗證仍可能使您暴露於[中間人攻擊](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)風險中。

**SSL 憑證固定**（SSL pinning）是一種可在客戶端使用的技術，用於避免此類攻擊。其原理是在開發階段將一組受信任的憑證清單嵌入（或固定）至客戶端，使得只有由這些受信任憑證簽署的請求才會被接受，任何自簽憑證都將被拒絕。

> 使用 SSL 憑證固定時，需注意憑證的有效期限。憑證通常每 1-2 年會過期，屆時不僅伺服器需更新憑證，應用程式內嵌的憑證也必須同步更新。一旦伺服器憑證更新後，任何仍內嵌舊憑證的應用程式將立即失效。

## 總結

不存在絕對無懈可擊的安全方案，但透過審慎規劃與嚴格執行，能大幅降低應用程式遭受安全漏洞攻擊的風險。請根據應用程式儲存資料的敏感性、用戶規模及駭客入侵可能造成的損害程度，投入相應的安全防護資源。切記：最安全的資料是那些從未被請求取得的資料。