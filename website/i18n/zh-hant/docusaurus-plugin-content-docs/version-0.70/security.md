---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程式時，安全性往往被忽視。雖然確實無法打造完全無法攻破的軟體（畢竟連銀行金庫也會被闖入），但遭受惡意攻擊或暴露安全漏洞的機率，與你投入保護應用程式的努力成反比。即使普通掛鎖能被撬開，它仍比櫥櫃掛鉤難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐與工具。這不是一份預檢清單，而是選項目錄，每項都能進一步保護你的應用程式與使用者。

## 儲存敏感資訊

切勿在應用程式碼中儲存敏感的API金鑰。任何包含在程式碼中的內容，都可能被檢查應用套件的人以純文字形式存取。雖然像 [react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv) 和 [react-native-config](https://github.com/luggit/react-native-config/) 這類工具適合用於添加環境變數（如API端點），但它們與伺服器端環境變數不同，後者常包含密鑰與API金鑰。

若必須在應用中使用API金鑰或密鑰存取資源，最安全的做法是在應用與資源間建立協調層。例如使用無伺服器函數（如AWS Lambda或Google Cloud Functions）轉發請求並附加所需金鑰。伺服器端程式碼中的密鑰無法像應用程式碼中的密鑰那樣被API消費者存取。

**針對需持久化的使用者資料，應根據敏感性選擇適當儲存方式。** 應用使用過程中常需在裝置上保存資料，無論是為了支援離線使用、減少網路請求，或是保存使用者存取權杖以避免每次重新驗證。

> **持久化 vs 非持久化** — 持久化資料會寫入裝置磁碟，讓應用能在多次啟動間讀取，無需重新網路請求或要求使用者輸入。但這也使資料更容易被攻擊者存取。非持久化資料永不寫入磁碟，因此不存在被存取的風險！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage) 是React Native社群維護的模組，提供非同步、未加密的鍵值儲存。Async Storage不跨應用共享：每個應用都有獨立的沙箱環境，無法存取其他應用的資料。

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

React Native並未內建儲存敏感資料的方法，但Android和iOS平台已有現成解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services) 可安全儲存使用者的少量敏感資訊，是存放憑證、權杖、密碼等不適合Async Storage資料的理想選擇。

#### Android - 加密Shared Preferences

[Shared Preferences](https://developer.android.com/reference/android/content/SharedPreferences) 是Android的持久化鍵值儲存方案，**預設不加密資料**。但 [加密型Shared Preferences](https://developer.android.com/topic/security/data) 會自動加密鍵與值。

#### Android - Keystore

The [Android Keystore](https://developer.android.com/training/articles/keystore) system lets you store cryptographic keys in a container to make it more difficult to extract from the device.

若要使用 iOS Keychain 服務或 Android Secure Shared Preferences，您可以自行編寫橋接程式碼，或使用封裝這些功能的函式庫（需自行承擔風險），這些函式庫會提供統一的 API。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-encrypted-storage](https://github.com/emeraldsanto/react-native-encrypted-storage) - 在 iOS 上使用 Keychain，在 Android 上使用 EncryptedSharedPreferences。
- [react-native-keychain](https://github.com/oblador/react-native-keychain)
- [react-native-sensitive-info](https://github.com/mCodex/react-native-sensitive-info) - 在 iOS 上是安全的，但在 Android 上使用 Shared Preferences（預設不安全）。不過有一個[分支](https://github.com/mCodex/react-native-sensitive-info/tree/keystore)使用 Android Keystore。
  - [redux-persist-sensitive-storage](https://github.com/CodingZeal/redux-persist-sensitive-storage) - 為 Redux 封裝 react-native-sensitive-info。

> **注意避免無意中儲存或暴露敏感資訊。** 這種情況可能意外發生，例如將敏感表單資料儲存在 redux state 中，並將整個 state 樹持久化到 Async Storage。或將用戶令牌和個人資訊發送到應用監控服務（如 Sentry 或 Crashlytics）。

## 認證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

Mobile apps have a unique vulnerability that is non-existent in the web: **deep linking**. Deep linking is a way of sending data directly to a native application from an outside source. A deep link looks like `app://` where `app` is your app scheme and anything following the // could be used internally to handle the request.

例如，如果您正在開發一個電子商務應用，可以使用 `app://products/1` 來深度連結到您的應用，並打開 id 為 1 的產品詳情頁面。您可以將其視為類似網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您絕不應在其中發送任何敏感資訊。

深度連結不安全的原因是沒有集中註冊 URL 方案的方法。作為應用開發者，您可以通過[在 Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)（iOS）或[在 Android 上添加 intent](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL 方案。

惡意應用可以通過註冊相同的方案來劫持您的深度連結，從而獲取連結中包含的資料。發送像 `app://products/1` 這樣的連結無害，但發送令牌則存在安全隱患。

當作業系統有多個應用可以打開連結時，Android 會顯示[歧義對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog)，讓用戶選擇使用哪個應用打開連結。但在 iOS 上，作業系統會替用戶做出選擇，因此用戶可能完全不知情。Apple 在後續 iOS 版本（iOS 11）中採取了措施解決此問題，實行了先到先得的原則，但此漏洞仍可能以其他方式被利用，詳情可閱讀[此處](https://thehackernews.com/2019/07/ios-custom-url-scheme.html)。使用[通用連結](https://developer.apple.com/ios/universal-links/)可以在 iOS 中安全地連結到應用內的內容。

### OAuth2 與重定向

OAuth2 認證協議在現今極為流行，被譽為最完善且安全的協議。OpenID Connect 協議也是基於此協議。在 OAuth2 中，用戶會被要求透過第三方進行認證。成功完成後，該第三方會重定向回請求的應用程式，並附帶一個驗證碼，此驗證碼可兌換為 JWT — 即 [JSON Web Token](https://jwt.io/introduction/)。JWT 是一種在網路上安全傳輸資訊的開放標準。

在網頁上，這個重定向步驟是安全的，因為網頁上的 URL 保證是唯一的。但這對應用程式來說並非如此，因為如前所述，沒有集中註冊 URL 方案的方法！為了解決這個安全問題，必須以 PKCE 的形式加入額外的檢查。

[PKCE](https://oauth.net/2/pkce/)，發音為「Pixy」，代表 Proof of Key Code Exchange，是 OAuth 2 規範的擴展。這涉及增加一層額外的安全性，以驗證認證和令牌交換請求來自同一個客戶端。PKCE 使用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密哈希演算法。SHA 256 會為任何大小的文字或檔案創建一個獨特的「簽名」，但它具有以下特性：

- 無論輸入檔案大小如何，簽名長度始終相同
- 保證相同的輸入始終產生相同的結果
- 單向性（即無法逆向工程以揭示原始輸入）

現在你有兩個值：

- **code_verifier** — 由客戶端生成的一個大型隨機字串
- **code_challenge** — code_verifier 的 SHA 256 哈希值

During the initial `/authorize` request, the client also sends the `code_challenge` for the `code_verifier` it keeps in memory. After the authorize request has returned correctly, the client also sends the `code_verifier` that was used to generate the `code_challenge`. The IDP will then calculate the `code_challenge`, see if it matches what was set on the very first `/authorize` request, and only send the access token if the values match.

這保證了只有觸發初始授權流程的應用程式才能成功將驗證碼兌換為 JWT。因此，即使惡意應用程式獲得了驗證碼，單獨使用它也毫無用處。要查看實際運作，請參考[此範例](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

一個適用於原生 OAuth 的函式庫是 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth)。React-native-app-auth 是一個與 OAuth2 提供者通訊的 SDK。它封裝了原生的 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 和 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 函式庫，並可支援 PKCE。

> React-native-app-auth 只有在你的身份提供者支援 PKCE 時才能支援 PKCE。

![OAuth2 with PKCE](/docs/assets/diagram_pkce.svg)

## 網路安全

你的 API 應始終使用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。SSL 加密可防止請求的資料在離開伺服器後、到達客戶端前以明文形式被讀取。你可以透過端點是否以 `https://` 而非 `http://` 開頭來判斷其是否安全。

### SSL 固定

使用 https 端點仍可能使你的資料容易遭到攔截。在 https 中，客戶端僅在伺服器能提供由客戶端預安裝的可信憑證授權機構簽署的有效憑證時才會信任伺服器。攻擊者可利用這一點，將惡意的根 CA 憑證安裝到用戶的裝置上，使客戶端信任由攻擊者簽署的所有憑證。因此，僅依賴憑證仍可能使你容易遭受[中間人攻擊](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)。

**SSL 憑證固定**（SSL pinning）是一種可在客戶端使用的技術，用以避免此類攻擊。其原理是在開發階段將一組受信任的憑證清單嵌入（或固定）至客戶端，使得只有由這些受信任憑證簽署的請求才會被接受，任何自簽憑證都將被拒絕。

> 使用 SSL 憑證固定時，需注意憑證的有效期限。憑證通常每 1-2 年會過期，屆時不僅伺服器需更新憑證，應用程式內嵌的憑證也必須同步更新。一旦伺服器憑證更新後，內嵌舊版憑證的應用程式將立即無法運作。

## 總結

雖然不存在絕對安全的防護方案，但透過謹慎規劃與持續維護，能大幅降低應用程式遭受安全漏洞的風險。請根據應用程式儲存的資料敏感度、用戶規模及駭客入侵可能造成的損害，投入相應程度的安全防護資源。切記：最安全的資料，往往是那些從未被請求取得的資料。