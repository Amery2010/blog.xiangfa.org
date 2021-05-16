---
title: 让 Hexo 支持 LaTeX 公式
author: 子丶言
date: 2020-09-02 20:11:42
mathjax: true
tags: ['Hexo', 'LaTex']
categories: ['Hexo']
---

通常情况下，你并不会去使用 LaTex 公式，但当你文章涉及数学公式时，你往往会在第一时间想到 $ LaTex $ 公式。网上有比较多的文章告诉你如何在 Hexo 里使用 $ LaTex $ 公式，但大部分都是推荐使用 `hexo-math` 插件来实现，但 `hexo-math` 的兼容性并不好，并且和 Hexo 内置的 MarkDown 渲染引擎存在冲突。因此我推荐使用另一种实现方案：通过增加 Hexo 主题脚本片段的方式实现 $ LaTex $ 语法高亮。

<!-- more -->

## 让 Hexo 主题使用 Mathjax

你可以从主题文件的 `README.md` 确认你的 Hexo 主题是否已经引入了 `Mathjax`，如果已经引入，那你只要按照说明来开启 `Mathjax` 支持即可。如果你的主题并不支持 `Mathjax`，那你可以按照以下步骤使你的 Hexo 支持 `Mathjax`。

1、你需要在 `themes/YourThemeName/layout/` 下新建文件 `mathjax.ejs` 文件。

```html
<% if (theme.mathjax.enable){ %>
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    tex2jax: {
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      processEscapes: true,
      skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
    }
  });
  MathJax.Hub.Queue(function() {
    var all = MathJax.Hub.getAllJax(), i;
    for(i=0; i < all.length; i += 1) {
        all[i].SourceElement().parentNode.className += ' has-jax';
    }
  });
</script>
<script type="text/javascript" src="<%- theme.mathjax.cdn %>"></script>
<% } %>
```
 
2、你需要在 `themes/YourThemeName/_config.yml` 末尾追加：

```yml
# MathJax Support
mathjax:
  enable: true
  cdn: https://cdn.jsdelivr.net/npm/mathjax@2.7.8/MathJax.js?config=TeX-AMS-MML_HTMLorMML
```

3、修改 `themes/YourThemeName/layout/post.ejs` 文件，在中间添加：

```html
<% if (theme.mathjax){ %>
  <%- partial('mathjax') %>
<% } %>
```

4、最好你需要在你用到 LaTex 公式的文章顶部配置里追加 Mathjax 配置：

```yml
---
title: 文章标题
date: 2020-09-02 20:11:42
mathjax: true
---
```

至此，大功告成~
你最后只需要重新生成一下文章就可以看到你的文章现在已经支持 LaTex 公式了。

## 测试

测试一下你的文章是否已经支持 LaTex 公式：

```bash
$$ J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \Gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha} $$
```

如果你的文章已经支持那么上面的 LaTex 语法将会有如下显示：

$$ J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \Gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha} $$
