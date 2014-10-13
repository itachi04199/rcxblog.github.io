---
layout: post
title: Bootstrap基础学习1
categories: Bootstrap学习
tags: 笔记 前端
---

#### 布局容器

Bootstrap 需要为页面内容和栅格系统包裹一个 `.container` 容器。我们提供了两个这样的类。注意，由于 padding 等属性的原因，这两种容器类不能互相嵌套。

```html
.container 类用于固定宽度并支持响应式布局的容器。
<div class="container">
  ...
</div>

.container-fluid 类用于 100% 宽度，占据全部视口（viewport）的容器。
<div class="container-fluid">
  ...
</div>
```

#### 栅格系统

Bootstrap 提供了一套响应式、移动设备优先的流式栅格系统，随着屏幕或视口（viewport）尺寸的增加，系统会自动分为最多12列。

下面看下Bootstrap的工作原理：

- “行（row）”必须包含在 `.container` （固定宽度）或 `.container-fluid` （100% 宽度）中，以便为其赋予合适的排列（aligment）和内补（padding）。
- 通过“行（row）”在水平方向创建一组“列（column）”。
- 你的内容应当放置于“列（column）”内，并且，只有“列（column）”可以作为行（row）”的直接子元素。
- 类似 `.row` 和 `.col-xs-4` 这种预定义的类，可以用来快速创建栅格布局。
- 如果一“行（row）”中包含了的“列（column）”大于 12，多余的“列（column）”所在的元素将被作为一个整体另起一行排列。

一般“列”（column）会有几种前缀，只是列的具体宽度会不同，详细见[bootstrap列参数](http://v3.bootcss.com/css/#grid-options)：

- `.col-xs-`
- `.col-sm-`
- `.col-md-`
- `.col-lg-`

如果在一个 `.row` 内包含的列（column）大于12个，包含多余列（column）的元素将作为一个整体单元被另起一行排列。

```html
<div class="row">
  <div class="col-xs-9">.col-xs-9</div>
  <div class="col-xs-4">.col-xs-4<br>Since 9 + 4 = 13 &gt; 12, this 4-column-wide div gets wrapped onto a new line as one contiguous unit.</div>
  <div class="col-xs-6">.col-xs-6<br>Subsequent columns continue along the new line.</div>
</div>
```

使用 `.col-md-offset-*` 类可以将列向右侧偏移。这些类实际是通过使用 * 选择器为当前元素增加了左侧的边距（margin）。例如，`.col-md-offset-4` 类将 `.col-md-4` 元素向右侧偏移了4个列（column）的宽度。

```html
<div class="row">
  <div class="col-md-4">.col-md-4</div>
  <div class="col-md-4 col-md-offset-4">.col-md-4 .col-md-offset-4</div>
</div>
<div class="row">
  <div class="col-md-3 col-md-offset-3">.col-md-3 .col-md-offset-3</div>
  <div class="col-md-3 col-md-offset-3">.col-md-3 .col-md-offset-3</div>
</div>
<div class="row">
  <div class="col-md-6 col-md-offset-3">.col-md-6 .col-md-offset-3</div>
</div>
```

为了使用内置的栅格系统将内容再次嵌套，可以通过添加一个新的 `.row` 元素和一系列 `.col-sm-*` 元素到已经存在的 `.col-sm-*` 元素内。被嵌套的行（row）所包含的列（column）的个数不能超过12（其实，没有要求你必须占满12列）。

```html
<div class="row">
  <div class="col-sm-9">
    Level 1: .col-sm-9
    <div class="row">
      <div class="col-xs-8 col-sm-6">
        Level 2: .col-xs-8 .col-sm-6
      </div>
      <div class="col-xs-4 col-sm-6">
        Level 2: .col-xs-4 .col-sm-6
      </div>
    </div>
  </div>
</div>
```

#### 标题

HTML 中的所有标题标签，`<h1>` 到 `<h6>` 均可使用。另外，还提供了 .h1 到 .h6 类，为的是给内联（inline）属性的文本赋予标题的样式。

```html
<h1>h1. Bootstrap heading</h1> 半黑体 36px
<h2>h2. Bootstrap heading</h2> 半黑体 30px
<h3>h3. Bootstrap heading</h3> 半黑体 24px
<h4>h4. Bootstrap heading</h4> 半黑体 18px
<h5>h5. Bootstrap heading</h5> 半黑体 14px
<h6>h6. Bootstrap heading</h6> 半黑体 12px
```

在标题内还可以包含 `<small>` 标签或赋予 `.small` 类的元素，可以用来标记副标题。

#### 页面主体

Bootstrap 将全局 font-size 设置为 14px，line-height 设置为 1.428。这些属性直接赋予 `<body>` 元素和所有段落元素。另外，`<p>` （段落）元素还被设置了等于 1/2 行高（即 10px）的底部外边距（margin）。

通过添加 .lead 类可以让段落突出显示。

```html
<p>...</p>
<p class="lead">...</p>
```

使用`<mark>`可以使文本进行高亮。

```html
You can use the mark tag to <mark>highlight</mark> text.
```

对于被删除的文本使用 `<del>`标签。对于没用的文本使用 `<s>` 标签。

```html
<del>highlight</del> text.
<s>This line of text is meant to </s>
```

带下划线的文本,可以使用`<ins>`或者`<u>`。

```html
<ins>This line of text.</ins>
<u>This line of text.</u>
```

#### 对齐

通过文本对齐类，可以简单方便的将文字重新对齐。

```html
<p class="text-left">Left aligned text.</p>
<p class="text-center">Center aligned text.</p>
<p class="text-right">Right aligned text.</p>
<p class="text-justify">Justified text.</p>
<p class="text-nowrap">No wrap text.</p>
```

#### 改变大小写

通过这几个类可以改变文本的大小写。

```html
<p class="text-lowercase">Lowercased text.</p>
<p class="text-uppercase">Uppercased text.</p>
<p class="text-capitalize">Capitalized text.</p>
```

【参考资料】

1. [Bootstrap中文网](http://www.bootcss.com/)

---EOF---

