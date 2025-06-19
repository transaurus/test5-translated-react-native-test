---
title: Making React Native apps accessible
author: Georgiy Kassabli
authorTitle: Software Engineer at Facebook
authorURL: 'https://www.facebook.com/georgiy.kassabli'
authorImageURL: 'https://scontent-sea1-1.xx.fbcdn.net/v/t1.0-1/c0.0.160.160/p160x160/1978838_795592927136196_1205041943_n.jpg?_nc_log=1&oh=d7a500fdece1250955a4d27b0a80fee2&oe=59E8165A'
hero: '/blog/assets/blue-hero.png'
tags: [engineering]
---

隨著 React 在網頁端和 React Native 在行動端的推出，我們為開發者提供了一個新的前端框架來建構產品。打造一個穩健產品的關鍵面向之一，是確保所有人都能使用它，包括視力受損或其他障礙的使用者。React 和 React Native 的無障礙功能 API（Accessibility API）能讓您將任何以 React 驅動的體驗，變得可供使用輔助技術的人士操作，例如為盲人和視障者設計的螢幕閱讀器。

For this post, we're going to focus on React Native apps. We've designed the React Accessibility API to look and feel similar to the Android and iOS APIs. If you've developed accessible applications for Android, iOS, or the web before, you should feel comfortable with the framework and nomenclature of the React AX API. For instance, you can make a UI element _accessible_ (therefore exposed to assistive technology) and use _accessibilityLabel_ to provide a string description for the element:

```
<View accessible={true} accessibilityLabel=”This is simple view”>
```

讓我們透過 Facebook 自家以 React 驅動的產品之一：**廣告管理員應用程式（Ads Manager app）**，來逐步解析 React AX API 稍微複雜一些的應用方式。

<footer>
  <a
    href="https://code.facebook.com/posts/435862739941212/making-react-native-apps-accessible/"
    className="btn">Read more</a>
</footer>

> 此為節錄內容。完整文章請見 [Facebook Code](https://code.facebook.com/posts/435862739941212/making-react-native-apps-accessible/)。