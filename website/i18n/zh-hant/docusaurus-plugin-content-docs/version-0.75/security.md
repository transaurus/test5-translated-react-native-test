---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在開發應用程式時，安全性經常被忽視。雖然打造完全無法被攻破的軟體確實是不可能的——畢竟我們至今仍未發明出完全無法破解的鎖（銀行的保險庫也還是會被闖入）。然而，遭受惡意攻擊或暴露於安全漏洞的機率，與您願意投入多少心力來保護應用程式免受此類事件影響成反比。普通的掛鎖雖然可以被撬開，但比起櫥櫃掛鉤仍然難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及協助強化應用程式安全的工具。這並非起飛前的檢查清單——而是一系列選項的目錄，每個選項都能進一步保護您的應用程式與使用者。

## 儲存敏感資訊

切勿將敏感的 API 金鑰儲存在應用程式代碼中。任何包含在代碼裡的內容，都可能被檢查應用程式套件的人以純文字形式存取。像 [react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv) 和 [react-native-config](https://github.com/luggit/react-native-config/) 這類工具很適合用來添加環境特定的變數（如 API 端點），但它們不應與伺服器端的環境變數混淆，後者常包含密鑰和 API 金鑰。

若必須透過 API 金鑰或密鑰從應用程式存取某些資源，最安全的處理方式是建立一個位於應用程式與資源之間的協調層。這可以是無伺服器函數（例如使用 AWS Lambda 或 Google Cloud Functions），它能轉發帶有所需 API 金鑰或密鑰的請求。伺服器端代碼中的密鑰無法像應用程式代碼中的密鑰那樣被 API 消費者存取。

**對於需持久化的使用者資料，請根據其敏感性選擇適當的儲存類型。** 隨著應用程式的使用，您常會發現需要將資料儲存在裝置上，無論是為了支援離線使用、減少網路請求，或是為了在使用者會話之間保存其存取權杖，避免每次使用應用程式時都需重新驗證。

> **持久化 vs 非持久化** — 持久化資料會被寫入裝置的磁碟，這讓資料能在應用程式多次啟動間被讀取，無需再次發送網路請求獲取或要求使用者重新輸入。但這也使得資料更容易被攻擊者存取。非持久化資料則從不會寫入磁碟——因此根本沒有資料可供存取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage) 是 React Native 的一個由社群維護的模組，提供非同步、未加密的鍵值儲存。Async Storage 不會在應用程式之間共享：每個應用程式都有自己的沙箱環境，無法存取其他應用程式的資料。

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

React Native 並未內建任何儲存敏感資料的方式。不過，Android 和 iOS 平台已有現成的解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services) 允許您安全地儲存使用者的少量敏感資訊。這是儲存憑證、權杖、密碼以及其他不應放在 Async Storage 中的敏感資料的理想場所。

#### Android - 加密共享偏好設定

[共享偏好設定](https://developer.android.com/reference/android/content/SharedPreferences) 是 Android 上等效的持久化鍵值資料儲存方式。**共享偏好設定中的資料預設並未加密**，但 [加密共享偏好設定](https://developer.android.com/topic/security/data) 封裝了 Android 的 SharedPreferences 類別，並自動加密鍵與值。

#### Android - 金鑰庫

The [Android Keystore](https://developer.android.com/training/articles/keystore) system lets you store cryptographic keys in a container to make it more difficult to extract from the device.

若要使用 iOS Keychain 服務或 Android Secure Shared Preferences，您可以自行編寫橋接程式碼，或使用封裝這些功能的函式庫（需自行承擔風險），這些函式庫會提供統一的 API。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-keychain](https://github.com/oblador/react-native-keychain)

> **請注意無意中儲存或暴露敏感資訊的情況。** 這種情況可能意外發生，例如將敏感表單資料儲存在 redux 狀態中，並將整個狀態樹持久化到 Async Storage 中。或是將使用者令牌和個人資訊發送到應用程式監控服務（如 Sentry 或 Crashlytics）。

## 認證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

Mobile apps have a unique vulnerability that is non-existent in the web: **deep linking**. Deep linking is a way of sending data directly to a native application from an outside source. A deep link looks like `app://` where `app` is your app scheme and anything following the // could be used internally to handle the request.

例如，如果您正在開發一個電子商務應用程式，您可以使用 `app://products/1` 來深度連結到您的應用程式，並打開 id 為 1 的產品詳細頁面。您可以將這些連結視為網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您不應在其中發送任何敏感資訊。

深度連結不安全的原因是沒有集中註冊 URL 方案的方法。作為應用程式開發者，您可以通過 [在 Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)（iOS）或 [在 Android 上添加意圖](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL 方案。

惡意應用程式可以通過註冊相同的方案來劫持您的深度連結，從而獲取連結中包含的資料。發送像 `app://products/1` 這樣的連結無害，但發送令牌則是一個安全問題。

當作業系統在打開連結時有兩個或更多應用程式可供選擇時，Android 會顯示一個 [歧義對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog) 並詢問使用者選擇哪個應用程式來打開連結。然而，在 iOS 上，作業系統會為您做出選擇，因此使用者可能完全不知情。Apple 在後續的 iOS 版本（iOS 11）中已採取措施解決此問題，實施了先到先得的原則，儘管此漏洞仍可能以其他方式被利用，您可以在此 [閱讀更多資訊](https://thehackernews.com/2019/07/ios-custom-url-scheme.html)。使用 [通用連結](https://developer.apple.com/ios/universal-links/) 可以在 iOS 中安全地連結到您的應用程式內容。

### OAuth2 與重新導向

OAuth2 認證協議現今非常流行，被譽為最完整且安全的協議。OpenID Connect 協議也基於此。在 OAuth2 中，使用者被要求通過第三方進行認證。成功完成後，該第三方會重新導向回請求的應用程式，並帶有一個驗證碼，該驗證碼可以兌換為 JWT —— [JSON Web Token](https://jwt.io/introduction/)。JWT 是一種在網路上安全傳輸資訊的開放標準。

在網頁上，這個重新導向步驟是安全的，因為網頁上的 URL 保證是唯一的。但對於應用程式來說並非如此，因為如前所述，沒有集中註冊 URL 方案的方法！為了解決這個安全問題，必須以 PKCE 的形式添加額外的檢查。

[PKCE](https://oauth.net/2/pkce/)（發音為「Pixy」）全稱為 Proof of Key Code Exchange，是 OAuth 2 規範的擴充功能。它透過增加一層安全驗證機制，確保認證請求與令牌交換請求來自同一客戶端。PKCE 採用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密雜湊演算法，該演算法能為任意大小的文字或檔案產生獨特「簽章」，其特性如下：

- 無論輸入檔案大小，輸出長度固定
- 相同輸入必定產生相同結果
- 具單向性（無法逆向推導原始輸入）

此時您將持有兩個數值：

- **code_verifier**：由客戶端生成的大型隨機字串
- **code_challenge**：code_verifier 的 SHA 256 雜湊值

During the initial `/authorize` request, the client also sends the `code_challenge` for the `code_verifier` it keeps in memory. After the authorize request has returned correctly, the client also sends the `code_verifier` that was used to generate the `code_challenge`. The IDP will then calculate the `code_challenge`, see if it matches what was set on the very first `/authorize` request, and only send the access token if the values match.

此機制確保僅有觸發初始授權流程的應用程式能成功兌換驗證碼為 JWT。即使惡意應用取得驗證碼，該碼單獨也毫無價值。實作範例可參考[此示範](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

適用於原生 OAuth 的推薦函式庫為 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth)。此 SDK 封裝了原生 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 與 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 庫，能與 OAuth2 供應商溝通並支援 PKCE。

> 注意：react-native-app-auth 的 PKCE 支援取決於您的身份供應商是否實作該功能。

![OAuth2 搭配 PKCE 流程圖](/docs/assets/diagram_pkce.svg)

## 網路安全

API 必須始終採用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。此加密能防止資料在伺服器傳送至客戶端過程中被明文竊取，安全端點可透過 `https://` 開頭（而非 `http://`）識別。

### SSL 憑證綁定

Using https endpoints could still leave your data vulnerable to interception. With https, the client will only trust the server if it can provide a valid certificate that is signed by a trusted Certificate Authority that is pre-installed on the client. An attacker could take advantage of this by installing a malicious root CA certificate to the user’s device, so the client would trust all certificates that are signed by the attacker. Thus, relying on certificates alone could still leave you vulnerable to a [man-in-the-middle attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).

**SSL 憑證綁定**技術可防範此類攻擊。其原理是在開發階段將受信任憑證清單嵌入（綁定）至客戶端，使應用程式僅接受綁定憑證簽署的請求，拒絕自簽憑證。

> 實作 SSL 綁定時需注意憑證有效期。憑證通常每 1-2 年失效，屆時需同步更新伺服器與應用內嵌的憑證。伺服器更新後，仍使用舊憑證的應用將立即無法運作。

## 總結

雖然沒有萬無一失的安全處理方式，但透過有意識的努力與謹慎，仍能大幅降低應用程式遭受安全漏洞的風險。請根據應用程式中儲存資料的敏感度、用戶數量，以及駭客入侵可能造成的損害程度，投入相應的安全防護資源。切記：從一開始就不請求的資訊，要存取它自然就困難得多。