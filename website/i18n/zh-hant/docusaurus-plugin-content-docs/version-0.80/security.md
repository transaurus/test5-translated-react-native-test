---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程式時，安全性往往被忽視。雖然確實無法打造完全無法攻破的軟體（畢竟就連銀行金庫也會被闖入），但遭受惡意攻擊或暴露安全漏洞的機率，與你投入保護應用程式的努力成反比。即使普通掛鎖能被撬開，它仍比櫥櫃掛鉤難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及強化應用安全的工具。這不是一份檢查清單，而是各種選項的目錄，每項都能為你的應用和用戶提供更進一步的保護。

## 儲存敏感資訊

切勿在應用程式碼中儲存敏感的API金鑰。任何包含在程式碼中的內容，都可能被檢查應用套件的人以純文字形式存取。雖然像 [react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv) 和 [react-native-config](https://github.com/luggit/react-native-config/) 這類工具適合用於添加環境變數（如API端點），但它們與伺服器端的環境變數不同，後者常包含密鑰和API金鑰。

若必須在應用中使用API金鑰或密鑰存取資源，最安全的做法是在應用與資源間建立協調層。這可以是無伺服器函數（如使用AWS Lambda或Google Cloud Functions），它能轉發請求並附加所需的API金鑰或密鑰。伺服器端程式碼中的密鑰無法像應用程式碼中的密鑰那樣被API消費者存取。

**對於需持久化的用戶數據，請根據敏感度選擇適當的儲存類型。** 隨著應用使用，你會經常需要將數據儲存在裝置上，無論是為了支援離線使用、減少網路請求，或是保存用戶的存取權杖以避免每次使用時重新驗證。

> **持久化 vs 非持久化** — 持久化數據會寫入裝置磁碟，讓應用能在多次啟動間讀取數據，無需重新發送網路請求或用戶再次輸入。但這也使數據更容易被攻擊者存取。非持久化數據永不寫入磁碟，因此根本無數據可存取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage) 是React Native的社群維護模組，提供非同步、未加密的鍵值儲存。Async Storage不在應用間共享：每個應用都有獨立的沙箱環境，無法存取其他應用的數據。

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

React Native並未內建儲存敏感數據的方式，但Android和iOS平台已有現成解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services) 可安全儲存用戶的小型敏感資訊，是存放憑證、權杖、密碼等不該放入Async Storage之敏感數據的理想位置。

#### Android - 加密共享偏好設定

[共享偏好設定](https://developer.android.com/reference/android/content/SharedPreferences) 是Android的持久化鍵值儲存方案。**預設情況下，共享偏好設定中的數據未加密**，但 [加密共享偏好設定](https://developer.android.com/topic/security/data) 封裝了該類別，會自動加密鍵與值。

#### Android - 金鑰庫

The [Android Keystore](https://developer.android.com/training/articles/keystore) system lets you store cryptographic keys in a container to make it more difficult to extract from the device.

若要使用 iOS Keychain 服務或 Android Secure Shared Preferences，您可以自行編寫橋接程式碼，或使用封裝這些功能的函式庫（需自行承擔風險），這些函式庫會提供統一的 API。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-keychain](https://github.com/oblador/react-native-keychain)

> **請注意不要無意中儲存或暴露敏感資訊。** 這種情況可能意外發生，例如將敏感表單資料儲存在 redux state 中，並將整個 state 樹持久化到 Async Storage。或是將使用者 token 和個人資訊傳送到應用程式監控服務（如 Sentry 或 Crashlytics）。

## 認證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

Mobile apps have a unique vulnerability that is non-existent in the web: **deep linking**. Deep linking is a way of sending data directly to a native application from an outside source. A deep link looks like `app://` where `app` is your app scheme and anything following the // could be used internally to handle the request.

舉例來說，如果您正在開發一個電子商務應用程式，您可以使用 `app://products/1` 來深度連結到您的應用程式，並開啟 id 為 1 的產品詳細頁面。您可以將這些連結想像成網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您不應該在其中傳送任何敏感資訊。

深度連結不安全的原因是沒有集中式的 URL scheme 註冊方法。作為應用程式開發者，您可以透過 [在 Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)（iOS）或 [在 Android 上添加 intent](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何您選擇的 URL scheme。

惡意應用程式可以透過註冊相同的 scheme 來劫持您的深度連結，從而獲取連結中包含的資料。傳送像 `app://products/1` 這樣的連結並無害處，但傳送 token 則會引發安全問題。

當作業系統在開啟連結時有兩個或更多應用程式可供選擇時，Android 會顯示 [歧義對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog) 並要求使用者選擇使用哪個應用程式來開啟連結。然而在 iOS 上，作業系統會替使用者做出選擇，因此使用者可能完全不知情。Apple 在後續的 iOS 版本（iOS 11）中已採取措施解決此問題，實施了先到先得的原則，但此漏洞仍可能以其他方式被利用，您可以在[這裡](https://thehackernews.com/2019/07/ios-custom-url-scheme.html)閱讀更多相關資訊。使用 [universal links](https://developer.apple.com/ios/universal-links/) 可以在 iOS 中安全地連結到應用程式內的內容。

### OAuth2 與重新導向

OAuth2 認證協定現今非常流行，被譽為最完整且安全的協定。OpenID Connect 協定也是基於此協定。在 OAuth2 中，使用者會被要求透過第三方進行認證。成功完成後，第三方會重新導向回請求的應用程式，並帶有一個驗證碼，該驗證碼可兌換為 JWT —— [JSON Web Token](https://jwt.io/introduction/)。JWT 是一種在網路上安全傳輸資訊的開放標準。

在網頁上，這個重新導向步驟是安全的，因為網頁上的 URL 保證是唯一的。但這不適用於應用程式，因為如前所述，沒有集中式的 URL scheme 註冊方法！為了解決這個安全問題，必須添加 PKCE 作為額外的檢查。

[PKCE](https://oauth.net/2/pkce/)，發音為「Pixy」，全名為「Proof of Key Code Exchange」，是 OAuth 2 規範的擴充功能。它透過增加一層安全驗證機制，確保認證請求與令牌交換請求來自同一個客戶端。PKCE 採用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密雜湊演算法。SHA 256 能為任何大小的文字或檔案產生獨特的「簽章」，其特性如下：

- 無論輸入檔案大小，輸出長度固定
- 相同輸入必定產生相同結果
- 單向不可逆（無法從簽章反推原始輸入）

此時您會持有兩個數值：

- **code_verifier** - 由客戶端生成的隨機長字串
- **code_challenge** - code_verifier 的 SHA 256 雜湊值

During the initial `/authorize` request, the client also sends the `code_challenge` for the `code_verifier` it keeps in memory. After the authorize request has returned correctly, the client also sends the `code_verifier` that was used to generate the `code_challenge`. The IDP will then calculate the `code_challenge`, see if it matches what was set on the very first `/authorize` request, and only send the access token if the values match.

此機制確保只有發起初始授權流程的應用程式能成功用驗證碼兌換 JWT。即使惡意應用取得驗證碼，單獨持有也毫無用處。實際運作範例可參考[此示範](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

適用於原生 OAuth 的推薦函式庫是 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth)。該 SDK 封裝了原生 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 與 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 函式庫，可支援 PKCE 機制。

> 注意：react-native-app-auth 的 PKCE 支援度取決於您的身份提供者（Identity Provider）是否實作此功能。

![OAuth2 搭配 PKCE 流程圖](/docs/assets/diagram_pkce.svg)

## 網路安全

API 必須全程使用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。SSL 加密能防止資料在伺服器與客戶端傳輸過程中被以明文竊取。安全端點的識別特徵是網址開頭為 `https://` 而非 `http://`。

### SSL 憑證綁定

僅使用 https 端點仍可能使資料遭攔截。在 https 架構下，客戶端僅信任由預裝可信憑證授權機構（CA）簽署的伺服器憑證。攻擊者可能透過在用戶裝置安裝惡意根 CA 憑證，使客戶端誤認攻擊者簽發的憑證為可信來源，導致傳統憑證驗證機制仍存在[中間人攻擊](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)風險。

**SSL 憑證綁定**技術可從客戶端防範此類攻擊。其原理是在開發階段將受信任的憑證清單嵌入（綁定）至客戶端，使得僅有符合預存憑證的請求會被接受，任何自簽憑證都將被拒絕。

> 實作 SSL 綁定時需注意憑證有效期。憑證通常每 1-2 年會過期，屆時需同步更新伺服器與應用程式內的憑證。若伺服器更新憑證後，未同步更新應用內綁定憑證，將導致應用完全無法運作。

## 總結

雖然沒有萬無一失的安全處理方式，但透過有意識的努力與謹慎，仍能大幅降低應用程式遭受安全漏洞的風險。請根據應用程式中儲存資料的敏感性、用戶數量，以及駭客入侵可能造成的損害程度，投入相應的安全防護資源。切記：最安全的資料就是那些從未被請求過的資料。