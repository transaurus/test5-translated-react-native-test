---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在開發應用程式時，安全性往往容易被忽視。雖然我們無法打造完全無法攻破的軟體（畢竟就連銀行金庫也可能被闖入），但遭受惡意攻擊或暴露於安全漏洞的風險，與你投入保護應用程式的努力成反比。就像普通掛鎖雖能被撬開，但突破它仍遠比打開一個櫥櫃掛鉤困難得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及強化應用程式安全的工具。這並非一份簡單的檢查清單，而是一系列選項的目錄，每項都能為你的應用程式和用戶提供更深層的保護。

## 儲存敏感資訊

切勿將敏感API金鑰儲存在應用程式代碼中。任何包含在代碼中的內容都可能被檢查應用套件的人以純文字形式存取。雖然像[react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv)和[react-native-config](https://github.com/luggit/react-native-config/)這類工具適合用於添加環境變數（如API端點），但它們與伺服器端的環境變數不同，後者常包含密鑰和API金鑰。

若必須透過API金鑰或密鑰存取資源，最安全的做法是在應用程式與資源間建立協調層。例如使用無伺服器函數（如AWS Lambda或Google Cloud Functions）來轉發請求並附加所需金鑰。伺服器端代碼中的密鑰無法像應用代碼中的密鑰那樣被API使用者直接存取。

**針對需持久化的用戶數據，應根據敏感程度選擇合適的儲存方式。** 應用程式使用過程中，常需要將數據儲存在裝置上，無論是為了支援離線使用、減少網路請求，或是保存用戶的存取權杖以避免每次重新驗證。

> **持久化 vs 非持久化** — 持久化數據會寫入裝置磁碟，讓應用程式在多次啟動間能讀取數據，無需重新取得。但這也使得數據更容易被攻擊者存取。非持久化數據則永不寫入磁碟，自然無數據可竊取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage)是React Native社群維護的模組，提供非同步、未加密的鍵值儲存。Async Storage不跨應用共享：每個應用都有獨立的沙箱環境，無法存取其他應用的數據。

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

React Native並未內建儲存敏感數據的方案，但Android和iOS平台已有現成解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services)可安全儲存用戶的小型敏感數據，是保存憑證、權杖、密碼等不適合存於Async Storage之資料的理想場所。

#### Android - 加密共享偏好設定

[共享偏好設定](https://developer.android.com/reference/android/content/SharedPreferences)是Android的持久化鍵值儲存方案，**預設不加密數據**。但[加密版共享偏好設定](https://developer.android.com/topic/security/data)會自動加密鍵與值。

#### Android - 金鑰庫

The [Android Keystore](https://developer.android.com/training/articles/keystore) system lets you store cryptographic keys in a container to make it more difficult to extract from the device.

若要使用 iOS Keychain 服務或 Android 安全共享偏好設定，您可以自行編寫橋接程式碼，或選擇使用封裝這些功能的函式庫（需自行承擔風險），這些函式庫會提供統一的 API。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)  
- [react-native-keychain](https://github.com/oblador/react-native-keychain)

> **請注意無意間儲存或暴露敏感資訊的情況。** 這可能意外發生，例如將敏感表單資料儲存在 redux 狀態中，並將整個狀態樹持久化到 Async Storage。或將使用者令牌和個人資訊發送到應用程式監控服務（如 Sentry 或 Crashlytics）。

## 驗證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

Mobile apps have a unique vulnerability that is non-existent in the web: **deep linking**. Deep linking is a way of sending data directly to a native application from an outside source. A deep link looks like `app://` where `app` is your app scheme and anything following the // could be used internally to handle the request.

例如，如果您正在開發一個電子商務應用程式，可以使用 `app://products/1` 來深度連結到您的應用程式，並開啟 ID 為 1 的產品詳細頁面。您可以將這些連結想像成網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您絕不應在其中傳送任何敏感資訊。

深度連結不安全的原因是沒有統一的 URL 方案註冊方法。作為應用程式開發者，您可以透過 [在 Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)（iOS）或 [在 Android 上添加意圖](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL 方案。

惡意應用程式可以透過註冊相同的方案來劫持您的深度連結，從而獲取連結中包含的資料。傳送像 `app://products/1` 這樣的連結無害，但傳送令牌則會引發安全問題。

當作業系統在開啟連結時有兩個或更多應用程式可選時，Android 會向使用者顯示 [歧義對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog)，詢問他們選擇哪個應用程式來開啟連結。然而在 iOS 上，作業系統會替使用者做出選擇，因此使用者可能完全不知情。Apple 在後續的 iOS 版本（iOS 11）中採取了措施來解決此問題，實施了「先到先得」原則，但此漏洞仍可能以其他方式被利用，更多資訊可參考[此處](https://thehackernews.com/2019/07/ios-custom-url-scheme.html)。使用 [通用連結](https://developer.apple.com/ios/universal-links/) 可以在 iOS 中安全地連結到應用程式內的內容。

### OAuth2 與重新導向

OAuth2 驗證協議現今非常流行，被譽為最完整且安全的協議。OpenID Connect 協議也基於此。在 OAuth2 中，使用者需透過第三方進行驗證。成功完成後，第三方會重新導向回請求的應用程式，並附上可兌換為 JWT（[JSON Web Token](https://jwt.io/introduction/)）的驗證碼。JWT 是一種在網路上安全傳輸資訊的開放標準。

在網頁上，此重新導向步驟是安全的，因為網頁上的 URL 保證是唯一的。但這不適用於應用程式，因為如前所述，沒有統一的 URL 方案註冊方法！為了解決此安全問題，必須添加 PKCE 作為額外的檢查機制。

[PKCE](https://oauth.net/2/pkce/)（發音為「Pixy」）代表「Proof of Key Code Exchange」，是 OAuth 2 規範的擴展。它透過增加一層安全性驗證，確保認證與令牌交換請求來自同一個客戶端。PKCE 使用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密雜湊演算法。SHA 256 會為任何大小的文字或檔案產生獨特的「簽章」，但具有以下特性：

- 無論輸入檔案大小，輸出長度固定
- 相同輸入必定產生相同結果
- 單向性（無法逆向推導原始輸入）

現在你會得到兩個值：

- **code_verifier** - 由客戶端生成的大型隨機字串
- **code_challenge** - code_verifier 的 SHA 256 雜湊值

During the initial `/authorize` request, the client also sends the `code_challenge` for the `code_verifier` it keeps in memory. After the authorize request has returned correctly, the client also sends the `code_verifier` that was used to generate the `code_challenge`. The IDP will then calculate the `code_challenge`, see if it matches what was set on the very first `/authorize` request, and only send the access token if the values match.

這確保了只有觸發初始授權流程的應用程式才能成功將驗證碼兌換為 JWT。因此，即使惡意應用程式獲取了驗證碼，單獨使用也毫無意義。若要查看實際運作方式，請參考[此範例](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

適用於原生 OAuth 的推薦函式庫是 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth)。該 SDK 封裝了原生 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 和 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 函式庫，可支援 PKCE 協定。

> 注意：react-native-app-auth 僅在您的身份提供者支援 PKCE 時才能啟用此功能。

![OAuth2 搭配 PKCE 流程圖](/docs/assets/diagram_pkce.svg)

## 網路安全

您的 API 應始終使用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。SSL 加密能防止資料在伺服器傳送至客戶端過程中被以明文竊取。安全端點的識別特徵是網址以 `https://` 開頭而非 `http://`。

### SSL 憑證綁定

即使使用 https 端點，資料仍可能遭攔截。在 https 架構下，客戶端僅信任由預裝可信憑證機構（CA）簽署的有效憑證。攻擊者可透過在用戶裝置安裝惡意根 CA 憑證，使客戶端信任攻擊者簽發的所有憑證。因此，單純依賴憑證驗證仍可能遭受[中間人攻擊](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)。

**SSL 憑證綁定**是客戶端防範此類攻擊的技術。其原理是在開發階段將受信任的憑證清單嵌入（或「綁定」）至客戶端，使得僅接受由這些憑證簽署的請求，而拒絕所有自簽憑證。

> 實施 SSL 綁定時需注意憑證有效期。憑證通常每 1-2 年會過期，屆時需同步更新伺服器與應用程式內的憑證。一旦伺服器憑證更新，內嵌舊憑證的應用程式將立即失效。

## 總結

雖然沒有萬無一失的安全處理方式，但透過有意識的努力和謹慎態度，仍能大幅降低應用程式遭受安全漏洞的風險。請根據應用程式儲存資料的敏感度、用戶數量，以及駭客入侵帳戶可能造成的損害程度，投入相應比例的安全防護資源。切記：那些從未被請求過的資訊，要取得它們的難度會顯著提高。