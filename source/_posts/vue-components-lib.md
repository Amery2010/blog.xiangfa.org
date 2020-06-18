---
title: Vue 组件库构建流程
author: 子丶言
date: 2020-05-25 18:45:15
tags: ['Vue 组件库', 'Vue', '组件库']
categories: 
  - Vue
---

```
Leader：我们最近好几个项目都有用到相似的业务逻辑，比如，手机验证码注册功能最好能有一个组件库来支持，这样就不需要要多次编写相似的代码了。
Me：我..
Leader：做一个 vue 的组件库应该没什么问题吧。
Me：额..
Leader：我记得你 OKR 里有写到这个。
Me：没问题！
```
~~以上内容纯属虚构~~
<!-- more --> 

## 查找组件库

既然要开始构建一个 vue 的组件库，特别是针对移动端 App 的组件库，首先需要的做到就是去了解是否有成熟的组件可以直接用。虽然搜出来的 vue 的组件库并不少，但像 iView、element UI 和 Ant Design Vue 这类组件库都是 focus 在后台开发领域的，并不适用于移动端，而移动端目前比较完善的只有 [Vant](https://github.com/youzan/vant) 这个有赞开发的 vue 组件库了。~~有赞，打钱！~~

Vant 这个库组件种类还是很丰富的，但..我接到的需求却是开发一个功能型的组件库，主要是实现一些公司内部常用业务的组件开发。~~做程序员实在是太南了。~~

既然没办法~~直接拿 vant 交差~~解决问题，那是不是可以借鉴 Vant 的方案来做一个 vue 组件库呢？

![打包脚本](https://h5.jiliguala.com/activity/bf6a3b5f7598acea22631a56026e4236.png)

当看到这么多打包脚本就直接把我劝退了。既然~~Fork~~借鉴无望，那就只能撸起袖子自己干了。

## 如何开始构建工作？

如果之前没有相关经验，当然是 Google 一下咯！

不错~有人已经实践过了，那就不需要自己再~~造轮子~~开发了，那就点开前两个链接看一下别人是怎么去实践的。*掘金，打钱！*

- [手把手带你撸一个vue组件库！](https://juejin.im/post/5afcd516f265da0b9e65414b)
- [详解：Vue cli3 库模式搭建组件库并发布到npm](https://juejin.im/post/5bbab9de5188255c8c0cb0e3)

第一个链接中的文章写得很细致，是一个完整的从0到1的构建流程，不过，我并不打算完全采用该方案。主要理由是...该文章发布于 2018年 5月 17日，啊，一年半了呀，vue-cli 都从 2 进化到 3 了！文章的前半部分目前依然适用，但打包发布部分已经过时。文章采用的打包发布方案用的依然是裸写 webpack 配置的方案，该方案虽然可行，但过于繁琐，而且需要开发者了解 webpack 的配置写法，其次 vue-cli 3 默认隐藏了 webpack 的相关配置，官方都不建议用户直接接触 webpack 配置了，我们为什么还要去简就繁呢？~~用好webpack太南了~~其实 vue-cli 3 已经有官方的方案了...

那我们点开第二个链接再来看一下另一个方案。图文并茂，而且是非常完整的从构建到发布的整一个流程！而且文章里采用的打包方案是基于 vue-cli 3 的，更重要的是文章告诉我一个信息：vue-cli 3 已经自带了组件的打包脚本。

```
"scripts": {
  // ...
  "lib": "vue-cli-service build --target lib --name vcolorpicker --dest lib packages/index.js"
}
```

我根据文章的开发步骤，成功生成了一个组件，并通过编写多个 packages 可以生成一个组件库。
等等..好像有些不对劲？

为什么我编写了多个 packages 打包出来的依然是一个文件...虽然打包成单文件还是多文件对于组件库来说影响不大，但对使用者而言，如果为了使用组件库里一个组件，而需要引入一整个组件库...~~这还玩个球呀~~

## 如何实现多文件打包？

既然遇到了问题，那就想办法去解决一下吧！既然 vue-cli 3 自带了打包脚本，那就去官方文档上看一下，没准就有打包成多文件的方案呢。

[Vue Cli 指南](https://cli.vuejs.org/zh/guide/)

很快我就找到了**构建目标**章节。里面有构建库的方案，也有构建应用和 Web Components 的方案。其中，构建库的方案~~看来之前那位作者是完全信仰了尤大大呀~~跟上文提到的完全一致。

![官网构建方案](https://h5.jiliguala.com/activity/3aaceb16520d5c54e01cdc2310e95b2d.png)

通过 `--target lib` 默认生成 umd、umd.min 和 common 格式的文件以及 css。通过 Web Components 可以生成单个或多个 Web Components 组件...哎？我怎么是多个 Web Components 组件的打包方案，我要到多文件 lib 方案呢？我又看了一遍文档，依然没有发现我希望的多文件组件库方案，难道是尤大大忘了写进文档了？感觉又走进了死胡同了...

不行？咱就再 Google 一下吧！

- [组件库之按需加载](https://juejin.im/post/5d2c248151882556d1683363)
- [教你搭建按需加载的Vue组件库](https://juejin.im/post/5d3e4a66f265da1b7c6161ee)

第一篇文章给你科普的是“什么叫按需加载”，完全不是我想要的内容。
第二篇文章虽然有提到组件库的打包，但实际上是教你如何使用 `babel-plugin-component` 实现组件库的按需加载。

那我换一个关键词 Google 一下试试？"vue组件库 构建 按需加载"

[vue-cli 3，构建lib模式的时候，怎么才能做到把包按组件进行 ...](https://github.com/vuejs/vue-cli/issues/2851)

搜索出来的文章大多跟我想要解决的问题无关，倒是其中一个 github/vue-cli 的 issue 引起了我的注意。原来也有人再苦恼我遇到的问题，而且还给官方提了 issue，心喜，难道我的问题有解了？可事实总是残酷的，这个 issue 里满满的负能量...总之，三个字“不支持”。

兜兜转转，还是回到了原地。既然 vue-cli 3 支持打包单文件组件，那我是不是可以~~魔改~~修改 vue-cli 的打包脚本实现一个打包多文件的方案呢？

那我就从 vue-cli 的源码里开始找吧...

经过漫长的查找定位...终于找到了我需要的打包 lib 的[源码](https://github.com/vuejs/vue-cli/blob/dev/packages/%40vue/cli-service/lib/commands/build/index.js)：

```
// https://github.com/vuejs/vue-cli/blob/master/packages/@vue/cli-service/lib/commands/build/index.js
vue-cli/packages/@vue/cli-service/lib/commands/build/index.js
```

~~鸡汁如我，藏得这么深都能被我找到。~~

不看源码还不知道，原来 vue-cli-service 有着这么多的可选参数：

```
options: {
  '--mode': `specify env mode (default: production)`,
  '--dest': `specify output directory (default: ${options.outputDir})`,
  '--modern': `build app targeting modern browsers with auto fallback`,
  '--no-unsafe-inline': `build app without introducing inline scripts`,
  '--target': `app | lib | wc | wc-async (default: ${defaults.target})`,
  '--inline-vue': 'include the Vue module in the final bundle of library or web component target',
  '--formats': `list of output formats for library builds (default: ${defaults.formats})`,
  '--name': `name for lib or web-component mode (default: "name" in package.json or entry filename)`,
  '--filename': `file name for output, only usable for 'lib' target (default: value of --name)`,
  '--no-clean': `do not remove the dist directory before building the project`,
  '--report': `generate report.html to help analyze bundle content`,
  '--report-json': 'generate report.json to help analyze bundle content',
  '--skip-plugins': `comma-separated list of plugin names to skip for this run`,
  '--watch': `watch for changes`
}
```

可...似乎并没有设置多文件的打包方案。原以为“柳暗花明”，结果依然“山穷水尽”。都走了这么远的路了，怎么可以就此放弃！

## 从头开始思考

既然没办法走阳关大道，那只要有路还是可以走一走的。

通过之前的思考，目前我可以掌握以下三个关键信息：

- vue-cli 3 支持打包组件
- 可以通过 npm 依赖实现组件的按需加载（使用时）
- 可以通过自己编写脚本实现能够支持按需加载的组件库

通过以上信息，可以证明实现支持按需加载的组件库还是有可能的。~~啊，呸，我早就知道了，可惜臣妾还是做不到呀！~~

那么我们不妨来综合一下以上三点：通过脚本多次执行 vue-cli 的打包命令，生成可以直接按需加载的组件库结构。

看来有戏！

vue-cli 的脚本是通过命令行执行的，那么可以借助 [shelljs](https://www.npmjs.com/package/shelljs) 可以实现。但生成的 umd、umd.min 以及 demo.html 文件都不是我们需要的，当然我们可以借助 shelljs 在构建后将他们删除。那有没有更好的方式让他们一开始就不生成呢？

我们回过头去 vue-cli-service 的脚本配置，其中的 `--formats` 选项虽然在官方文档里没有提及，但实际上是可用的，通过设置 `--formats commonjs` 可以很容易的实现在构建时只生成 `index.common.js` 和 `index.css` 这两个文件。

那有没有办法再把 `index.common.js` 变成 `index.js` 呢？关是猜，当时是没有用的，既然文档里都没提及 `--formats`，自然也不会有相应的使用说明，我们还得去看 vue-cli 的源码！

```
// https://github.com/vuejs/vue-cli/blob/master/packages/@vue/cli-service/lib/commands/build/resolveLibConfig.js
vue-cli/packages/@vue/cli-service/lib/commands/build/resolveLibConfig.js
```

通过 vue-cli-service 的依赖引用，我很快定位到了这个文件，但也很快确定了一点，没有参数可以移除 common 后缀，因为 common 文件的导出是固定带 common 后缀的！

既然此路不同..那我就绕道使用 shelljs 在构建后移除对应的 common 后缀吧。为什么不早这么做呢，因为我~~比较懒~~相信尤大大啊！

默认导出的目录结构显然不能满足按需加载插件的需求，所以很多成熟的组件库都会自己写脚本去生成对应目录结构。但我既然已经上了 vue-cli 的车了，总不该半路“跳车”吧。~~我真的懒得写那么复杂的构建脚本...~~解决方案便是再次借助万能的 shelljs 来实现~

## 打包脚本代码

完整的打包代码如下：

```js
const shell = require('shelljs')
const version = require('../package.json').version

let scripts = []

shell.ls('packages').forEach(file => {
  /**
   * 由于 vue-cli-service 在 build 过程中会先删除导出的父级目录，
   * 因此需要先执行主入口文件的打包命令
   */
  if (file === 'index.js') {
    scripts.unshift({
      type: 'index',
      script: 'vue-cli-service build --target lib --name index --formats commonjs --dest lib packages/index.js',
    })
  } else {
    scripts.push({
      type: 'package',
      filename: file,
      script: `vue-cli-service build --target lib --name index --formats commonjs --dest lib/${file} packages/${file}/index.js`,
    })
  }
})

scripts.forEach(config => {
  if (config.type === 'index') {
    shell.exec(config.script)
    /**
     * 目前构建项目利用的是 vue-cli 原生的 lib 构建方案，
     * 但目前并不支持生成不带 .common 后缀的文件，
     * 此处脚本就是用于 hack 该问题的代码
     */
    shell.mv('lib/index.common.js', 'lib/vcomp.js')
    shell.mv('lib/index.css', 'lib/vcomp.css')
    // 更新组件库版本号
    shell.sed('-i', '0.0.0', version, 'lib/vcomp.js')
  } else if (config.type === 'package') {
    shell.exec(config.script)
    shell.mv(`lib/${config.filename}/index.common.js`, `lib/${config.filename}/index.js`)
    shell.mkdir(`lib/${config.filename}/style`)
    shell.echo(`require('../index.css')`).to(`lib/${config.filename}/style/index.js`)
  }
})
```

至此，通过“曲线救国”的方式，我顺利的完成了 vue 组件库开发的任务。

注意，编译后的项目最好避免使用有副作用的 `postcss` 插件，有些 `postcss` 插件，比如 `postcss-px2rem` 会影响项目的构建，属于有副作用类型的插件。
