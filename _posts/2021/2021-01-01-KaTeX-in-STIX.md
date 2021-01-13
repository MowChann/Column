---
layout: post
title: KaTeX 的 STIX 字体解决方案
subtitle: 可能是博客上显示数学公式最完美的方案
date: 2021-01-01 20:00:00
categories: [前端]
author: MowChan
latex: true
tags: [前端, 数学公式, LaTeX, KaTeX]
incomplete: true
---

目前来说，在网页上显示数学公式最常用的方案是引入 JavaScript 对 LaTeX 代码同步渲染，使用最多的库是 MathJax 和 KaTeX。

KaTeX 与 MathJax 相比最大的优点就是整个库体积小巧，同步加载，公式绘制的速度非常快，因此适用于常见数学公式的需求场景。然而 KaTeX 并不能像 MathJax 那样可以指定字体，而是使用默认的Latin Modern Math，在分辨率较低的使用场景下并不容易识别，不够美观。所以，本文试图用识别度更高的 STIX 字体替换 KaTeX 默认字体，并做出一定的优化。

### 字体文件替换

在`katex.min.css`中，引用具体的字体文件，定义成字体族名，再对 CSS 绘制的公式各个部分应用不同的字体样式，这就是 KaTeX 核心的渲染功能。

```css
@font-face {
    font-family: KaTeX_AMS;
    src: url(fonts/KaTeX_AMS-Regular.woff2) format("woff2"),url(fonts/KaTeX_AMS-Regular.woff) format("woff"),url(fonts/KaTeX_AMS-Regular.ttf) format("truetype");
    font-weight: 400;
    font-style: normal
}
```

因此，在最初的尝试，我将字体定义的文件路径修改为 STIX 文件的绝对路径。然而 CSS 中引用绝对路径的文件存在跨域的问题，触发服务器安全机制，无法正常加载字体文件。最终我决定，直接替换原有的字体文件。

查看 KaTeX 默认字体的文件列表（以woff为例，除此之外还有ttf、woff2格式的同名文件）

```
KaTeX_AMS-Regular.woff
KaTeX_Caligraphic-Bold.woff
KaTeX_Caligraphic-Regular.woff
KaTeX_Fraktur-Bold.woff
KaTeX_Fraktur-Regular.woff
KaTeX_Main-Bold.woff
KaTeX_Main-BoldItalic.woff
KaTeX_Main-Italic.woff
KaTeX_Main-Regular.woff
KaTeX_Math-BoldItalic.woff
KaTeX_Math-Italic.woff
KaTeX_SansSerif-Bold.woff
KaTeX_SansSerif-Italic.woff
KaTeX_SansSerif-Regular.woff
KaTeX_Script-Regular.woff
KaTeX_Size1-Regular.woff
KaTeX_Size2-Regular.woff
KaTeX_Size3-Regular.woff
KaTeX_Size4-Regular.woff
KaTeX_Typewriter-Regular.woff
```

MathJax 提供了很多数学字体，有 Asana Math，Gyre Pagella，Latin Modern，Neo Euler，STIX Web等等。有 eot、otf、woff格式，我们以STIX 字体的woff格式为例，再将woff转换为 KaTeX 对应需要的 woff2、ttf格式

```
STIXMathJax_Alphabets-Bold.woff
STIXMathJax_Alphabets-BoldItalic.woff
STIXMathJax_Alphabets-Italic.woff
STIXMathJax_Alphabets-Regular.woff
STIXMathJax_Arrows-Bold.woff
STIXMathJax_Arrows-Regular.woff
STIXMathJax_DoubleStruck-Bold.woff
STIXMathJax_DoubleStruck-BoldItalic.woff
STIXMathJax_DoubleStruck-Italic.woff
STIXMathJax_DoubleStruck-Regular.woff
STIXMathJax_Fraktur-Bold.woff
STIXMathJax_Fraktur-Regular.woff
STIXMathJax_Latin-Bold.woff
STIXMathJax_Latin-BoldItalic.woff
STIXMathJax_Latin-Italic.woff
STIXMathJax_Latin-Regular.woff
STIXMathJax_Main-Bold.woff
STIXMathJax_Main-BoldItalic.woff
STIXMathJax_Main-Italic.woff
STIXMathJax_Main-Regular.woff
STIXMathJax_Marks-Bold.woff
STIXMathJax_Marks-BoldItalic.woff
STIXMathJax_Marks-Italic.woff
STIXMathJax_Marks-Regular.woff
STIXMathJax_Misc-Bold.woff
STIXMathJax_Misc-BoldItalic.woff
STIXMathJax_Misc-Italic.woff
STIXMathJax_Misc-Regular.woff
STIXMathJax_Monospace-Regular.woff
STIXMathJax_Normal-Bold.woff
STIXMathJax_Normal-BoldItalic.woff
STIXMathJax_Normal-Italic.woff
STIXMathJax_Operators-Bold.woff
STIXMathJax_Operators-Regular.woff
STIXMathJax_SansSerif-Bold.woff
STIXMathJax_SansSerif-BoldItalic.woff
STIXMathJax_SansSerif-Italic.woff
STIXMathJax_SansSerif-Regular.woff
STIXMathJax_Script-BoldItalic.woff
STIXMathJax_Script-Italic.woff
STIXMathJax_Script-Regular.woff
STIXMathJax_Shapes-Bold.woff
STIXMathJax_Shapes-BoldItalic.woff
STIXMathJax_Shapes-Regular.woff
STIXMathJax_Size1-Regular.woff
STIXMathJax_Size2-Regular.woff
STIXMathJax_Size3-Regular.woff
STIXMathJax_Size4-Regular.woff
STIXMathJax_Size5-Regular.woff
STIXMathJax_Symbols-Bold.woff
STIXMathJax_Symbols-Regular.woff
STIXMathJax_Variants-Bold.woff
STIXMathJax_Variants-BoldItalic.woff
STIXMathJax_Variants-Italic.woff
STIXMathJax_Variants-Regular.woff
```

可以看到 MathJax 的 STIX 字体分割地非常详细，和 KaTeX 的字体差别较大，因此绝不能完整地替换。使用 FontCreator 软件对两种字体的字模分析对比后，我发现`STIXMathJax_Main`的字模足以覆盖`KaTeX_Main`、`KaTeX_Math`两个文件，而且字符的内部名称也一致，替换后可能的问题最少。

因此，替换到原有的字体文件后，再将 CSS 中 `KaTeX_Main`、`KaTeX_Math`指向同一文件。

```css
@font-face {
    font-family: KaTeX_Main;
    src: url(fonts/KaTeX_Main-Regular.woff2) format("woff2"),url(fonts/KaTeX_Main-Regular.woff) format("woff"),url(fonts/KaTeX_Main-Regular.ttf) format("truetype");
    font-weight: 400;
    font-style: normal
}
@font-face {
    font-family: KaTeX_Math;
    src: url(fonts/KaTeX_Main-Italic.woff2) format("woff2"),url(fonts/KaTeX_Main-Italic.woff) format("woff"),url(fonts/KaTeX_Main-Italic.ttf) format("truetype");
    font-weight: 400;
    font-style: italic
}
```

#### 样式调整

默认的 KaTeX 与 MathJax 绘制公式的文字都略大于标准文字大小，如 KaTeX 的默认大小为 `1.21em`，在与正文混排时看起来会比较奇怪。根据 KaTeX 的官方文档，在`katex.min.css`中将`.katex`字体大小修改为`1.1em`是最合适的。

```css
.katex {
    font: normal 1.1em KaTeX_Main,Times New Roman,serif;
    line-height: 1.2;
    text-indent: 0;
    text-rendering: auto;
    border-color: currentColor
}
```

### KaTeX 常用配置

#### 分隔符

分隔符用于限定 KaTeX 的代码范围，可以手动指定多种不同的分隔符。对于公式排版来说，有两种文字环绕方式，内联（inline）和显示（display），前者表示公式与文字混排，和其它正文在同一行中；后者表示公式单独列出一行，内容居中。

在 Markdown 中，`$`是最常用的公式分隔符，`$`表示 inline，`$$`表示 display，如下配置。

```javascript
delimiters: [
    {left: "$$", right: "$$", display: true},
    {left: "$", right: "$", display: false}
]
```

#### 宏

宏可以用于设置别名（alias）或缩短指令长度。

例如，宏可以解决大于等于号或小于等于号样式不一致的问题。对于多大数出版物，它们的符号样式等于号是斜式（slanted）的（⩽、⩾），而常用的指令或是 MathType、微软公式这些常用的公式软件，符号样式中等号是水平的（≤、≥）。这两者并没有什么区别，只是斜式在正式的出版物中更为常见，也更容易识别。但这两种样式在 Unicode 中是不同的编码字符，LaTeX 指令也不同，水平式的用`\ge`或`\geq`表示大于等于，斜式用`\geqslant`表示。因此，我们可以通过指定宏，将`\ge`替换为`\geqslant`，这样符号就统一规范成斜式。

```javascript
macros: {
    "\\ge": "\\geqslant",
    "\\le": "\\leqslant",
    "\\geq": "\\geqslant",
    "\\leq": "\\leqslant"
}
```

### 示例效果

点$P(x,y)$到原点距离为$\vert OP\vert=\sqrt{x^2+y^2}$

```latex
点$P(x,y)$到原点距离为$\vert OP\vert =\sqrt{x^2+y^2}$
```

对于一般的圆$x^2+y^2+Dx+Ey+F=0$，其圆心坐标为$\left(-\cfrac{D}{2},-\cfrac{E}{2}\right)$，半径为$\cfrac{\sqrt{D^2+E^2-4F}}{2}$

```latex
对于一般的圆$x^2+y^2+Dx+Ey+F=0$，其圆心坐标为$\left(-\cfrac{D}{2},-\cfrac{E}{2}\right)$，半径为$\cfrac{\sqrt{D^2+E^2-4F}}{2}$
```

$$
e^{\pi i} + 1 = 0
$$

```latex
e^{\pi i} + 1 = 0
```
$$
\begin{aligned} \sin 2\theta & = 2\sin \theta \cos \theta \\ & = \cfrac{2 \tan \theta}{1+\tan^2 \theta} \end{aligned}
$$

```latex
\begin{aligned} \sin 2\theta & = 2\sin \theta \cos \theta \\ & = \cfrac{2 \tan \theta}{1+\tan^2 \theta} \end{aligned}
```
注意：KaTeX 不支持`\begin{align}`，因此对齐时应当用`\begin{aligned}`

$$
f(x) = 
\left\{ \begin{array}{ll}
\lambda e^{-\lambda x}, & x>0 \\
0, & 其它
\end{array} \right.
(\lambda > 0)
$$

```latex
f(x) = \left\{ \begin{array}{ll} \lambda e^{-\lambda x}, & x>0 \\ 0, & 其它 \end{array} \right. (\lambda > 0)
```
$$
A_{m,n} = \begin{pmatrix} a_{1,1} & a_{1,2} & \cdots & a_{1,n} \\ a_{2,1} & a_{2,2} & \cdots & a_{2,n} \\ \vdots & \vdots & \ddots & \vdots \\ a_{m,1} & a_{m,2} & \cdots & a_{m,n} \end{pmatrix}
$$

```latex
A_{m,n} = \begin{pmatrix} a_{1,1} & a_{1,2} & \cdots & a_{1,n} \\ a_{2,1} & a_{2,2} & \cdots & a_{2,n} \\ \vdots & \vdots & \ddots & \vdots \\ a_{m,1} & a_{m,2} & \cdots & a_{m,n} \end{pmatrix}
```
$$
\oint_{\Gamma}Pdx+Qdy+Rdz = \iint_{\Sigma}\begin{vmatrix} \cos \alpha & \cos \beta & \cos \gamma \\ \cfrac{\partial}{\partial x} & \cfrac{\partial}{\partial y} & \cfrac{\partial}{\partial z} \\ P & Q & R \end{vmatrix}dS
$$

```latex
\oint_{\Gamma}Pdx+Qdy+Rdz = \iint_{\Sigma}\begin{vmatrix} \cos \alpha & \cos \beta & \cos \gamma \\ \cfrac{\partial}{\partial x} & \cfrac{\partial}{\partial y} & \cfrac{\partial}{\partial z} \\ P & Q & R \end{vmatrix}dS
```

### 参考来源

<https://katex.org/docs/font.html>

<https://katex.org/docs/supported.html>

<https://math.stackexchange.com/questions/247719/meaning-of-geqslant-leqslant-eqslantgtr-eqslantless/247745>

