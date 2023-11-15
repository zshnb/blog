---
title: electron react如何使用Node.js API
date: 2023-11-15 15:17:11
tags:
  - Electron
  - React
  - Javascript
---
最近在做一个编辑器项目，因为打算做跨平台应用，于是技术方案使用了electron，同时希望用React做渲染层，于是使用了https://github.com/electron-react-boilerplate/electron-react-boilerplate作为脚手架。在开发过程中遇到一些需要调用Node.js api的场景，比如读写文件，与子进程交互。

一开始尝试直接在React层写`fs.readdir` ，发现会报错需要webpack5的polyfill。于是尝试搜索**electron nodejs webpack polyfill**，发现出来的结果都需要很复杂的配置，于是我尝试在脚手架仓库issue里搜索相关问题，还真找到一个相近的讨论，具体可以点[这里](https://github.com/electron-react-boilerplate/electron-react-boilerplate/issues/3340)查看，总结如下：
<!--more-->

Electron的架构有3个部分

1. Main Thread：主线程，启动整个程序
2. Preload：胶水层，连接主线程和渲染层的通信
3. Web：渲染层，渲染UI用的

因此如果希望在UI层使用Node.js的api，需要

1. 在preload.js，暴露出通信方法

   ```jsx
   const { contextBridge, ipcRenderer } = require('electron')
   contextBridge.exposeInMainWorld('electronAPI', {
     ipcRenderer: {
       sendMessage(channel: Channels, ...args: unknown[]) {
         ipcRenderer.send(channel, ...args);
       },
       on(channel: Channels, func: (...args: unknown[]) => void) {
         const subscription = (_event: IpcRendererEvent, ...args: unknown[]) =>
           func(...args);
         ipcRenderer.on(channel, subscription);
   
         return () => {
           ipcRenderer.removeListener(channel, subscription);
         };
       },
       once(channel: Channels, func: (...args: unknown[]) => void) {
         ipcRenderer.once(channel, (_event, ...args) => func(...args));
       },
     },
   })
   ```

2. 在main.js，监听事件，并作出响应

   ```jsx
   ipcMain.on('example', (event, title) => {
       const webContents = event.sender
       const win = BrowserWindow.fromWebContents(webContents)
       win.setTitle(title)
   		event.reply('ping', 'ping');
   })
   ```

3. 在React里触发事件，并监听事件返回值

   ```jsx
   const { ipcRenderer } = window.electronAPI;
   const setButton = document.getElementById('btn')
   const titleInput = document.getElementById('title')
   setButton.addEventListener('click', () => {
     const title = titleInput.value
     // Call the function registered in the window under 'ElectronAPI', as said you can change this by chagning the preload file 
     // if you want to use something different from 'ElectronAPI'
     ipcRenderer.sendMessage('example')
   	ipcRenderer.on('ping', (args) => {console.log(args)}) 
   })
   ```

这样就能完成从渲染层到主线程的通信了。因此之后所有关于Node.js api的部分都要放在main.js里。