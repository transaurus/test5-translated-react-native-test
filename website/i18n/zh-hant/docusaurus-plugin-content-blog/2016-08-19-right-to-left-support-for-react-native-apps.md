---
title: Right-to-Left Layout Support For React Native Apps
author: Mengjue (Mandy) Wang
authorTitle: Software Engineer Intern at Facebook
authorURL: 'https://github.com/MengjueW'
authorImageURL: 'https://avatars0.githubusercontent.com/u/13987140?v=3&s=128'
tags: [engineering]
---

在將應用程式發布至應用商店後，國際化是擴大受眾範圍的下一步。全球有超過20個國家和無數人口使用從右至左（RTL）的語言。因此，讓您的應用程式支援RTL對這些用戶來說是必要的。

我們很高興地宣布，React Native已經改進以支援RTL佈局。這項功能現在已經可以在[react-native](https://github.com/facebook/react-native)的主分支中使用，並且將在下一個RC版本：[`v0.33.0-rc`](https://github.com/facebook/react-native/releases)中提供。

這涉及更改[css-layout](https://github.com/facebook/css-layout)——RN使用的核心佈局引擎，以及RN核心實現，以及特定的開源JS組件以支援RTL。

為了在生產環境中測試RTL支援，最新版本的**Facebook Ads Manager**應用（第一個跨平台100% RN應用）現在已經在阿拉伯語和希伯來語中提供RTL佈局，適用於[iOS](https://itunes.apple.com/app/id964397083)和[Android](https://play.google.com/store/apps/details?id=com.facebook.adsmanager)。以下是這些RTL語言中的應用外觀：

<>
<img src="/blog/assets/rtl-ama-ios-arabic.png" width={280} style={{ margin: 10 }} />
<img src="/blog/assets/rtl-ama-android-hebrew.png" width={280} style={{ margin: 10 }} />
</>

## RN中RTL支援的概述變更

[css-layout](https://github.com/facebook/css-layout)已經有`start`和`end`的佈局概念。在從左至右（LTR）佈局中，`start`意味著`left`，而`end`意味著`right`。但在RTL中，`start`意味著`right`，而`end`意味著`left`。這意味著我們可以讓RN依賴`start`和`end`的計算來得出正確的佈局，這包括`position`、`padding`和`margin`。

此外，[css-layout](https://github.com/facebook/css-layout)已經讓每個組件的方向繼承自其父組件。這意味著，我們只需將根組件的方向設置為RTL，整個應用就會翻轉。

下圖從高層次描述了這些變更：

![](/blog/assets/rtl-rn-core-updates.png)

這些包括：

- [css-layout對絕對定位的RTL支援](https://github.com/facebook/css-layout/commit/46c842c71a1232c3c78c4215275d104a389a9a0f)
- 在RN核心實現中將`left`和`right`映射到`start`和`end`以用於陰影節點
- 並公開一個[橋接實用模塊](https://github.com/facebook/react-native/blob/f0fb228ec76ed49e6ed6d786d888e8113b8959a2/Libraries/Utilities/I18nManager.js)來幫助控制RTL佈局

通過這次更新，當您允許應用程式使用RTL佈局時：

- 每個組件的佈局將水平翻轉
- 如果您使用RTL-ready的開源組件，某些手勢和動畫將自動具有RTL佈局
- 可能需要最少的額外努力來讓您的應用完全RTL-ready

## 讓應用程式RTL-ready

1. To support RTL, you should first add the RTL language bundles to your app.

   - See the general guides from [iOS](https://developer.apple.com/library/ios/documentation/MacOSX/Conceptual/BPInternational/LocalizingYourApp/LocalizingYourApp.html#//apple_ref/doc/uid/10000171i-CH5-SW1) and [Android](https://developer.android.com/training/basics/supporting-devices/languages.html).

2. Allow RTL layout for your app by calling the `allowRTL()` function at the beginning of native code. We provided this utility to only apply to an RTL layout when your app is ready. Here is an example:

   iOS:

   ```objc
   // in AppDelegate.m
     [[RCTI18nUtil sharedInstance] allowRTL:YES];
   ```

   Android:

   ```java
   // in MainActivity.java
     I18nUtil sharedI18nUtilInstance = I18nUtil.getInstance();
     sharedI18nUtilInstance.allowRTL(context, true);
   ```

3. For Android, you need add `android:supportsRtl="true"` to the [`<application>`](https://developer.android.com/guide/topics/manifest/application-element.html) element in `AndroidManifest.xml` file.

現在，當你重新編譯應用程式並將裝置語言切換為 RTL 語言（例如阿拉伯語或希伯來語），應用程式佈局應會自動切換為 RTL。

## 撰寫 RTL 相容的元件

一般而言，多數元件已預設支援 RTL，例如：

- 由左至右佈局

<img src="/blog/assets/rtl-demo-listitem-ltr.png" width="300" />

- 由右至左佈局

<img src="/blog/assets/rtl-demo-listitem-rtl.png" width="300" />

但仍有幾種情況需特別注意，此時需使用 [`I18nManager`](https://github.com/facebook/react-native/blob/f0fb228ec76ed49e6ed6d786d888e8113b8959a2/Libraries/Utilities/I18nManager.js)。[`I18nManager`](https://github.com/facebook/react-native/blob/f0fb228ec76ed49e6ed6d786d888e8113b8959a2/Libraries/Utilities/I18nManager.js) 中的常數 `isRTL` 可判斷應用程式是否處於 RTL 佈局，以便你根據佈局方向進行必要調整。

#### 具方向性意義的圖示

若元件包含圖示或圖片，由於 RN 不會翻轉原始圖像，這些圖示在 LTR 和 RTL 佈局中會以相同方向顯示。因此，你需根據佈局方向手動翻轉它們。

- 由左至右佈局

<img src="/blog/assets/rtl-demo-icon-ltr.png" width="300" />

- 由右至左佈局

<img src="/blog/assets/rtl-demo-icon-rtl.png" width="300" />

以下是兩種根據方向翻轉圖示的方法：

- 對圖片元件加入 `transform` 樣式：

  ```jsx
  <Image
    source={...}
    style={{transform: [{scaleX: I18nManager.isRTL ? -1 : 1}]}}
  />
  ```

- 或根據方向切換圖片來源：

  ```jsx
  let imageSource = require('./back.png');
  if (I18nManager.isRTL) {
    imageSource = require('./forward.png');
  }
  return <Image source={imageSource} />;
  ```

#### 手勢與動畫

在 Android 和 iOS 開發中，切換至 RTL 佈局時，手勢與動畫方向會與 LTR 佈局相反。目前 RN 核心代碼層級尚未直接支援手勢與動畫的 RTL 調整，而是由元件層級實作。好消息是，部分元件（如 [`SwipeableRow`](https://github.com/facebook/react-native/blob/38a6eec0db85a5204e85a9a92b4dee2db9641671/Libraries/Experimental/SwipeableRow/SwipeableRow.js) 和 [`NavigationExperimental`](https://github.com/facebook/react-native/tree/master/Libraries/NavigationExperimental)）已支援 RTL。但其他包含手勢操作的元件仍需手動實作 RTL 支援。

一個能清楚展示手勢RTL支援的範例是[`SwipeableRow`](https://github.com/facebook/react-native/blob/38a6eec0db85a5204e85a9a92b4dee2db9641671/Libraries/Experimental/SwipeableRow/SwipeableRow.js)。

<p align="center">
  <img src="/blog/assets/rtl-demo-swipe-ltr.png" width={280} style={{margin: 10}} />
  <img src="/blog/assets/rtl-demo-swipe-rtl.png" width={280} style={{margin: 10}} />
</p>

##### 手勢範例

```js
// SwipeableRow.js
_isSwipingExcessivelyRightFromClosedPosition(gestureState: Object): boolean {
  // ...
  const gestureStateDx = IS_RTL ? -gestureState.dx : gestureState.dx;
  return (
    this._isSwipingRightFromClosed(gestureState) &&
    gestureStateDx > RIGHT_SWIPE_THRESHOLD
  );
},
```

##### 動畫範例

```js
// SwipeableRow.js
_animateBounceBack(duration: number): void {
  // ...
  const swipeBounceBackDistance = IS_RTL ?
    -RIGHT_SWIPE_BOUNCE_BACK_DISTANCE :
    RIGHT_SWIPE_BOUNCE_BACK_DISTANCE;
  this._animateTo(
    -swipeBounceBackDistance,
    duration,
    this._animateToClosedPositionDuringBounce,
  );
},
```

## 維護您的RTL就緒應用程式

即使在初始發布RTL相容的應用程式後，您很可能仍需迭代新功能。為了提升開發效率，[`I18nManager`](https://github.com/facebook/react-native/blob/f0fb228ec76ed49e6ed6d786d888e8113b8959a2/Libraries/Utilities/I18nManager.js)提供了`forceRTL()`函數，讓您無需變更測試裝置的語言即可快速測試RTL佈局。您可能想在應用程式中提供一個簡單的切換開關。以下是RNTester中RTL範例的程式碼片段：

<p align="center">
  <img src="/blog/assets/rtl-demo-forcertl.png" width="300" />
</p>

```js
<RNTesterBlock title={'Quickly Test RTL Layout'}>
  <View style={styles.flexDirectionRow}>
    <Text style={styles.switchRowTextView}>forceRTL</Text>
    <View style={styles.switchRowSwitchView}>
      <Switch
        onValueChange={this._onDirectionChange}
        style={styles.rightAlignStyle}
        value={this.state.isRTL}
      />
    </View>
  </View>
</RNTesterBlock>;

_onDirectionChange = () => {
  I18nManager.forceRTL(!this.state.isRTL);
  this.setState({isRTL: !this.state.isRTL});
  Alert.alert(
    'Reload this page',
    'Please reload this page to change the UI direction! ' +
      'All examples in this app will be affected. ' +
      'Check them out to see what they look like in RTL layout.',
  );
};
```

開發新功能時，您可以輕鬆切換此按鈕並重新載入應用程式來查看RTL佈局。這樣做的好處是您無需變更語言設定來測試，但某些文字對齊方式不會改變，如下一節所述。因此，在發布前使用RTL語言測試您的應用程式始終是個好主意。

## 當前限制與未來計劃

RTL支援應涵蓋您應用程式中大多數的使用者體驗，但目前仍存在一些限制：

- 文字對齊行為在Android和iOS上有所不同
  - 在iOS上，預設文字對齊取決於當前使用的語言包，它們會固定對齊某一側。在Android上，預設文字對齊取決於文字內容的語言，例如英文會靠左對齊，阿拉伯文會靠右對齊。
  - 理論上這應該在跨平台時保持一致，但某些使用者可能偏好某種對齊方式。可能需要更多使用者體驗研究來找出文字對齊的最佳實踐。

* 沒有「絕對」的左/右

  如前所述，我們將JS端的`left`/`right`樣式映射為`start`/`end`，所有程式碼中的`left`在RTL佈局中會變成畫面上的「右」，而`right`會變成「左」。這很方便，因為您無需大幅修改產品程式碼，但這意味著無法在程式碼中指定「絕對左」或「絕對右」。未來可能需要允許元件不受語言影響控制其方向。

* 讓手勢和動畫的RTL支援對開發者更友好

  目前，要使手勢和動畫相容RTL仍需要一些程式設計工作。未來，理想情況是找到一種方法讓手勢和動畫的RTL支援對開發者更友好。

## 立即試用！

Check out the [`RTLExample`](https://github.com/facebook/react-native/blob/master/packages/rn-tester/js/examples/RTL/RTLExample.js) in the `RNTester` to understand more about RTL support, and let us know how it works for you!

最後，感謝您的閱讀！我們希望React Native的RTL支援能幫助您為國際使用者擴展應用程式！