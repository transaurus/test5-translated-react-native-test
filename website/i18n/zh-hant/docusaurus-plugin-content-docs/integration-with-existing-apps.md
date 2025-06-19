---
id: integration-with-existing-apps
title: Integration with Existing Apps
hide_table_of_contents: true
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem';

import IntegrationApple from './\_integration-with-existing-apps-ios.md'; import
IntegrationKotlin from './\_integration-with-existing-apps-kotlin.md';

React Native 非常適合從零開始開發新行動應用程式，同時也能完美整合至現有原生應用中，用於添加單一視圖或用戶流程。只需幾個步驟，您就能加入基於 React Native 的新功能、畫面、視圖等元件。

具體步驟會根據您目標平台的不同而有所差異。

<Tabs groupId="language" queryString defaultValue="kotlin" values={[ {label: 'Android (Java & Kotlin)', value: 'kotlin'}, {label: 'iOS (Objective-C and Swift)', value: 'apple'}, ]}>

<TabItem value="kotlin">

<IntegrationKotlin />

</TabItem>
<TabItem value="apple">

<IntegrationApple />

</TabItem>
</Tabs>