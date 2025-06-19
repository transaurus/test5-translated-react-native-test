---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程式時，安全性往往被忽視。雖然確實無法打造完全無懈可擊的軟體（畢竟就連銀行金庫也會被攻破），但遭受惡意攻擊或暴露安全漏洞的機率，與你投入的防護努力成反比。就像普通掛鎖雖能被撬開，但突破難度仍遠高於櫃門掛鉤！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐與工具。這不是檢查清單，而是選項目錄——每項都能為你的應用和用戶提供更深層防護。

## 儲存敏感資訊

切勿在應用程式碼中儲存敏感API金鑰。任何寫入程式碼的內容都可能被檢查應用套件的人以明文獲取。雖然[react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv)和[react-native-config](https://github.com/luggit/react-native-config/)等工具適合管理API端點等環境變數，但它們不同於伺服器端環境變數（後者常包含密鑰）。

若必須在應用中使用API金鑰存取資源，最安全的做法是建立應用與資源間的協調層（如AWS Lambda或Google Cloud Functions的無伺服器函數）。伺服器端程式碼中的密鑰不會像應用程式碼中的密鑰那樣被API使用者直接獲取。

**根據敏感度選擇合適的持久化儲存類型。** 無論是支援離線使用、減少網路請求，或是保存用戶存取權杖避免重複登入，應用運行時常需在設備上儲存資料。

> **持久化 vs 非持久化** — 持久化資料會寫入設備磁碟，使應用重啟後仍能讀取，但同時增加被攻擊者存取的風險。非持久化資料永不寫入磁碟——自然無數據可竊！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage)是React Native社群維護的異步、未加密鍵值儲存模組。各應用的Async Storage處於獨立沙箱環境，無法跨應用存取數據。

| **Do** use async storage when...              | **Don't** use async storage for... |
| --------------------------------------------- | ---------------------------------- |
| Persisting non-sensitive data across app runs | Token storage                      |
| Persisting Redux state                        | Secrets                            |
| Persisting GraphQL state                      |                                    |
| Storing global app-wide variables             |                                    |

#### 開發者須知

<Tabs groupId="guide" queryString defaultValue="web" values={constants.getDevNotesTabs(["web"])}>

<TabItem value="web">

> Async Storage is the React Native equivalent of Local Storage from the web

</TabItem>
</Tabs>

### 安全儲存

React Native未內建敏感數據儲存方案，但iOS和Android平台已有現成解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services)可安全儲存用戶的小型敏感數據，適合保存憑證、權杖、密碼等不應存入Async Storage的資訊。

#### Android - 加密Shared Preferences

[Shared Preferences](https://developer.android.com/reference/android/content/SharedPreferences)是Android的持久化鍵值儲存方案，**預設不加密**。但[加密版Shared Preferences](https://developer.android.com/topic/security/data)會自動加密鍵與值。

#### Android - Keystore

The [Android Keystore](https://developer.android.com/training/articles/keystore) system lets you store cryptographic keys in a container to make it more difficult to extract from the device.

若要使用 iOS Keychain 服務或 Android Secure Shared Preferences，您可以自行編寫橋接程式碼，或選擇使用封裝這些功能的函式庫（需自行承擔風險），這些函式庫會提供統一的 API。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-encrypted-storage](https://github.com/emeraldsanto/react-native-encrypted-storage) - 在 iOS 上使用 Keychain，在 Android 上使用 EncryptedSharedPreferences。
- [react-native-keychain](https://github.com/oblador/react-native-keychain)
- [react-native-sensitive-info](https://github.com/mCodex/react-native-sensitive-info) - 在 iOS 上是安全的，但在 Android 上使用 Shared Preferences（預設不安全）。不過有一個[分支](https://github.com/mCodex/react-native-sensitive-info/tree/keystore)使用 Android Keystore。
  - [redux-persist-sensitive-storage](https://github.com/CodingZeal/redux-persist-sensitive-storage) - 為 Redux 封裝 react-native-sensitive-info。

> **注意避免無意中儲存或暴露敏感資訊。** 這種情況可能意外發生，例如將敏感表單資料儲存在 redux state 中，並將整個 state 樹持久化到 Async Storage。或將用戶令牌和個人資訊發送到應用監控服務（如 Sentry 或 Crashlytics）。

## 認證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

Mobile apps have a unique vulnerability that is non-existent in the web: **deep linking**. Deep linking is a way of sending data directly to a native application from an outside source. A deep link looks like `app://` where `app` is your app scheme and anything following the // could be used internally to handle the request.

例如，如果您正在構建一個電子商務應用，可以使用 `app://products/1` 來深度連結到您的應用，並打開 id 為 1 的產品詳情頁面。您可以將這些連結視為類似於網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您絕不應在其中發送任何敏感資訊。

深度連結不安全的原因是沒有集中註冊 URL 方案的方法。作為應用開發者，您可以通過在 iOS 的 [Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app) 或在 Android 上 [添加 intent](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL 方案。

惡意應用可以通過註冊相同的方案來劫持您的深度連結，從而獲取連結中包含的數據。發送類似 `app://products/1` 的連結無害，但發送令牌則存在安全隱患。

當操作系統在打開連結時有兩個或更多應用可供選擇時，Android 會向用戶顯示 [歧義對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog) 並詢問使用哪個應用打開連結。然而在 iOS 上，操作系統會替用戶做出選擇，因此用戶可能完全不知情。Apple 在後續 iOS 版本（iOS 11）中採取了措施來解決此問題，實施了先到先得的原則，儘管此漏洞仍可能以其他方式被利用，您可以在此 [閱讀更多](https://thehackernews.com/2019/07/ios-custom-url-scheme.html)。使用 [通用連結](https://developer.apple.com/ios/universal-links/) 可以在 iOS 中安全地連結到應用內的內容。

### OAuth2 與重定向

OAuth2 認證協議在當今極為流行，被譽為最完善且安全的協議。OpenID Connect 協議也是基於此。在 OAuth2 中，用戶需透過第三方進行身份驗證。成功完成後，該第三方會重定向回請求應用程式，並附帶一個可兌換為 JWT（[JSON Web Token](https://jwt.io/introduction/)）的驗證碼。JWT 是一種在網絡上安全傳輸資訊的開放標準。

在網絡上，此重定向步驟是安全的，因為網址具有唯一性保證。但這不適用於應用程式，如前所述，應用程式沒有集中式的網址方案註冊機制！為解決此安全問題，必須透過 PKCE 加入額外的檢查。

[PKCE](https://oauth.net/2/pkce/)（發音為「Pixy」）代表 Proof of Key Code Exchange，是 OAuth 2 規範的擴展。它透過加入一層額外的安全性來驗證身份驗證和令牌交換請求是否來自同一客戶端。PKCE 使用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密雜湊演算法。SHA 256 會為任何大小的文字或檔案產生唯一的「簽章」，其特性如下：

- 無論輸入檔案大小，輸出長度固定
- 相同輸入必定產生相同結果
- 單向性（無法逆向推導原始輸入）

此時你會得到兩個值：

- **code_verifier**：由客戶端生成的大型隨機字串
- **code_challenge**：code_verifier 的 SHA 256 雜湊值

During the initial `/authorize` request, the client also sends the `code_challenge` for the `code_verifier` it keeps in memory. After the authorize request has returned correctly, the client also sends the `code_verifier` that was used to generate the `code_challenge`. The IDP will then calculate the `code_challenge`, see if it matches what was set on the very first `/authorize` request, and only send the access token if the values match.

這確保只有觸發初始授權流程的應用程式才能成功將驗證碼兌換為 JWT。因此，即使惡意應用程式取得驗證碼，單獨使用也毫無價值。如需實際範例，請參閱[此示例](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

若需原生 OAuth 解決方案，可考慮使用 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth)。此 SDK 封裝了原生 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 和 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 函式庫，能與 OAuth2 提供者通訊並支援 PKCE。

> 僅當你的身份提供者支援時，react-native-app-auth 才能啟用 PKCE 功能。

![OAuth2 with PKCE](/docs/assets/diagram_pkce.svg)

## 網絡安全

你的 API 應始終使用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。SSL 加密能防止資料在離開伺服器後至抵達客戶端前以明文形式被讀取。安全端點的識別特徵是其以 `https://` 開頭（而非 `http://`）。

### SSL 憑證綁定

即使使用 https 端點，資料仍可能遭攔截。在 https 中，客戶端僅在伺服器能提供由可信憑證機構（CA）簽署的有效憑證時才會信任該伺服器，且該 CA 需預先安裝於客戶端。攻擊者可透過在使用者裝置上安裝惡意根 CA 憑證來濫用此機制，使客戶端信任攻擊者簽署的所有憑證。因此，僅依賴憑證仍可能使你暴露於[中間人攻擊](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)風險中。

**SSL 憑證釘選（SSL Pinning）** 是一種可在客戶端使用的技術，用於避免此類攻擊。其原理是在開發階段將一組受信任的憑證嵌入（或「釘選」）至客戶端，使得只有由這些受信任憑證簽署的請求才會被接受，任何自簽憑證都將被拒絕。

> 使用 SSL 憑證釘選時，需注意憑證的有效期限。憑證通常每 1-2 年會過期，屆時不僅伺服器端需更新憑證，應用程式內嵌的憑證也必須同步更新。一旦伺服器端的憑證更新後，任何仍內嵌舊憑證的應用程式將立即無法運作。

## 總結

沒有任何方法能絕對保證安全性，但透過謹慎規劃與持續關注，可大幅降低應用程式遭受安全攻擊的可能性。請根據應用程式儲存的資料敏感性、用戶規模及駭客入侵可能造成的損害，投入相應的安全防護資源。切記：最安全的資料，往往是那些從未被請求過的資料。