---
id: sourcemaps
title: Source Maps
---

Source maps 可讓您將轉換後的檔案映射回原始來源檔案，其主要目的是輔助除錯並協助調查正式版建置中的問題。

若無 source maps，當正式版建置發生錯誤時，您將看到類似以下的堆疊追蹤：

```text
TypeError: Cannot read property 'data' of undefined
  at anonymous(app:///index.android.bundle:1:4277021)
  at call(native)
  at p(app:///index.android.bundle:1:227785)
```

若已生成 source maps，堆疊追蹤將包含原始來源檔案的路徑、檔名與行號：

```text
TypeError: Cannot read property 'data' of undefined
  at anonymous(src/modules/notifications/Permission.js:15:requestNotificationPermission)
  at call(native)
  at p(node_modules/regenerator-runtime/runtime.js:64:Generator)
```

這讓您能透過可解讀的堆疊追蹤來分類正式版問題。

請遵循以下指示開始使用 source maps。

## 在 Android 上啟用 source maps

### Hermes

:::info
預設情況下，source maps 會在 Release 模式中自動建置，除非設定了 `hermesFlagsRelease`。此情況下需手動啟用 source maps。
:::

請確認您應用程式的 `android/app/build.gradle` 檔案中已設定以下內容：

```groovy
project.ext.react = [
    enableHermes: true,
    hermesFlagsRelease: ["-O", "-output-source-map"], // plus whichever flag was required to set this away from default
]
```

若設定正確，您應能在 Metro 建置輸出中看到 source map 的輸出位置。

```text
Writing bundle output to:, android/app/build/generated/assets/react/release/index.android.bundle
Writing sourcemap output to:, android/app/build/intermediates/sourcemaps/react/release/index.android.bundle.packager.map
```

開發版建置不會產生 bundle 因此已具備符號表，但若開發版建置有 bundle 需求，可參照前述方式使用 `hermesFlagsDebug` 來啟用 source maps。

## 在 iOS 上啟用 source maps

source maps 預設為停用狀態。需定義 `SOURCEMAP_FILE` 環境變數來啟用。

請於 Xcode 中前往 Build Phase 的 "Bundle React Native code and images" 階段。

在檔案頂部其他 export 指令附近，新增 `SOURCEMAP_FILE` 項目並指定偏好路徑與名稱，如下所示：

```
export SOURCEMAP_FILE="$(pwd)/../main.jsbundle.map";

export NODE_BINARY=node
../node_modules/react-native/scripts/react-native-xcode.sh
```

若設定正確，您應能在 Metro 建置輸出中看到 source map 的輸出位置。

```text
Writing bundle output to:, Build/Intermediates.noindex/ArchiveIntermediates/application/BuildProductsPath/Release-iphoneos/main.jsbundle
Writing sourcemap output to:, Build/Intermediates.noindex/ArchiveIntermediates/application/BuildProductsPath/Release-iphoneos/main.jsbundle.map
```

## 手動符號化

:::info
您可於 [符號化](symbolication.md) 頁面閱讀關於手動符號化堆疊追蹤的說明。
:::