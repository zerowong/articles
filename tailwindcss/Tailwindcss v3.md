# Tailwind CSS v3.0

Tailwind CSS v3.0 来了--带来了令人难以置信的性能提升，巨大的工作流程改进，以及数量惊人的新功能。

如需了解一些最酷的新功能，请查看我们 YouTube 频道上的["Tailwind CSS v3.0 有什么新功能"](https://www.youtube.com/watch?v=mSC6GwizOag)视频。

Tailwind CSS v3.0 一定是我们有史以来最激动人心的版本，包括以下改进:

- [JIT，无时不在](#just-in-time-all-the-time) — 大幅减少构建时间、可堆叠的 `variants`、支持任意值、更好的浏览器性能，等等。
- [每种颜色都是开箱即用](#every-color-out-of-the-box) — 包括所有扩展的调色板颜色，如青色、玫瑰色、紫红色和青柠色，以及 50 种灰色。
- [彩色阴影](#colored-box-shadows) — 以获得有趣的发光和反射效果，以及彩色背景上更自然的阴影。
- [滚动捕捉 API](#scroll-snap-api) — 一套全面的、可组合的、用于纯 CSS 滚动捕捉的实用工具。
- [多列布局](#multi-column-layout) — 由此，你终于可以建立你一直梦想的在线报纸页了。
- [原生的表单控件样式设计](#native-form-control-styling) — 使复选框、单选按钮和文件输入符合你的风格，而不需要重新发明轮子。
- [打印修饰符](#print-modifier) — 就在你的 HTML 中控制你的网站在别人打印时的样子。
- [现代长宽比 API](#modern-aspect-ratio-api) — 不再需要更多的奇技淫巧，除非你需要支持 Safari 14，但仍然有用。
- [花式下划线样式](#fancy-underline-styles) — 这是使你的边缘项目最终起飞的缺失部分。
- [RTL 和 LTR 修饰符](#rtl-and-ltr-modifiers) — 以便在搭建多方向的网站时进行完全地控制。
- [人像和景观修饰符](#portrait-and-landscape-modifiers) — 说实话，这只是因为它们真的很容易添加。
- [任意的属性](#arbitrary-properties) — 现在，Tailwind 支持我们甚至从未听说过的 CSS 属性。
- [发挥 CDN 的作用](#play-cdn) — 新的 JIT 引擎被集成到 CDN 脚本中，直接在浏览器中运行。
- **大量的其它实用工具** — 包括对 touch-action、will-change、flex-basis、text-indent、scroll-behavior 等的支持。

再加上一个漂亮的、全新的[文档](https://tailwindcss.com)，每一页都有改进的内容和例子。

从 npm 获取最新版本，即刻开始使用 Tailwind CSS v3.0。

```shell
npm install -D tailwindcss@latest postcss autoprefixer
```

...或者前往[Tailwind Play](https://play.tailwindcss.com)，直接在浏览器中尝试最新的功能。

Tailwind CSS v3.0 是框架的一个新的主要版本，有一些小的 breaking changes，但我们已经非常努力地使升级过程尽可能顺利，对于大多数项目，你应该能够安装 v3.0 而不做任何修改。

例如，[Tailwind UI](https://tailwindui.com)可能是地球上最大的 Tailwind 项目，每一个模板都与 V2 和 V3 完全兼容，不需要任何改变。

关于迁移到 v3.0 的更多细节和步骤说明，请查看[升级指南](https://tailwindcss.com/docs/upgrade-guide)。

---

## JIT，无时不在

早在三月，我们就推出了全新的[JIT 引擎](https://tailwindcss.com/blog/just-in-time-the-next-generation-of-tailwind-css)，它带来了巨大的性能提升，解锁了令人兴奋的新功能，如[任意值](https://tailwindcss.com/docs/adding-custom-styles#using-arbitrary-values)，并使复杂的 `variant` 配置成为过去式。

在 Tailwind CSS v3.0 中，新引擎已经稳定，并取代了经典引擎，所以每个 Tailwind 项目都可以从这些改进中受益，开箱即用。

---

## 每种颜色都是开箱即用

在新引擎之前，我们在开发中总是不得不小心翼翼地处理 CSS 文件的大小，而我们必须做出的最大权衡之一就是谨慎地限制调色板。

在 v3.0 中，扩展调色板中的每一种颜色都是默认启用的，包括青色、青色、天空、紫红色、玫瑰色和 50 种灰色。

查看[调色板](https://tailwindcss.com/docs/customizing-colors)以了解更多。

---

## 彩色阴影

多年来，人们一直要求我们提供彩色阴影，但要以一种可组合的方式支持它，并使之真正有意义，比我预期的要难得多。

在经过大概五次错误的尝试后，我们终于找到了一个我们喜欢的方法，现在 Tailwind CSS v3.0 包含了彩色阴影:

![彩色阴影](https://cdn.apasser.xyz/articles/Tailwindcss%20v3/1.png "彩色阴影")

```html
<button class="bg-cyan-500 **shadow-lg shadow-cyan-500/50** ...">
  Subscribe
</button>
<button class="bg-blue-500 **shadow-lg shadow-blue-500/50** ...">
  Subscribe
</button>
<button class="bg-indigo-500 **shadow-lg shadow-indigo-500/50** ...">
  Subscribe
</button>
```

在[阴影颜色](https://tailwindcss.com/docs/box-shadow-color)文档中了解更多。

---

## 滚动捕捉 API

我们为 CSS Scroll Snap 模块增加了一套全面的工具，让你有能力直接在你的 HTML 中建立非常丰富的滚动抓取体验。

![滚动捕捉 API](https://cdn.apasser.xyz/articles/Tailwindcss%20v3/2.png "滚动捕捉 API")

```html
<div class="**snap-x** ...">
  <div class="snap-center ...">
    <img
      src="https://images.unsplash.com/photo-1604999565976-8913ad2ddb7c?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=320&h=160&q=80"
    />
  </div>
  <div class="snap-center ...">
    <img
      src="https://images.unsplash.com/photo-1540206351-d6465b3ac5c1?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=320&h=160&q=80"
    />
  </div>
  <div class="snap-center ...">
    <img
      src="https://images.unsplash.com/photo-1622890806166-111d7f6c7c97?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=320&h=160&q=80"
    />
  </div>
  <div class="snap-center ...">
    <img
      src="https://images.unsplash.com/photo-1590523277543-a94d2e4eb00b?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=320&h=160&q=80"
    />
  </div>
  <div class="snap-center ...">
    <img
      src="https://images.unsplash.com/photo-1575424909138-46b05e5919ec?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=320&h=160&q=80"
    />
  </div>
  <div class="snap-center ...">
    <img
      src="https://images.unsplash.com/photo-1559333086-b0a56225a93c?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=320&h=160&q=80"
    />
  </div>
</div>
```

从新的[滚动边距](https://tailwindcss.com/docs/scroll-margin)工具开始，来了解整个 API。

---

## 多列布局

我们增加了对[列](https://developer.mozilla.org/en-US/docs/Web/CSS/columns)的支持--报纸布局的那种。这些实际上是非常有用的，而且对于像页脚导航布局这样的事情也很有用。

![多列布局](https://cdn.apasser.xyz/articles/Tailwindcss%20v3/3.png "多列布局")

```html
<div class="**columns-1** **sm:columns-3** ...">
  <p>...</p>
  <!-- ... -->
</div>
```

在[columns](https://tailwindcss.com/docs/columns)文档中了解更多--也可以查看新的[break-after/inside/before](https://tailwindcss.com/docs/break-after)实用工具。

---

## 原生的表单控件样式设计

我们增加了对新的`accent-color`属性的支持，以及对文件输入按钮样式的修饰符，使你比以往任何时候都更容易在原生表单控件上发挥自己的作用。

![原生的表单控件样式设计](https://cdn.apasser.xyz/articles/Tailwindcss%20v3/4.png "原生的表单控件样式设计")

```html
<form>
  <div class="flex items-center space-x-6">
    <div class="shrink-0">
      <img
        class="h-16 w-16 object-cover rounded-full"
        src="https://images.unsplash.com/photo-1580489944761-15a19d654956?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1361&q=80"
        alt="Current profile photo"
      />
    </div>
    <label class="block">
      <span class="sr-only">Choose profile photo</span>
      <input
        type="file"
        class="block w-full text-sm text-slate-500
        **file:mr-4** **file:py-2** **file:px-4**
        **file:rounded-full** **file:border-0**
        **file:text-sm** **file:font-semibold**
        **file:bg-violet-50** **file:text-violet-700**
        **hover:file:bg-violet-100**
      "
      />
    </label>
  </div>
  <label
    class="mt-6 flex items-center justify-center space-x-2 text-sm font-medium text-slate-600"
  >
    <input type="checkbox" class="**accent-violet-500**" checked />
    <span>Yes, send me all your stupid updates</span>
  </label>
</form>
```

在[强调色](https://tailwindcss.com/docs/accent-color)和[文件输入按钮](https://tailwindcss.com/docs/hover-focus-and-other-states#file-input-buttons)文档中了解更多。

---

## 打印修饰符

新的`print`修饰符可以让你决定网站在~~动物们~~人们打印时的样子。

```html
<div>
  <article class="**print:hidden**">
    <h1>My Secret Pizza Recipe</h1>
    <p>This recipe is a secret, and must not be shared with anyone</p>
    <!-- ... -->
  </article>
  <div class="hidden **print:block**">
    Are you seriously trying to print this? It's secret!
  </div>
</div>
```

在[打印样式](https://tailwindcss.com/docs/hover-focus-and-other-states#print-styles)文档中了解更多。

---

## 现代长宽比 API

我们增加了对新的原生`aspect-ratio`属性的支持，它正开始得到浏览器的支持。

```html
<iframe
  class="w-full **aspect-video** ..."
  src="https://www.youtube.com/..."
></iframe>
```

在[纵横比](https://tailwindcss.com/docs/aspect-ratio)文档中了解更多。

---

## 花式下划线样式

现在你可以改变下划线的颜色、厚度等样式。

![花式下划线样式](https://cdn.apasser.xyz/articles/Tailwindcss%20v3/5.png "花式下划线样式")

```html
<p>
  I’m Derek, an astro-engineer based in Tatooine. I like to build X-Wings at
  <a href="#" class="underline **decoration-sky-500 decoration-2**"
    >My Company, Inc</a
  >. Outside of work, I like to
  <a
    href="#"
    class="underline **decoration-pink-500 decoration-dotted decoration-2**"
    >watch pod-racing</a
  >
  and have
  <a
    href="#"
    class="underline **decoration-indigo-500 decoration-wavy decoration-2**"
    >light-saber</a
  >
  fights.
</p>
```

在[文本装饰颜色](https://tailwindcss.com/docs/text-decoration-color)、[文本装饰样式](https://tailwindcss.com/docs/text-decoration-style)、[文本装饰厚度](https://tailwindcss.com/docs/text-decoration-thickness)和[文本下划线偏移](https://tailwindcss.com/docs/text-underline-offset) 文档中了解更多。

---

## RTL 和 LTR 修饰符

我们用新的`rtl`和`ltr`修饰符增加了对多方向布局的实验性支持。

![RTL 和 LTR 修饰符](https://cdn.apasser.xyz/articles/Tailwindcss%20v3/6.png "RTL 和 LTR 修饰符")

```html
<div class="group flex items-center">
  <img class="shrink-0 h-12 w-12 rounded-full" src="..." alt="" />
  >
  <div class="ltr:ml-3 rtl:mr-3">
    <p
      class="text-sm font-medium text-slate-700 group-hover:text-slate-900"
      dark-class="text-sm font-medium text-slate-300 group-hover:text-white"
    >
      ...
    </p>
    <p
      class="text-sm font-medium text-slate-500 group-hover:text-slate-700"
      dark-class="text-sm font-medium text-slate-500 group-hover:text-slate-300"
    >
      ...
    </p>
  </div>
</div>
```

在[RTL 支持](https://tailwindcss.com/docs/hover-focus-and-other-states#rtl-support)文档中了解更多。

---

## 人像和景观修饰符

使用新的`portrait`和`landscape`修饰符，当视口处于特定方向时，有条件地添加样式。

```html
<div>
  <div class="portrait:hidden">
    <!-- ... -->
  </div>
  <div class="landscape:hidden">
    <p>
      This experience is designed to be viewed in landscape. Please rotate your
      device to view the site.
    </p>
  </div>
</div>
```

[此功能的文档](https://tailwindcss.com/docs/hover-focus-and-other-states#viewport-orientation)的内容比本帖的这部分内容还要少。

---

## 任意的属性

这些属性可能是不支持的，但我们已经使添加完全任意的 CSS 成为可能，你可以将其与`hover`、`lg`等修饰符结合起来。

```html
<div class="[mask-type:luminance] hover:[mask-type:alpha]">
  <!-- ... -->
</div>
```

这就是内联样式想成为的样子。在[任意属性](https://tailwindcss.com/docs/adding-custom-styles#arbitrary-properties) 文档中了解更多。

---

## 发挥 CDN 的作用

没有办法为 Tailwind CSS v3.0 做一个合理的基于 CSS 的 CDN 构建，所以我们不得不做一些不同的事情--我们建立了一个 JavaScript 库。

```diff-html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      <title>Example</title>
+     <script src="https://cdn.tailwindcss.com/"></script>
    </head>
    <body>
      <!-- -->
    </body>
  </html>
```

将该脚本标签添加到任何 HTML 文档中，您就可以在浏览器中使用 Tailwind 的每一项功能。它仅用于开发目的，但它确实是一种有趣的方式，可以建立简单的演示，或在一个新的想法上进行探索。

在[Play CDN](https://tailwindcss.com/docs/installation/play-cdn)文档中了解更多。

---

这就是 - Tailwind CSS v3.0! 今天就去新的[文档](https://tailwindcss.com)开始使用它吧。

对于每一个变化的全面清单，请查看 GitHub 上的[changelog](https://github.com/tailwindlabs/tailwindcss/blob/master/CHANGELOG.md)。
