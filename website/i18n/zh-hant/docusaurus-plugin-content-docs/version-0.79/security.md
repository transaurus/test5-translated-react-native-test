---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程式時，安全性往往被忽視。雖然確實無法打造完全無懈可擊的軟體（畢竟就連銀行金庫也可能被攻破），但遭受惡意攻擊或暴露安全漏洞的機率，與你投入應用程式防護的努力成反比。即使普通掛鎖能被撬開，它仍比簡易掛鉤難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及強化應用程式安全的工具。這並非事前檢查清單，而是提供多種選擇方案，每項都能進一步保護你的應用程式與使用者。

## 儲存敏感資訊

切勿將敏感API金鑰儲存在應用程式代碼中。任何包含在代碼裡的內容，都可能被檢查應用套件的人以明文形式存取。雖然像[react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv)和[react-native-config](https://github.com/luggit/react-native-config/)這類工具適合用於添加環境變數（如API端點），但它們與伺服器端環境變數不同，後者常包含機密資訊和API金鑰。

若必須在應用程式中使用API金鑰或密鑰存取資源，最安全的做法是在應用程式與資源間建立協調層。例如使用無伺服器函數（如AWS Lambda或Google Cloud Functions）來轉發請求並附加所需API金鑰。伺服器端代碼中的密鑰不會像應用程式代碼中的密鑰那樣被API使用者輕易取得。

**針對需持久化的使用者數據，應根據敏感程度選擇合適的儲存方式。** 使用應用程式時，常需要將數據保存在裝置上（無論是為了支援離線使用、減少網路請求，或是保存使用者存取權杖以避免每次重新驗證）。

> **持久化 vs 非持久化** — 持久化數據會寫入裝置磁碟，讓應用程式在多次啟動間都能讀取，無需重新網路請求或要求使用者輸入。但這也使得數據更易被攻擊者存取。非持久化數據永不寫入磁碟，自然無數據可竊取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage)是React Native社群維護的模組，提供非同步、未加密的鍵值儲存。Async Storage不跨應用程式共享：每個應用都有獨立的沙箱環境，無法存取其他應用的數據。

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

React Native原生未內建敏感數據儲存方案，但Android和iOS平台已有現成解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services)可安全儲存使用者的敏感小數據，適合存放憑證、權杖、密碼等不應存入Async Storage的資訊。

#### Android - 加密共享偏好設定

[共享偏好設定](https://developer.android.com/reference/android/content/SharedPreferences)是Android的持久化鍵值儲存方案，**預設未加密**。但[加密版共享偏好設定](https://developer.android.com/topic/security/data)會自動加密鍵與值。

#### Android - 金鑰庫

The [Android Keystore](https://developer.android.com/training/articles/keystore) system lets you store cryptographic keys in a container to make it more difficult to extract from the device.

若要使用 iOS Keychain 服務或 Android Secure Shared Preferences，您可以自行編寫橋接程式碼，或使用封裝這些功能的函式庫（風險自負），這些函式庫會提供統一的 API。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-keychain](https://github.com/oblador/react-native-keychain)

> **請注意無意中儲存或暴露敏感資訊的情況。** 這種情況可能意外發生，例如將敏感表單資料儲存在 redux 狀態中，並將整個狀態樹持久化到 Async Storage 中。或將使用者令牌和個人資訊發送到應用程式監控服務（如 Sentry 或 Crashlytics）。

## 認證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

Mobile apps have a unique vulnerability that is non-existent in the web: **deep linking**. Deep linking is a way of sending data directly to a native application from an outside source. A deep link looks like `app://` where `app` is your app scheme and anything following the // could be used internally to handle the request.

例如，如果您正在開發一個電子商務應用程式，您可以使用 `app://products/1` 來深度連結到您的應用程式，並打開 id 為 1 的產品詳細頁面。您可以將這些連結視為網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您絕不應該在其中發送任何敏感資訊。

深度連結不安全的原因是沒有集中註冊 URL 方案的方法。作為應用程式開發人員，您可以通過 [在 Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)（iOS）或 [在 Android 上添加意圖](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL 方案。

惡意應用程式可以通過註冊相同的方案來劫持您的深度連結，從而獲取連結中包含的資料。發送像 `app://products/1` 這樣的連結無害，但發送令牌則是一個安全問題。

當作業系統在打開連結時有兩個或更多應用程式可供選擇時，Android 會向用戶顯示 [歧義對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog)，並詢問用戶選擇使用哪個應用程式來打開連結。然而，在 iOS 上，作業系統會為您做出選擇，因此用戶可能完全不知情。Apple 在後續的 iOS 版本（iOS 11）中已採取措施解決此問題，實施了先到先得的原則，儘管此漏洞仍可能以其他方式被利用，您可以 [在此](https://thehackernews.com/2019/07/ios-custom-url-scheme.html) 閱讀更多相關資訊。使用 [通用連結](https://developer.apple.com/ios/universal-links/) 可以在 iOS 中安全地連結到應用程式內的內容。

### OAuth2 與重新導向

OAuth2 認證協議現今非常流行，被譽為最完整且安全的協議。OpenID Connect 協議也基於此。在 OAuth2 中，用戶被要求通過第三方進行認證。成功完成後，該第三方會重新導向回請求的應用程式，並帶有一個驗證碼，該驗證碼可兌換為 JWT —— [JSON Web Token](https://jwt.io/introduction/)。JWT 是一種在網路上安全傳輸資訊的開放標準。

在網頁上，這個重新導向步驟是安全的，因為網頁上的 URL 保證是唯一的。但這對應用程式來說並非如此，因為如前所述，沒有集中註冊 URL 方案的方法！為了解決這個安全問題，必須以 PKCE 的形式添加額外的檢查。

[PKCE](https://oauth.net/2/pkce/)（發音為「Pixy」）全稱為「Proof of Key Code Exchange」，是OAuth 2規範的擴展功能。它通過增加一層安全驗證機制，確保認證請求與令牌交換請求來自同一客戶端。PKCE採用[SHA 256](https://www.movable-type.co.uk/scripts/sha256.html)加密哈希算法，該算法能為任意大小的文本或文件生成獨特的「簽名」，其特性包括：

- 無論輸入文件大小，輸出長度固定
- 相同輸入必定產生相同結果
- 單向不可逆（無法通過簽名反推原始輸入）

此時您將獲得兩個值：

- **code_verifier** - 由客戶端生成的隨機長字符串
- **code_challenge** - code_verifier的SHA 256哈希值

During the initial `/authorize` request, the client also sends the `code_challenge` for the `code_verifier` it keeps in memory. After the authorize request has returned correctly, the client also sends the `code_verifier` that was used to generate the `code_challenge`. The IDP will then calculate the `code_challenge`, see if it matches what was set on the very first `/authorize` request, and only send the access token if the values match.

此機制確保只有發起初始授權流程的應用程序能成功兌換驗證碼獲取JWT。即使惡意應用截獲驗證碼，單獨持有也毫無用處。實際運作範例可參考[此演示](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

推薦使用的原生OAuth庫是[react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth)。該SDK封裝了原生[AppAuth-iOS](https://github.com/openid/AppAuth-iOS)和[AppAuth-Android](https://github.com/openid/AppAuth-Android)庫，可支持PKCE機制。

> 注意：react-native-app-auth的PKCE功能需依賴身份提供者（IDP）的支持。

![OAuth2 with PKCE](/docs/assets/diagram_pkce.svg)

## 網絡安全

API必須始終使用[SSL加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。SSL能防止數據在服務器與客戶端傳輸過程中被明文竊取。安全端點的識別特徵是採用`https://`開頭而非`http://`。

### SSL憑證綁定

僅使用https仍可能使數據面臨攔截風險。在https協議下，客戶端僅信任由預裝可信證書頒發機構（CA）簽署的服務器證書。攻擊者可通過在用戶設備安裝惡意根CA證書，使客戶端信任攻擊者簽發的所有證書，導致單純依賴證書驗證仍可能遭受[中間人攻擊](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)。

**SSL憑證綁定**技術可通過客戶端預置可信證書列表來防範此類攻擊。開發階段將可信證書嵌入（綁定）至客戶端後，僅攜帶這些證書簽名的請求會被接受，自簽名證書將被拒絕。

> 實施SSL綁定時需注意證書有效期。證書通常1-2年過期，服務器更新證書後，內嵌舊證書的應用將立即失效，必須同步更新應用內證書。

## 總結

雖然沒有萬無一失的安全處理方式，但透過有意識的努力和謹慎態度，仍能大幅降低應用程式遭受安全漏洞的可能性。請根據應用程式中儲存資料的敏感性、用戶數量，以及駭客入侵可能造成的損害程度，投入相應比例的安全防護資源。切記：那些從未被請求過的資訊，往往最難被竊取。