---
title: Chrome插件开发无法收到消息排查
date: 2025-03-07 11:12:49
tags:
  - Chrome extension
---

最近开发的插件里有个功能是，从content_script发送消息到background，等待background处理完content_script执行剩下逻辑，类似以下逻辑
<!--more-->
```
// content_script.js
async function handle() {
  const response = await chrome.runtime.sendMessage({
    type: 'download',
    messageId
  })
  if (response.ok) {
    console.log('download voice success');
  } else {
    console.log('download voice error');
  }
}

// background.js
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  ...
  await doSync(sendResponse) // 异步函数
}

async doSync(sendResponse) {
  sendResponse({ok: true})
}

```
期望行为是当background执行完异步函数，返回ok，content_script会打印success，实际结果是content_script收到的response永远是undefined，且background的异步函数确实异步在执行。

搜了一下谷歌官方文档，里面提到

> When you send a message, the event listener that handles the message is passed an optional third argument, `sendResponse`. This is a function that takes a JSON-serializable object that is used as the return value to the function that sent the message. By default, the `sendResponse` callback must be called synchronously. If you want to do asynchronous work to get the value passed to `sendResponse`, you **must** return a literal `true` (not just a truthy value) from the event listener. Doing so will keep the message channel open to the other end until `sendResponse` is called.
>

意思是当消息监听里含有异步操作时，必须要在消息监听函数最后返回true，所以上面的函数要修改成

```
// background.js
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  ...
  doSync(sendResponse) // 异步函数
  return true
}

async doSync(sendResponse) {
  sendResponse({ok: true})
}

```
这样在content_script里就能如期收到返回值了。
