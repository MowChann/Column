---
layout: post
title: 让KaTeX支持\ref和\label（引用、标签）功能
subtitle: 玩转KaTeX，宏真是个好东西！
date: 2021-05-19 17:00:00
update: 2021-08-17 23:45:00
categories: [Frontend]
author: MowChan
tags: [前端, KaTeX, 数学公式, 引用, LaTeX]
latex: true
nightly: true
incomplete: false
---



最近在使用 KaTeX 时发现，一般 LaTeX 引擎或者 MathJax 支持的 `\ref`、`\label`功能在KaTeX中是没有的。而且尽管在 Github 上提出加入这两个指令的 issue 很多，但官方明确说明目前是没有这个计划的。

受一条 [Issue](https://github.com/falgon/roki-web/issues/34#issuecomment-673056997) 启发，我尝试用 KaTeX 的宏功能结合其它可用的指令模拟了`\ref`、`\label`功能，效果很好。

首先需要对 KaTeX 进行配置。

KaTeX 默认是禁用`\htmlId`、`\href`命令的，因此需要先信任这两条指令。（KaTeX 的 `trust` 配置值类型为**布尔型**或**函数**，我之前的**数组**类型是不符合要求的，在 [GitHub 上的讨论](https://github.com/KaTeX/KaTeX/issues/2003#issuecomment-863923688) 指出了这一点错误，正确的配置方法参考[官方文档](https://github.com/KaTeX/KaTeX/blob/master/docs/options.md)）

```javascript
trust: (context) => ['\\htmlId', '\\href'].includes(context.command),
macros: {
  "\\eqref": "\\href{###1}{(\\text{#1})}",
  "\\ref": "\\href{###1}{\\text{#1}}",
  "\\label": "\\htmlId{#1}{}"
}
```

然后在配置中定义我们需要的宏。

在KaTeX的宏定义中，可以以`#数字`的形式设置占位符，按顺序对应定义的参数。如`\test{1}{2}{3}`宏指令的定义若要依次引用第1、2、3个参数，就以`#1`、`#2`、`#3`占位即可。占位符可重复引用，不同的占位符最多9个。

`\label`指令用`\htmlId`模拟，用于生成页面内锚点，在LaTeX中与`\tag`或`\caption`配合使用，所以我们在这里也将`\htmlId`的第二个参数留空，在页面上不做显示。

`\ref`和`\eqref`指令都是引用，区别在于后者会加上括号，一般用于公式的引用。我们用`\href`指令来模拟，第一个参数是跳转的链接，第二个指令是引用显示。一般引用显示都是数字或字母的编号，为了避免被 KaTeX 引擎识别为变量，加一个`\text`指令。需要注意的是，由于页面锚点的URL也是用`#`表示，这会引发和宏定义占位符的冲突。所幸 KaTeX 的引擎考虑到了这一点，使用两个`#`，也就是`##`就可以表示`#`本身，而不会被当作占位符解析（其实是我试出来的）。

样例如下

```html
Search by definition $\eqref{1-3}$ or $\ref{1-3}$.

<br>
<br>

$$
\begin{aligned} \sin 2\theta = 2\sin \theta \cos \theta \\ = \cfrac{2 \tan \theta}{1+\tan^2 \theta} \end{aligned} \label{1-3}\tag{1-3}
$$
```

经过测试，指令内所填的引用名无论是数字、英文还是常用的字符组合都可以正常使用，如图

![](https://cdn.jsdelivr.net/gh/mowchann/blog-static/image/column/post/2021/20210519-180634.webp)



#### 参考链接

[Referencing formula numbers in articles · Issue #34 · falgon/roki-web (github.com)](https://github.com/falgon/roki-web/issues/34)

[Macros - Supported Functions · KaTeX](https://katex.org/docs/supported.html#macros)

[HTML - Supported Functions · KaTeX](https://katex.org/docs/supported.html#html)

[Options · KaTeX](https://katex.org/docs/options.html)

[KaTeX/options.md at master · KaTeX/KaTeX (github.com)](https://github.com/KaTeX/KaTeX/blob/master/docs/options.md)

