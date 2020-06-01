---
title: Electron 学习笔记 - TypeScript 的使用
author: 子丶言
date: 2020-06-01 17:21:00
tags: ['Electron', 'Electron 学习笔记', '跨平台', '桌面应用']
---

## 引言

Electron（原名为Atom Shell）是 GitHub 开发的一个开源框架。它允许使用 Node.js（作为后端）和 Chromium（作为前端）完成桌面GUI应用程序的开发。Electron 现已被多个开源 Web 应用程序用于前端与后端的开发，著名项目包括 GitHub 的 Atom 和微软的 Visual Studio Code。

Electron 使用 JavaScript 作为基础编程语言，你实际上是在 Chromium 中编写前端代码，在 Node.js 中编写后端代码。随着时代的发展，TypeScript 越来越受到前端开发人员的欢迎，源于 TypeScript 是 JavaScript 的严格超集，不仅包含 JavaScript 的语法，而且还提供了静态类型检查。

## 在项目中使用 TypeScript

### 针对新项目

目前 Electron 并不支持直接使用 TypeScript 进行开发，但官方提供了 [electron-quick-start-typescript](https://github.com/electron/electron-quick-start-typescript)，我们将在此基础上进行开发。

在运行以下代码之前，你需要确保已经安装了 [Git](https://git-scm.com/) 和 [Node.js](https://nodejs.org/zh-cn/)。

```shell
# 拉取项目
git clone https://github.com/electron/electron-quick-start-typescript
# 进行项目目录
cd electron-quick-start-typescript
# 安装依赖
npm install
# 运行项目
npm start
```

这样你就可以在项目中愉快的使用 TypeScript 进行项目开发了。

### 针对老项目

当你在某一天需要把一个现有 TypeScript 项目转成 Electron 应用时，你可能会遇到一些麻烦。你的项目可能使用了 webpack 进行代码编译，但 webpack 并没有对 Electron 做优化，因此编译后的代码往往无法直接使用 electron 运行。这时候你可能会想到重构项目，但往往重构项目代码的成本会远超你的预期，并且可能会带来更多的不可预知的问题。

如果你退一步，将 Electron 作为项目插件来运行的话，那么问题就可以轻松解决了。

```shell
npm install electron --save-dev
```

你可以在原来项目的下创建一个 `electron` 目录（你可以使用其他目录名称），并创建 main.ts、preload.ts 文件。

```typescript
// main.ts
import { app, BrowserWindow } from 'electron'
import * as path from 'path'

let mainWindow: Electron.BrowserWindow

function createWindow() {
  // 创建浏览器窗口
  mainWindow = new BrowserWindow({
    height: 600,
    width: 800,
  })

  // 加载应用入口页面，此处假设你的应用编译后的应用入口为 build/index.html
  mainWindow.loadFile(path.join(__dirname, '../build/index.html'))

  // 打开开发者工具
  mainWindow.webContents.openDevTools()

  // 在窗口关闭时触发
  mainWindow.on('closed', () => {
    mainWindow = null
  })
}

// Electron会在初始化完成并且准备好创建浏览器窗口时调用这个方法，部分 API 在 ready 事件触发后才能使用。
app.on('ready', createWindow)

// 当所有窗口都被关闭后退出
app.on('window-all-closed', () => {
  // 在 macOS 上，除非用户用 Cmd + Q 确定地退出，否则绝大部分应用及其菜单栏会保持激活。
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  // 在macOS上，当单击dock图标并且没有其他窗口打开时，通常在应用程序中重新创建一个窗口。
  if (mainWindow === null) {
    createWindow()
  }
})
```

最后你需要在你的 `package.json` 文件中的 `script` 处添加额外的 electron 脚本：

```json
{
  // 此处省略其他的配置内容
  "script": {
    "electron": "npm run build && tsc ./electron/*.ts -t ES5 -m commonjs && electron ./electron/main.js"
  }
}
```

如果你遇到一些 TypeScript 的编译错误提示，你可以尝试使用 `npm install @types/node --save-dev` 来解决。
