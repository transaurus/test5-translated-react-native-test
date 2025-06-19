---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在開發應用程式時，安全性經常被忽視。雖然確實無法打造完全無法攻破的軟體（畢竟銀行金庫仍可能被入侵），但遭受惡意攻擊或暴露安全漏洞的機率，與你投入的防護努力成反比。就像普通掛鎖雖能被撬開，但比起簡易掛鉤仍難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及強化應用安全的工具。這不是一份檢查清單，而是各種防護選項的目錄，每項都能進一步保護你的應用與使用者。

## 儲存敏感資訊

切勿將敏感 API 金鑰儲存在應用程式碼中。任何包含在程式碼中的內容，都可能被檢查應用套件的人以純文字形式讀取。雖然 [react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv) 和 [react-native-config](https://github.com/luggit/react-native-config/) 等工具適合管理環境變數（如 API 端點），但它們不同於伺服器端的環境變數，後者常包含機密金鑰。

若必須在應用中使用 API 金鑰或密鑰存取資源，最安全的做法是在應用與資源間建立中介層。例如使用無伺服器函數（如 AWS Lambda 或 Google Cloud Functions）轉發請求並附加所需金鑰。伺服器端程式碼中的密鑰不會像應用程式碼中的密鑰那樣被外部直接讀取。

**根據敏感程度選擇適當的持久化儲存方式**。隨著應用使用，你常需要將資料儲存在裝置上，無論是為了支援離線使用、減少網路請求，或是保存使用者存取權杖以避免每次重新驗證。

> **持久化 vs 非持久化** — 持久化資料會寫入裝置磁碟，讓應用在多次啟動間能讀取資料，無需重新網路請求或要求使用者輸入。但這也使資料更容易被攻擊者存取。非持久化資料永不寫入磁碟，自然無從竊取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage) 是 React Native 的社群維護模組，提供非同步、未加密的鍵值儲存。Async Storage 不跨應用共享：每個應用都有獨立的沙箱環境，無法存取其他應用的資料。

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

React Native 原生未內建敏感資料儲存方案，但 Android 和 iOS 平台已有現成解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services) 可安全儲存使用者的敏感小資料，是存放憑證、權杖、密碼等不適合 Async Storage 之敏感資訊的理想位置。

#### Android - 加密 Shared Preferences

[Shared Preferences](https://developer.android.com/reference/android/content/SharedPreferences) 是 Android 的持久化鍵值儲存方案，**預設未加密**。但 [加密版 Shared Preferences](https://developer.android.com/topic/security/data) 會自動加密鍵與值。

#### Android - 金鑰庫

The [Android Keystore](https://developer.android.com/training/articles/keystore) system lets you store cryptographic keys in a container to make it more difficult to extract from the device.

若要使用 iOS Keychain 服務或 Android Secure Shared Preferences，您可以自行編寫橋接程式碼，或使用封裝這些功能的函式庫（需自行承擔風險）並提供統一的 API。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-keychain](https://github.com/oblador/react-native-keychain)

> **注意無意間儲存或暴露敏感資訊的情況。** 這可能意外發生，例如將敏感表單資料儲存在 redux state 中，並將整個狀態樹持久化到 Async Storage。或將使用者令牌和個人資訊發送到應用程式監控服務（如 Sentry 或 Crashlytics）。

## 認證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

Mobile apps have a unique vulnerability that is non-existent in the web: **deep linking**. Deep linking is a way of sending data directly to a native application from an outside source. A deep link looks like `app://` where `app` is your app scheme and anything following the // could be used internally to handle the request.

例如，如果您正在構建一個電子商務應用程式，您可以使用 `app://products/1` 來深度連結到您的應用程式，並打開 id 為 1 的產品詳細頁面。您可以將這些連結想像成網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您絕不應在其中發送任何敏感資訊。

深度連結不安全的原因是沒有集中式的 URL scheme 註冊方法。作為應用程式開發人員，您可以通過 [在 Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)（iOS）或 [在 Android 上添加 intent](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL scheme。

惡意應用程式可以通過註冊相同的 scheme 來劫持您的深度連結，從而獲取連結中包含的資料。發送像 `app://products/1` 這樣的連結無害，但發送令牌則是一個安全問題。

當作業系統在打開連結時有兩個或更多應用程式可選時，Android 會向用戶顯示 [歧義對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog) 並詢問用戶選擇使用哪個應用程式來打開連結。然而在 iOS 上，作業系統會替用戶做出選擇，因此用戶可能完全不知情。Apple 在後續的 iOS 版本（iOS 11）中已採取措施解決此問題，實施了先到先得的原則，但此漏洞仍可能以其他方式被利用，您可以 [在此](https://thehackernews.com/2019/07/ios-custom-url-scheme.html) 閱讀更多相關資訊。使用 [通用連結](https://developer.apple.com/ios/universal-links/) 可以在 iOS 中安全地連結到應用程式內的內容。

### OAuth2 與重新導向

OAuth2 認證協議現今非常流行，被譽為最完整且安全的協議。OpenID Connect 協議也基於此。在 OAuth2 中，用戶需通過第三方進行認證。成功完成後，該第三方會重新導向回請求的應用程式，並附帶一個可兌換為 JWT（[JSON Web Token](https://jwt.io/introduction/)）的驗證碼。JWT 是一種在網路上安全傳輸資訊的開放標準。

在網頁上，此重新導向步驟是安全的，因為網頁上的 URL 保證是唯一的。但這不適用於應用程式，因為如前所述，沒有集中式的 URL scheme 註冊方法！為了解決此安全問題，必須添加 PKCE 形式的額外檢查。

[PKCE](https://oauth.net/2/pkce/)（發音為「Pixy」）全稱為「Proof of Key Code Exchange」，是 OAuth 2 規範的擴充功能。它透過增加一層安全驗證機制，確保認證請求與令牌交換請求來自同一客戶端。PKCE 採用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密雜湊演算法，該演算法能為任意大小的文字或檔案產生獨特「簽章」，其特性如下：

- 無論輸入檔案大小，輸出長度固定
- 相同輸入必定產生相同結果
- 單向不可逆（無法從簽章反推原始輸入）

此時您將持有兩個數值：

- **code_verifier**：由客戶端生成的隨機長字串
- **code_challenge**：code_verifier 的 SHA 256 雜湊值

During the initial `/authorize` request, the client also sends the `code_challenge` for the `code_verifier` it keeps in memory. After the authorize request has returned correctly, the client also sends the `code_verifier` that was used to generate the `code_challenge`. The IDP will then calculate the `code_challenge`, see if it matches what was set on the very first `/authorize` request, and only send the access token if the values match.

此機制確保僅有觸發初始授權流程的應用程式能成功兌換驗證碼為 JWT。即使惡意應用取得驗證碼，單獨持有亦無效用。實作範例可參考[此示範](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

適用於原生 OAuth 的推薦函式庫為 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth)。該 SDK 封裝了原生 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 與 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 函式庫，可支援 PKCE 機制。

> 注意：react-native-app-auth 的 PKCE 支援需視您的身份提供者（Identity Provider）是否實作此功能而定。

![OAuth2 搭配 PKCE 流程圖](/docs/assets/diagram_pkce.svg)

## 網路安全

API 必須全程使用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。SSL 能防止資料在伺服器與客戶端傳輸過程中被以明文竊取，安全端點可透過 `https://` 開頭（而非 `http://`）識別。

### SSL 憑證綁定

僅使用 https 仍可能使資料遭攔截。在 https 架構下，客戶端僅信任由預裝可信憑證機構（CA）簽署的憑證。攻擊者可能透過在用戶裝置安裝惡意根憑證，使客戶端誤認攻擊者簽發的憑證。此類[中間人攻擊](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)風險依然存在。

**SSL 憑證綁定**技術可防範此類攻擊。其原理是在開發階段將受信任憑證清單嵌入（綁定）至客戶端，使應用程式僅接受特定憑證簽署的請求，拒絕自簽憑證。

> 實作 SSL 綁定時需注意憑證有效期。憑證通常每 1-2 年失效，屆時需同步更新伺服器與應用內嵌憑證。若伺服器更新憑證後，用戶端應用仍使用舊憑證，將導致連線失敗。

## 總結

雖然沒有萬無一失的安全處理方式，但透過有意識的努力和謹慎態度，仍能大幅降低應用程式遭受安全漏洞的可能性。請根據應用程式中儲存資料的敏感性、用戶數量，以及駭客入侵可能造成的損害程度，投入相應的安全防護資源。切記：那些從未被請求過的資訊，其被竊取的難度自然會大幅提高。