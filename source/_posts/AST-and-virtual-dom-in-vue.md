---
title: Vue 中的 AST 与虚拟 Dom
author: 子丶言
date: 2021-01-14 14:07:52
tags: ['vue', 'AST', 'virtual dom', '虚拟 Dom', '抽象语法树']
categories: ['Vue']
---

在 Vue 的概念中有两个最基础也最难理解的概念 AST 与虚拟 Dom。总有很多新手把这两个概念搞混，以为 AST 就是虚拟 Dom，或认为虚拟 Dom 的结构才叫 AST，这其实都是错的。网上有不少半壶水的专家，张口闭口，三句不离这两个词汇，但他们真正能说明白的少之又少，说错的更不在少数...我不敢保证我的解答都是正确的，但我能保证在讲述过程中有查阅过相关资料，本文仅供参考。
<!-- more -->

### 抽象语法树(Abstract syntax tree)

> 在计算机科学中，称之为抽象语法树，或简称语法树，是源代码语法结构的一种抽象表示。它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。之所以说语法是“抽象”的，是因为这里的语法并不会表示出真实语法中出现的每个细节。[维基百科](https://zh.wikipedia.org/zh-cn/%E6%8A%BD%E8%B1%A1%E8%AA%9E%E6%B3%95%E6%A8%B9)

从维基百科的描述可知，AST(Abstract syntax tree) 是一种语法结构的抽象表示。[AST Explorer](https://astexplorer.net/) 这个网站可以帮你更加直观的展示 AST 的抽象结构。比如以下代码：

```html
<div class="demo">
  <span class="text">hello world</span>
</div>
```

在 HTML 解析引擎下会生成负载的抽象结构：

![AST](https://gaeacdn.jiliguala.com/devjlgl/tmp/426787a8c8fa353ea63efaf0abacc9f3.png)

看上去就想是一个大的结构化对象，非常接近于 JSON 的形式。你不需要去理解这个结构的意义，因为 AST 不是给开发人员阅读的，而是提供给编译器做语义分析用的。AST 的具体概念我这里不做展开，感兴趣的朋友可以阅读 [AST抽象语法树——最基础的javascript重点知识，99%的人根本不了解](https://segmentfault.com/a/1190000016231512) 这篇文章。


### 虚拟 Dom(Virtual Dom)

虚拟 Dom 的概念并不是 Vue 项目最先提出的，但我们可以在 Vue 的官方文档中找到对应的解释：

    Vue 通过建立一个虚拟 DOM 来追踪自己要如何改变真实 DOM。请仔细看这行代码：

    ```js
    return createElement('h1', this.blogTitle)
    ```

    `createElement` 到底会返回什么呢？其实不是一个实际的 DOM 元素。它更准确的名字可能是 `createNodeDescription`，因为它所包含的信息会告诉 Vue 页面上需要渲染什么样的节点，包括及其子节点的描述信息。我们把这样的节点描述为“虚拟节点 (virtual node)”，也常简写它为“VNode”。“虚拟 DOM”是我们对由 Vue 组件树建立起来的整个 VNode 树的称呼。

由此可知，Vue 项目的虚拟 Dom 是 Vue 整个 `VNode` 树的称呼。

#### 虚拟 Dom 的优势

如果你经历过 JQuery 的时代，你应该知道 Dom 也是一个对象，那虚拟 Dom 与真实 Dom 相比有什么优势么？

在回答这个问题之前，我们先来看一下原生的 Dom 长啥样吧！我们可以使用以下代码打印一个纯粹的 div 的 Dom 结构:

```js
console.dir(document.createElement('div'))
```

![div 的 Dom 对象内容](https://gaeacdn.jiliguala.com/devjlgl/tmp/b7cca713ac0c458635b463d74b81a574.png)

以上只截取了一小部分属性，你会发现这是一个相当复杂的对象，那虚拟 Dom 又长啥样呢？

```vue
<template>
  <div class="demo">
    <span class="text">hello world</span>
  </div>
</template>
```

以上代码的虚拟 Dom 的结构大概是这样的：

```js
{
  tag: 'div'
  data: {
    class: 'demo'
  },
  children: [
    {
      tag: 'span',
      data: {
        class: 'text'
      }
      text: 'hello world'
    }
  ]
}
```

w(ﾟДﾟ)w，两者差距太大了，简直是柚子和橘子的区别。

虚拟 Dom 主要是用于渲染前的状态比较的，也就是我们经常提到的 diff 过程，不参与 UI 的展示，因此结构上异常精简。

回到之前的问题，虚拟 Dom 的主要优势其实就是结构简单！


### AST 与 虚拟 Dom 的关系

上面的内容分别介绍了 AST 与 虚拟 Dom 这两个概念，那这两个又有什么关系呢？

**这两者其实并没有关系**╮(╯▽╰)╭。因为他们是在不同的阶段的产物。

AST 大多是在编译过程中出现，常规的开发框架并不涉及 AST 的概念。Vue 中 `.vue` 后缀的文件使用了 ”All in one file“ 的形式，不再是传统的 js 文件，其中 `<template></template>` 标签中的内容更包含了一些自定义的 vue 语法，比如 `v-if`、`v-for`，这些语法即不属于 js 规范，更不是 html 的语法，常规的 js 引擎无法处理，因此在交给 js 引擎运行之前，需要转换成标准的 js 语法，这时候就轮到 AST 出场了。vue-cli 会将模板代码解析成 AST 语法结构，再转换成 js 代码，即我们常说的 `build` 过程。

虚拟 Dom 是在运行时（vue-runtime）出现的一种数据中间态。Vue 项目在每个 render 周期内都会产生一次虚拟 Dom，如果每次都使用虚拟 Dom 去重新渲染整个 UI，那么页面不可避免地会出现”闪屏“的现象。因此在 Vue 项目内部会先对新旧虚拟 Dom 进行一次 `diff` 操作，获取发生变化的节点内容，然后获取最小的改变内容进行 UI 渲染，这也是虚拟 Dom 比真实 Dom 高效的具体原因。所以那种虚拟 Dom 比真实 Dom 操作快的说法是错误的，因为虚拟 Dom 最终还是会渲染成真实 Dom，只是一个过程的先后罢了。

AST 是在编译过程中出现的，但项目运行时就不再有 AST 的概念了，而虚拟 Dom 是在 render 过程中出现的，主要用于比较 render 前后的数据结构变化。

在前端技术日新月异的时代，各类新技术、新名词不断涌现，你在使用过程中，需要知其然，而非人云亦云。
