---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程式時，安全性往往被忽視。雖然確實無法打造完全無懈可擊的軟體（畢竟銀行的保險庫也仍會被攻破），但遭受惡意攻擊或暴露於安全漏洞的機率，與你願意投入多少心力來保護應用程式成反比。普通的掛鎖雖能被撬開，但比起櫃鉤仍難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及協助強化應用程式安全的工具。這並非起飛前檢查清單，而是各種選項的目錄，每項都能為你的應用程式和用戶提供更多保護。

## 儲存敏感資訊

切勿將敏感API金鑰儲存在應用程式代碼中。任何包含在代碼中的內容，都可能被檢查應用套件的人以純文字形式存取。雖然像[react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv)和[react-native-config](https://github.com/luggit/react-native-config/)這類工具很適合用於添加環境變數（如API端點），但它們不應與伺服器端的環境變數混淆，後者常包含密鑰和API金鑰。

若必須透過應用程式存取某項資源的API金鑰或密鑰，最安全的處理方式是建立應用程式與資源間的協調層。這可以是無伺服器函數（例如使用AWS Lambda或Google Cloud Functions），它能轉發請求並附上所需的API金鑰或密鑰。伺服器端代碼中的密鑰不會像應用程式代碼中的密鑰那樣被API使用者存取。

**對於需持久化的用戶數據，請根據其敏感性選擇合適的儲存類型。** 隨著應用程式的使用，你常會需要將數據儲存在裝置上，無論是為了支援離線使用、減少網路請求，或是儲存用戶的存取權杖以便在會話間保持登入狀態。

> **持久化 vs 非持久化** — 持久化數據會被寫入裝置的磁碟，讓應用程式能在不同啟動間讀取，無需再次透過網路請求或要求用戶重新輸入。但這也使得數據更容易被攻擊者存取。非持久化數據則從不寫入磁碟，因此無數據可被存取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage)是React Native的社群維護模組，提供非同步、未加密的鍵值儲存。Async Storage不在應用程式間共享：每個應用程式都有自己的沙箱環境，無法存取其他應用程式的數據。

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

React Native並未內建任何儲存敏感數據的方式，但Android和iOS平台已有現成解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services)允許你安全地儲存用戶的小塊敏感資訊。這是儲存憑證、權杖、密碼等不應放在Async Storage中的敏感數據的理想位置。

#### Android - 安全共享偏好設定

[共享偏好設定](https://developer.android.com/reference/android/content/SharedPreferences)是Android上等效的持久化鍵值數據儲存。**共享偏好設定中的數據預設未加密**，但[加密共享偏好設定](https://developer.android.com/topic/security/data)封裝了Android的SharedPreferences類，會自動加密鍵與值。

#### Android - 金鑰庫

The [Android Keystore](https://developer.android.com/training/articles/keystore) system lets you store cryptographic keys in a container to make it more difficult to extract from the device.

若要使用 iOS Keychain 服務或 Android Secure Shared Preferences，您可以自行編寫橋接程式碼，或使用封裝這些功能的函式庫（但需自行承擔風險），這些函式庫會提供統一的 API。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-keychain](https://github.com/oblador/react-native-keychain)

> **請注意無意間儲存或暴露敏感資訊的情況。** 這種情況可能意外發生，例如將敏感表單資料儲存在 redux state 中，並將整個 state 樹持久化到 Async Storage。或將使用者令牌和個人資訊傳送到應用程式監控服務（如 Sentry 或 Crashlytics）。

## 身份驗證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

行動應用程式有一個在網頁上不存在的獨特漏洞：**深度連結**。深度連結是一種從外部來源直接將資料傳送到原生應用程式的方式。深度連結看起來像 `app://`，其中 `app` 是您的應用程式 scheme，而 // 之後的任何內容都可用於內部處理請求。

例如，如果您正在構建一個電子商務應用程式，您可以使用 `app://products/1` 來深度連結到您的應用程式，並打開 id 為 1 的產品詳細頁面。您可以將這些連結想像成網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您不應在其中傳送任何敏感資訊。

深度連結不安全的原因是沒有集中註冊 URL scheme 的方法。作為應用程式開發人員，您可以通過 [在 Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)（iOS）或 [在 Android 上添加 intent](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL scheme。

惡意應用程式可以通過註冊相同的 scheme 來劫持您的深度連結，從而獲取連結中包含的資料。傳送像 `app://products/1` 這樣的內容並無害處，但傳送令牌則是一個安全問題。

當作業系統在打開連結時有兩個或更多應用程式可選擇時，Android 會向使用者顯示 [歧義對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog)，並詢問他們選擇使用哪個應用程式來打開連結。然而，在 iOS 上，作業系統會為您做出選擇，因此使用者可能完全不知情。Apple 在後續的 iOS 版本（iOS 11）中已採取措施解決此問題，實施了先到先得的原則，儘管此漏洞仍可能以其他方式被利用，您可以 [在此](https://thehackernews.com/2019/07/ios-custom-url-scheme.html) 閱讀更多相關資訊。使用 [通用連結](https://developer.apple.com/ios/universal-links/) 可以在 iOS 中安全地連結到應用程式內的內容。

### OAuth2 與重新導向

OAuth2 身份驗證協議現今非常流行，被譽為最完整且安全的協議。OpenID Connect 協議也基於此。在 OAuth2 中，使用者被要求通過第三方進行身份驗證。成功完成後，此第三方會重新導向回請求的應用程式，並帶有一個驗證碼，該驗證碼可兌換為 JWT —— [JSON Web Token](https://jwt.io/introduction/)。JWT 是一種在網路上安全傳輸資訊的開放標準。

在網頁上，這個重新導向步驟是安全的，因為網頁上的 URL 保證是唯一的。但這對應用程式來說並不成立，因為如前所述，沒有集中註冊 URL scheme 的方法！為了解決這個安全問題，必須以 PKCE 的形式添加額外的檢查。

[PKCE](https://oauth.net/2/pkce/)（發音為「Pixy」）全稱為「Proof of Key Code Exchange」，是 OAuth 2 規範的擴充功能。它透過增加一層安全驗證機制，確保認證請求與令牌交換請求來自同一客戶端。PKCE 採用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密雜湊演算法，該演算法能為任意大小的文字或檔案產生獨特「簽章」，其特性如下：

- 無論輸入檔案大小，輸出長度固定
- 相同輸入必定產生相同結果
- 具不可逆性（無法逆向推導原始輸入）

流程中會產生兩個關鍵值：

- **code_verifier**：由客戶端生成的隨機長字串
- **code_challenge**：code_verifier 的 SHA 256 雜湊值

During the initial `/authorize` request, the client also sends the `code_challenge` for the `code_verifier` it keeps in memory. After the authorize request has returned correctly, the client also sends the `code_verifier` that was used to generate the `code_challenge`. The IDP will then calculate the `code_challenge`, see if it matches what was set on the very first `/authorize` request, and only send the access token if the values match.

此機制確保僅有發起初始授權流程的應用程式能成功兌換驗證碼取得 JWT。即使惡意應用取得驗證碼，單獨持有也毫無用處。實作範例可參考[此示範](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

適用於原生 OAuth 的推薦函式庫為 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth)。此 SDK 封裝了原生 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 與 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 庫，可支援 PKCE 機制。

> 注意：react-native-app-auth 的 PKCE 支援需視您的身份提供者（Identity Provider）是否實作該功能而定。

![OAuth2 搭配 PKCE 流程圖](/docs/assets/diagram_pkce.svg)

## 網路安全

API 應始終採用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。此加密技術能防止資料在伺服器與客戶端傳輸過程中被明文竊取，安全端點可透過 `https://` 開頭（而非 `http://`）識別。

### SSL 憑證綁定

Using https endpoints could still leave your data vulnerable to interception. With https, the client will only trust the server if it can provide a valid certificate that is signed by a trusted Certificate Authority that is pre-installed on the client. An attacker could take advantage of this by installing a malicious root CA certificate to the user’s device, so the client would trust all certificates that are signed by the attacker. Thus, relying on certificates alone could still leave you vulnerable to a [man-in-the-middle attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).

**SSL 憑證綁定**技術可防範此類攻擊。其原理是在開發階段將受信任的憑證清單嵌入（綁定）至客戶端，使應用程式僅接受預存憑證簽署的請求，拒絕任何自簽憑證。

> 實作 SSL 綁定時需注意憑證有效期。伺服器憑證通常每 1-2 年會過期，更新時需同步更新應用內嵌的憑證。若伺服器更新憑證後，仍使用舊憑證的應用將立即失效。

## 總結

雖然沒有萬無一失的安全處理方式，但透過有意識的努力和謹慎態度，仍能大幅降低應用程式遭受安全漏洞的可能性。請根據應用程式中儲存資料的敏感度、用戶數量，以及駭客入侵帳戶可能造成的損害程度，投入相應的安全資源。切記：從一開始就不請求的資訊，要取得它的難度會高得多。