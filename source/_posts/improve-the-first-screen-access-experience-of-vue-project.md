---
title: vue 项目极限优化，提升首屏访问体验
author: 子丶言
date: 2020-12-28 17:53:47
tags: ['vue', '项目优化']
categories: ['Vue']
---

自从带队开发公司的运营类项目的开发，这半年来似乎一直在关注 vue 项目的性能优化。作为公司的用户引流项目，活动的首屏加载时间长短会对用户转换率产生一些”蝴蝶效应“。该影响对于弱网络的用户群体的影响尤为明显。如果一个页面白屏时间超过 2.5s，那么用户很可能会认为页面存在问题而关闭页面，从而导致用户流失。
<!-- more -->

> 项目的优化是一条无休止的路。

常规的项目优化方式我在 [优化 vue 项目包大小，提升首屏加载速度](/2020/06/optimize-package-size-of-vue-project/) 这篇文章中已经有所提及，可以优先阅读。

## 首屏加载视觉提速

何谓“视觉提速”？其实这是我自己造的的一个名词。我们经常会听到白屏时间这个名词，主要是只用户从打开网页到看到网页内容的时间。如果一个页面白屏时长超过 2.5s，那么用户很大概率会认为页面打不开而关闭页面，你再精彩的页面内容在用户看不到的前提下，都是“白费功夫”。所以你可以从网站找到很多有关首屏加载优化的方案。其中最常见的就是 SSR 的方案，但这个方案需要后端服务器配合，在公司没有强大运维能力支持的情况下，贸然尝试 SSR，你的项目很可能会受到“致命打击”。

那么除去 SSR 之外就没办法减少白屏时间么？

在思考这个问题之前，我们需要思考另外一个问题：*如果让你给 Vue 项目加 loading，你会怎么做？*

是在 `App.vue` 这类入口文件上加一个 loading 组件么，如果你这么做，那你已经超过了全国 50% 的用户了。但你可知，Vue 项目在库函数载入完并初始化完成之前都是白屏状态，是不是很意外？

再问另一个问题：*你觉得从用户角度而言白屏时间是否就等于 vue 项目初始化完成之前的时间？*

对于这个问题，如果你深入过群众你就会知道，我们跟用户的理解还是存在很大隔阂的。对于用户而言，只要看到屏幕上有内容，即便是最基本的 loading，也能让用户感到“安心”，至少不会认为页面打不开而关闭页面。

如果你能看懂以上论述，那么视觉提速就会比较好理解，我们可以在 vue 初始化之前就在 Html 页面埋入一个 loading，这类 loading 可以使用 css 实现，也可以考虑使用 svg、gif、静态占位图片甚至是一个背景色！总之，你需要让用户第一时间看到页面有内容。我个人比较推荐使用 svg，因为其他几种方式跟 svg 比都存在一些劣势。

```html
<div id="app">
  <div style="padding-top: 30vh; text-align: center">
    <div style="margin: auto">
      <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" style="margin: auto; display: block;"
      width="200px" height="200px" viewBox="0 0 100 100" preserveAspectRatio="xMidYMid">
        <g transform="translate(20 50)">
          <circle cx="0" cy="0" r="6" fill="#31ca89" transform="scale(0.00244898 0.00244898)">
            <animateTransform attributeName="transform" type="scale" begin="-0.375s" calcMode="spline"
              keySplines="0.3 0 0.7 1;0.3 0 0.7 1" values="0;1;0" keyTimes="0;0.5;1" dur="1s" repeatCount="indefinite">
            </animateTransform>
          </circle>
        </g>
        <g transform="translate(40 50)">
          <circle cx="0" cy="0" r="6" fill="#ef465d" transform="scale(0.197344 0.197344)">
            <animateTransform attributeName="transform" type="scale" begin="-0.25s" calcMode="spline"
              keySplines="0.3 0 0.7 1;0.3 0 0.7 1" values="0;1;0" keyTimes="0;0.5;1" dur="1s" repeatCount="indefinite">
            </animateTransform>
          </circle>
        </g>
        <g transform="translate(60 50)">
          <circle cx="0" cy="0" r="6" fill="#ff9936" transform="scale(0.537416 0.537416)">
            <animateTransform attributeName="transform" type="scale" begin="-0.125s" calcMode="spline"
              keySplines="0.3 0 0.7 1;0.3 0 0.7 1" values="0;1;0" keyTimes="0;0.5;1" dur="1s" repeatCount="indefinite">
            </animateTransform>
          </circle>
        </g>
        <g transform="translate(80 50)">
          <circle cx="0" cy="0" r="6" fill="#ffe700" transform="scale(0.862077 0.862077)">
            <animateTransform attributeName="transform" type="scale" begin="0s" calcMode="spline"
              keySplines="0.3 0 0.7 1;0.3 0 0.7 1" values="0;1;0" keyTimes="0;0.5;1" dur="1s" repeatCount="indefinite">
            </animateTransform>
          </circle>
        </g>
      </svg>
    </div>
  </div>
</div>
```

**注意：此处有一个细节，`id="app"` 一般是 vue 项目的挂载 dom，vue 在 dom 挂载之后会清空该 dom 的内容，因此我们可以不用再额外考虑 loading 加载完成后的移除问题了~**

## 资源加载可视化

我的项目采用了大量的 [lottie](https://airbnb.design/lottie/) 动画，而 [lottie-web](https://github.com/airbnb/lottie-web) 动画引擎库，对于网页来说简直是巨无霸般的存在！而且 lottie 脚本文件的体积也不可小觑。光 lottie-web 一个库可能就比你的 vue 项目都要大，在这种前提下，即便你使用了按需载入（用户很可能会遭遇页面“假死”），也很难在用户体验上做到很好的平衡。

既然不能“硬”来，也不能“胡”来，我们只能考虑另辟蹊径了。我们是否可以考虑将资源加载可视化呢？

如果你接触过游戏，你可能就已经见过这种模式了...什么，我都没听过这个词，你说我见过？

你玩过王者农药、吃鸡这类手游么？这类大型手游包含大量的资源文件，资源初始化需要比较长的一段时间，因此会采用以下的方式进行资源加载进度的显示。

![某游戏启动画面截图](https://gaeacdn.jiliguala.com/devjlgl/tmp/b2970b994763578b330268087c8c4c08.jpg)

这就是资源加载可视化，你可以通过加载进度告诉用户需要等待的预期时间。这种方式在游戏或者软件领域经常出现，但在前端领域出现频率非常低，毕竟网页从来都不是大型应用的首选载体。虽然出现频率低，不代表我们不能使用。

```vue
<template>
  <section class="loader" v-show="visible">
    <div class="progress-bar">
      <div class="progress" :style="{ width: `${currentProgress}%` }"></div>
    </div>
    <slot v-if="currentProgress === 100"></slot>
  </section>
</template>
```

```javascript
const lottiejs = () => {
  return import(/* webpackChunkName: "lottie" */ 'lottie-web')
}
const animate = () => {
  return import(/* webpackChunkName: "animate" */ '@/assets/animate.json')
}

const loader = [lottiejs(), animate()].map((item) => {
  return item
    .then((data) => {
      this.loaded += 1
      return data
    })
    .catch((err) => {
      console.error(err.message)
    })
})
Promise.all(loader).then((resp) => {
  this.$emit('loaded', resp)
})
const timer = setInterval(() => {
  if (this.currentProgress < 100) {
    this.currentProgress = Math.floor((this.loaded / loader.length) * 100)
    if (this.currentProgress > 100) this.currentProgress = 100
  } else {
    clearInterval(timer)
  }
}, 300)
```

以上代码截取了组件的核心实现逻辑。实现效果如下：

![前端界面效果](https://gaeacdn.jiliguala.com/devjlgl/tmp/42ac76a28964fd8bea0b2cceda297c20.png)

细心的人应该已经发现了，这里使用了 webpack 的按需载入方案。按需载入默认返回一个 Promise 对象，而 Promise 对象是可以进行统计的！我们利用这一点可以做到依赖的预加载且独立页面按需载入并减少了主依赖包的大小，简直是一举多得呐~

可为什么不直接使用 `const lottiejs = import(/* webpackChunkName: "lottie" */ 'lottie-web')`，而要多此一举的包一个函数呢？这个主要是为了避免项目在 webpack 打包过程中的资源加载策略，如果直接引用资源，那么 webpack 会认为这个资源时内联的，会提前进行加载，这对于既希望预加载一些必要依赖，又希望不加载全部依赖的场景下，这会非常有用！

## 图片最优化和预加载

新项目涉及了大量的图片素材，如果按照常规的加载方式，会极大的降低用户体验。你可以试想一下如下场景：你打开一个页面，页面上的图片还在加载，但在加载过程中，半张图片还在执行动画效果...这个场景犹如，你在逛街突然看到半个身影，在街上飘...着实有些诡异！

### 图片预加载

解决当前页面图片加载过慢的情况一般可以考虑预加载的方式。但这里提到的预加载与传统方式的预加载还有些许不同，主要在于当前项目的图片分散在不同的页面间，因此传统的针对屏外图片预加载并不适合当下的场景。我们希望图片在页面出现之前就已经加载完成，进入到页面后，图片资源应该从浏览器文件缓存中直接读取。其实要做到这一点很简单，你只需要在进入页面前在后台请求一次相同资源路径的图片，浏览器在第二次访问时就会直接使用文件缓存中图片，而不再从服务器（或CDN）上下载图片。  
既然知道了原理，那么就很容易实现对应的加载逻辑：

```javascript
function loadImage(url) {
  return new Promise((resolve, reject) => {
    // 利用新建一个图片实例而不渲染实现图片预加载
    const image = new Image()
    image.onload = () => resolve(url)
    image.onerror = () => reject(new Error(`Image preload error: ${url}`))
    image.src = url
  })
}
```

预加载实际上是在后台提前偷跑图片加载流程，如果你需要预加载的图片资源非常多，势必会影响正常的网络数据，我们坚决不做“偷鸡不成蚀把米”的事！在 Http2 时代，你可能知道浏览器已经支持了资源的并行加载，我们可以既要竟可能地减少对项目的正常加载的影响，又要进行资源“偷跑”，我们可以考虑把图片加载任务进行分配加载，一批图片加载完了，再加载下一批图片。w(ﾟДﾟ)w，原来还能这么玩？

```javascript
// thread，表示每一批的资源数量
function preloadImage(images, thread = 3) {
  const tasks = []
  images.splice(0, thread).forEach((url) => {
    tasks.push(loadImage(url))
  })
  return Promise.all(tasks).then(() => {
    if (images.length > 0) preloadImage(images, thread)
  })
}
```

### 图片最优化

图片资源大小对加载速度有着最直接的影响。这个道理几乎所有人都知道，但很少有人去尝试最优解。我目前的项目使用 CDN 的方式对图片进行加载优化，但这种方式只能说是“治标不治本”。如果一张图片有几 M 大小，即便使用最快的 CDN，也会让绝大多数用户感觉到图片加载慢。所以我们一般在图片上传到 CDN 之前，对图片进行一次压缩，对图片的压缩实现有很多种方式，我推荐使用 [Tinypng](https://tinypng.com/) 提供的在线图片压缩服务。  
可现实总是残酷的，在设计师提供的 3倍图的场景下，有些图片即便是压缩也很可能会非常大，但我们真的对此无能为力了么？也并不是，你可以使用 CSS3 的 `media query` 或 `image-set`属性，对不同屏幕分辨率的图片进行设置，这就需要设计师把所有场景下的图片都提供给你，然后你需要在所有用到该图片的地方都需要设置！这不仅增加了你的工作量，连设计师都需要花时间额外给你提供对应的图片素材。这对于小公司而言，是一种相当“耗费”人力的做法，性价比极低...正因为性价比低，所以国内很少有公司采用这种方式来做图片加载优化。  
如果“人工”的方式不好走，那何不试试“智能”的方式呢？很多 CDN 服务商都有提供图片瘦身的方案，这里以七牛 CDN 为例：

```javascript
let checkedWebp = false
function slimImages(images) {
  if (checkedWebp) return images
  // checkWebpFeature 函数的实现，你可以参考网上的方案，这里由于篇幅原因不再展开
  return checkWebpFeature('alpha')
    .then((supportWebp) => {
      if (supportWebp) {
        _.entries(images).forEach(([key, val]) => {
          images[key] = `${val}?imageMogr2/thumbnail/!${Math.ceil((window.devicePixelRatio / 3) * 100) || 100}p/format/webp`
        })
      }
      return images
    })
    .finally(() => {
      checkedWebp = true
    })
}
```

上述代码主要做了两处优化：

1、不同分辨率的设备使用对应尺寸的图片 `Math.ceil((window.devicePixelRatio / 3) * 100)`，七牛云的 `thumbnail/!100p` 指令可以生成对应比例的图片，其中 100p 代表原始大小。
2、使用了 webp 格式，webp 是 Google 早期推出的一种高压缩的图片格式。这种图片格式据说曾帮助淘宝每年节省好几亿的流量费，Amazing！

#### 总结

通过以上优化，你可以看到图片在 1倍屏下得到了极大的优化：

![原始图片](https://gaeacdn.jiliguala.com/devjlgl/tmp/a4493e1b4402864ad0c8a334e1612d3e.png)
原始图片

![使用 Tinypng 压缩后的图片](https://gaeacdn.jiliguala.com/devjlgl/tmp/eb10b6a816482825c80e07d2b4bf7272.png)
使用 Tinypng 压缩后的图片

![借助 CDN 瘦身后的 2倍屏图片](https://gaeacdn.jiliguala.com/devjlgl/tmp/ed0d5cfd2ec16075695bf6d110757c07.png)
借助 CDN 瘦身后的 2倍屏图片

![借助 CDN 瘦身后的 1倍屏图片](https://gaeacdn.jiliguala.com/devjlgl/tmp/93ef51d5872dae2fab5787fac0a7b6f6.png)
借助 CDN 瘦身后的 1倍屏图片


