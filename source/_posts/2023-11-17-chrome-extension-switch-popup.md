---
title: chrome插件开发 - 点击插件图标出现弹窗或者监听事件
date: 2023-11-17 12:17:39
tags:
  - ChromeExtension
  - Javascript
---

最近在重构Notion Converter插件的整体UI和功能，其中有个重构场景是当点击插件图标时，根据所在页面不同，行为也有所区别。如果在notion页面点击，弹出插件主题。如果在其他页面点击，弹出popup。

根据[chrome插件文档](https://developer.chrome.com/docs/extensions/reference/action/#popup)说明，如果在manifest.json里配置了`default_popup`，那么popup点击事件就不会发送了。于是思路便是不设置`default_popup` ，同时监听插件点击事件，获取当前tab的url，如果是在notion页面，发送消息通知content_script.js
<!--more-->
```jsx
// background.js
chrome.action.onClicked.addListener(async (tab) => {
  const queryOptions = {active: true, lastFocusedWindow: true};
  const [currentTab] = await chrome.tabs.query(queryOptions);
  const url = currentTab.url
  if (/https:\/\/.*\.notion.so\/.*/.test(url)) {
    chrome.tabs.sendMessage(currentTab.id, {type: 'show-pop-content'});
  }
});
```

如果不在notion页面，设置popup，并弹出popup

```jsx
// background.js
chrome.action.onClicked.addListener(async (tab) => {
  const queryOptions = {active: true, lastFocusedWindow: true};
  const [currentTab] = await chrome.tabs.query(queryOptions);
  const url = currentTab.url
  if (/https:\/\/.*\.notion.so\/.*/.test(url)) {
    chrome.tabs.sendMessage(currentTab.id, {type: 'show-pop-content'})
  } else {
    chrome.action.setPopup({tabId: tab.id, popup: 'popup.html'})
    chrome.action.openPopup()
  }
});
```

刷新插件测试后，发现在非notion页面报错 Type Error: chrome.action.openPopup is not a function。

这个错误把我愣住了，明明在官方文档里有这个函数的定义，怎么跟我说不存在？

![](img1.png)

于是在谷歌上一阵搜索，找到一个[GitHub issue](https://github.com/GoogleChrome/developer.chrome.com/issues/2602)，发现这个问题从chrome99就存在了，而且这还是谷歌自己的问题。根据官方回复，这个API只能用在dev channel，然而定义出现在了文档上，把开发者都搞迷糊了。

![](img2.png)

后来他们修了一版，现在这个API可以用于policy-installed插件了。但很可惜个人插件依然无法使用，由于policy的原因。

![](img3.png)

不过好在讨论里有大佬给出了另一种解决方案，代码如下

```jsx
function setPopupForTab(tab) {
  const url = tab.url
  if (/https:\/\/.*\.notion.so\/.*/.test(url)) {
		// 取消popup的弹出页面
    chrome.action.setPopup({tabId: tab.id, popup: ''});
  } else {
    chrome.action.setPopup({tabId: tab.id, popup: 'popup.html'})
  }
}

chrome.action.onClicked.addListener(async (tab) => {
  const queryOptions = {active: true, lastFocusedWindow: true}
  const [currentTab] = await chrome.tabs.query(queryOptions)
  const url = currentTab.url
  if (/https:\/\/.*\.notion.so\/.*/.test(url)) {
    chrome.tabs.sendMessage(currentTab.id, {type: 'show-pop-content'})
  }
})

// 切换tab事件，根据tab变更popup
chrome.tabs.onActivated.addListener((activeInfo) => {
  chrome.tabs.get(activeInfo.tabId, (tab) => {
    setPopupForTab(tab)
  })
})
```

最后完美解决