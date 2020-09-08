---
title: Electron 学习笔记 - 使用 Bridge 通信模式解耦 Electron 逻辑
author: 子丶言
date: 2020-06-04 18:26:22
tags: ['Electron', 'Electron 学习笔记', '跨平台', '桌面应用', 'Bridge 通信']
categories: ['Electron 学习笔记']
---

虽然官方允许你直接在前端业务代码中直接使用 Electron 甚至直接引用 Node.js 依赖，但这种方式却对业务无意间侵入了前端业务代码。当项目加载远端业务页面以及业务代码由其他项目打包工具生成的情况下，你可能无法对前端业务代码做修改。如果引入 Native 与 HTML 页面的 Bridge 通信模式，这两类问题将迎刃而解。
<!-- more --> 

## Electron 业务通信逻辑

Electron 提供了 [ipcMain](https://www.electronjs.org/docs/api/ipc-main) 与 ipcRenderer 模块用以主进程与渲染进程之间的通信。

```javascript
// 在主进程中.
const { ipcMain } = require('electron')
ipcMain.on('asynchronous-message', (event, arg) => {
  console.log(arg) // 输出 "ping"
  event.reply('asynchronous-reply', 'pong')
})

ipcMain.on('synchronous-message', (event, arg) => {
  console.log(arg) // 输出 "ping"
  event.returnValue = 'pong'
})
```

```javascript
// 在渲染器进程 (网页) 中。
const { ipcRenderer } = require('electron')
console.log(ipcRenderer.sendSync('synchronous-message', 'ping')) // 输出 "pong"

ipcRenderer.on('asynchronous-reply', (event, arg) => {
  console.log(arg) // 输出 "pong"
})
ipcRenderer.send('asynchronous-message', 'ping')
```

如果你想在你的前端页面使用这种通信方式，那么你需要在 webview 中配置 `<webview src="http://www.google.com/" nodeintegration></webview>` 或者在 BrowserWindow 实例中的 webPreferences 属性里配置 `nodeIntegration: true`。这虽然能够解决进程间的通信问题，但也将 Node.js 环境引入了前端业务页面。

## 使用 Bridge 通信模式解决 Electron 逻辑耦合问题

当你加载的页面是一个外部页面，那么你可能需要在其他的项目里引入 Electron 作为项目依赖，那么 Electron 在无意间入侵了该项目的业务逻辑。倘若我们将 Electron 的进程通信方法封装为 Bridge 的模式，那么我们可以将业务代码与 Electron 解耦为 Bridge 间的通信。在 Bridge 通信模式下，你可以更好的处理业务降级工作。

以下我将提供我目前在业务使用的一种 jsBridge 通信方式。

### Electron 使用 contextBridge 注册 Native Bridge

Electron 提供了 [contextBridge](https://www.electronjs.org/docs/api/context-bridge) 用以注册 Native Bridge。在这之前你需要在 BrowserWindow 实例中的 webPreferences 属性里配置 `contextIsolation: true` 以及 `preload: path.join(__dirname, 'preload.js')` 引入 preload.js。

你需要在 preload.js 页面中引入以下逻辑：

```javascript
// Preload (Isolated World)
const { contextBridge, ipcRenderer } = require('electron')

contextBridge.exposeInMainWorld(
  'electron',
  {
    doThing: () => ipcRenderer.send('do-a-thing')
  }
)
```

注册完 Native Bridge 之后，你就可以直接在页面中调用你注册过的方法。

```javascript
// Renderer (Main World)
window.electron.doThing()
```

即便载入的页面是一个远端页面，并且该页面在不引入 Electron 的情况下，你也完全可以使用以上方法直接与 Electron 进行交互。

#### 使用 jsBridge 替代直接调用 Native Bridge

你可以对 Native Bridge 进行二次封装，这样你的页面代码就可以优雅降级了。

```javascript
window.jsBridge = {
  doThing: () => {
    if (typeof window.electron === 'object') {
      window.electron.doThing()
    } else {
      // 你可以在这里使用更完善的降级处理方案。
      console.log('Non-electron environment!')
    }
  }
}
```

这样你就可以在前端业务代码中大胆的使用 Bridge 通信模式了。

```javascript
window.jsBridge.doThing()
```

### 更完善的 Native Bridge 逻辑

之前已经说明了如何借助 [contextBridge](https://www.electronjs.org/docs/api/context-bridge) 来实现 Native Bridge。但之前的实现无法做到异步的数据回调通信。比如，你需要通过借助 Electron 的 Node.js 能力处理大量的数据，而这些数据的处理相当耗时，你不应当采用同步回调的模式与 Electron 进行通信，因为这样做会导致你的页面进入”假死“的状态。但如果采用异步的方式，你可能需要在前端业务代码中引入 Electron 的 ipcRenderer 方法用以进程间的通信。哎，怎么又绕回原地了呢？但不要这么早放弃希望，因为你可以通过实现一套完整的 jsBridge 的方式来解决这个问题，以下是提供我实现的一种支持异步回调的简易通信方式，你可以作为参考。

```javascript
// Main
ipcMain.on('postMessage', async (event, message) => {
  switch (message.bridgeName) {
    case 'doThing':
      try {
        const result = await doThing(message.data)
        event.reply('receiveMessage', {
          bridgeName: 'doThing',
          cid: message.cid,
          data: result,
        },)
      } catch (err) {
        event.reply('receiveMessage', {
          bridgeName: 'doThing',
          error: { code: 500, message: err.message },
        })
      }
      break
    default:
      break
  }
})
```

```javascript
// Preload (Isolated World)
const { contextBridge, ipcRenderer } = require('electron')

let cid = 0
const callbacks = {}

// 注册 nativeBridge
contextBridge.exposeInMainWorld(
  'electron',
  {
    invoke (bridgeName, data, callback) {
      // 如果不存在方法名或不为字符串，则提示调用失败
      if (typeof bridgeName !== 'string') {
        throw new Error('Invoke failed!')
      }
      // 与 Native 的通信信息
      const message = { bridgeName }
      if (typeof data !== 'undefined' || data !== null) {
        message.data = data
      }
      if (typeof callback !== 'function') {
        callback = () => null
      }
      cid = cid + 1
      // 存储回调函数
      callbacks[cid] = callback
      message.cid = cid
      ipcRenderer.send('postMessage', message)
      ipcRenderer.once('receiveMessage', (_, message): void => {
        const { data, cid, error } = message
        // 如果存在方法名，则调用对应函数
        if (typeof cid === 'number' && cid >= 1) {
          if (typeof error !== 'undefined') {
            callbacks[cid](error)
            delete callbacks[cid]
          } else if (callbacks[cid]) {
            callbacks[cid](null, data)
            delete callbacks[cid]
          } else {
            throw new Error('Invalid callback id')
          }
        } else {
          throw new Error('message format error')
        }
      })
    }
  }
)
```

```javascript
window.jsBridge = {
  doThing: (data, callback) => {
    if (typeof window.electron === 'object') {
      window.electron.invoke('doThing', data, callback)
    } else {
      // 你可以在这里使用更完善的降级处理方案。
      console.log('Non-electron environment!')
    }
  }
}
```

```javascript
window.jsBridge.doThing({}, () => {
  console.log('finished')
})
```
