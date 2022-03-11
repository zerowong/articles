> 原文: <https://tailwindcss.com/docs/upgrade-guide>

## 更新依赖

更新 Tailwind，以及 PostCSS 和 autoprefixer。

```bash
npm install -D tailwindcss@latest postcss@latest autoprefixer@latest
```

注意，Tailwind CSS v3.0 需要 PostCSS 8，而不再支持 PostCSS 7。如果你不能升级到 PostCSS 8，我们建议使用[Tailwind CLI](https://tailwindcss.com/docs/installation)，而不是将 Tailwind 作为 PostCSS 插件安装。

如果你在你的自定义 CSS 中使用嵌套(与 PostCSS 嵌套插件相结合)，你还应该在你的 PostCSS 配置中[configure the `tailwindcss/nesting` plugin](https://tailwindcss.com/docs/using-with-preprocessors#nesting)，以确保与 Tailwind CSS v3.0 兼容。

### 官方插件

我们所有的第一方插件都已更新并且与 v3.0 兼容。

如果你正在使用我们的任何插件，请确保同时将它们全部更新到最新版本，以避免错误。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/1.png)

### Play CDN

对于 Tailwind CSS v3.0，我们过去提供的基于 CSS 的 CDN 构建已被新的[Play CDN](https://tailwindcss.com/docs/installation/play-cdn)所取代，它让你在浏览器中获得新引擎的全部功能，而无需构建步骤。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/2.png)

Play CDN 是为开发而设计的--在生产中，编译你自己的静态 CSS 构建是一个更好的选择。

---

## 迁移到 JIT 引擎

我们在三月份宣布的新的[Just-in-Time engine](https://tailwindcss.com/blog/just-in-time-the-next-generation-of-tailwind-css)已经取代了经典引擎。

新引擎按需生成你的项目所需的样式，并可能需要对你的项目做一些小的改动，这取决于你如何配置 Tailwind。

如果你在 Tailwind CSS v2.x 中已经选择了`mode: 'jit'`，你可以在 v3.0 中安全地从你的配置中删除它。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/3.png)

### 配置源

由于 Tailwind 不再使用 PurgeCSS，我们已经将`purge`选项更名为`content`，以更好地反映它的用途。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/4.png)

由于我们不再使用 PurgeCSS 了，一些 purge 选项已经改变。关于这些选项的更多信息，请参阅新的[content configuration](https://tailwindcss.com/docs/content-configuration)文档。

### 移除黑暗模式配置

黑暗模式功能现在默认以`media`策略启用，所以你可以从你的`tailwind.config.js`文件中完全删除这个选项，除非你使用`class`策略。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/5.png)

如果这个选项目前被设置为`false`，你也可以安全地删除它:

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/6.png)

### 移除 variant 配置

在 Tailwind CSS v3.0 中，每一个 variant 都默认启用，所以你可以从`tailwind.config.js`文件中删除`variant`部分。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/7.png)

### 用 @layer 代替 @variants

由于现在所有的 variant 都是默认启用的，你不再需要使用`@variants`或`@responsive`指令来明确启用这些自定义 CSS。

相反，使用`@layer`指令将任何自定义 CSS 添加到适当的 "layer":

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/8.png)

任何添加到 Tailwind 某一 layer 的自定义 CSS 将自动支持 variant。

更多信息请参见[adding custom styles using CSS and @layer](https://tailwindcss.com/docs/adding-custom-styles#using-css-and-layer)。

### 自动 transform 和 filter

在 Tailwind CSS v3.0 中，像 `scale-50`和 `brightness-75`这样的 transform 和 filter utilities 将自动生效，而不需要添加 `transform`、`filter`或 `backdrop-filter`类。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/9.png)

虽然留下它们并没有什么坏处，但它们可以被安全地删除。

---

## 调色板变化

Tailwind CSS v3.0 现在默认包括扩展调色板中的每一种颜色，包括以前禁用的颜色，如 cyan、rose、fuchsia 和 lime，以及所有五种灰色及其变体。

### 删除了颜色的别名

在 v2.0 版中，一些默认的颜色实际上是扩展颜色的别名。

| v2 Default | v2 Extended |
| ---------- | ----------- |
| `green`    | `emerald`   |
| `yellow`   | `amber`     |
| `purple`   | `violet`    |

在 v3.0 中，这些颜色默认使用其扩展名称，所以以前的`bg-green-500`现在是`bg-emerald-500`，`bg-green-500`现在指的是扩展调色板中的绿色。

如果你在你的项目中使用这些颜色，最简单的升级方法是在你的`tailwind.config.js`文件中把它们改回以前的名字。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/10.png)

如果你已经在使用自定义调色板，这一变化对你完全没有影响。

### 重新命名的灰度

作为默认启用所有扩展颜色的一部分，我们给不同的灰度等级起了更短的单字名称，以使它们更实用，使它们在同一时间共存时不那么尴尬。

| v2 Default | v2 Extended | v3 Unified |
| ---------- | ----------- | ---------- |
| N/A        | `blueGray`  | `slate`    |
| `gray`     | `coolGray`  | `gray`     |
| N/A        | `gray`      | `zinc`     |
| N/A        | `trueGray`  | `neutral`  |
| N/A        | `warmGray`  | `stone`    |

如果你引用了任何一个扩展的灰色，你应该更新你的引用。以新的名称为例:

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/11.png)

如果你没有引用任何扩展调色板中的灰色，这个变化对你完全没有影响。

---

## 类名称的变化

Tailwind CSS v3.0 中的一些类名已经改变，以避免命名冲突，改善开发者的体验，或使其有可能支持新的功能。

在可能的情况下，我们也保留了旧的名称，所以这些变化很多都是非破坏性的，但我们鼓励你更新到新的类名称。

### overflow-clip/ellipsis

那些可恶的浏览器开发者添加了一个真正的`overflow: clip`属性，所以现在用`overflow-clip`来表示`text-overflow: clip`是一个非常糟糕的主意。

我们把`overflow-clip`重命名为`text-clip`，并把`overflow-ellipsis`重命名为`text-ellipsis`，以避免命名上的冲突。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/12.png)

这可能没什么影响，因为`text-clip`的使用情况非常少，它只是因为需要而被包括在内。

### flex-grow/shrink

我们添加了`grow-*`和`shrink-*`作为`flex-grow-*`和`flex-shrink-*`的别名。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/13.png)

旧的类名将永远有效，但我们鼓励你更新到新的类名。

### outline-black/white

由于浏览器在渲染轮廓时终于开始尊重边界半径，我们为`outline-style`、`outline-color`、`outline-width`和`outline-offset`属性添加了单独的 utilities。

这意味着`outline-white`和`outline-black`现在只设置轮廓的颜色，而在 v2 中它们设置颜色、宽度、样式和偏移。

如果你在你的项目中使用`outline-white`或`outline-black`，你可以通过在你的项目中添加以下自定义 CSS 来恢复原来的样式。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/14.png)

另外，你可以在你的 CSS 中用以下类更新它们。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/15.png)

### decoration-clone/slice

我们添加了`box-decoration-clone`和`box-decoration-slice`作为`decoration-clone`和`decoration-slice`的别名，以避免与所有使用`decoration-`命名空间的新`text-decoration`utilities 混淆。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/16.png)

旧的类名将永远有效，但我们鼓励你更新到新的类名。

---

## 其他小变化

Tailwind CSS v3.0 有一些其他的小改动，这些改动不太可能影响到很多人，但需要在这里做个提示。

### 分隔符不能是破折号

在 V3.0 中，破折号(`-`)字符不能作为自定义分隔符使用，因为它在引擎中引入了一个解析歧义。

你必须改用其他字符，如`_`。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/17.png)

### 前缀不能是一个函数

在 Tailwind CSS v3.0 之前，可以将你的类前缀定义为一个函数。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/18.png)

这在新引擎中是不可能的，我们不得不取消对这一功能的支持。

取而代之的是，使用一个静态前缀，它对 Tailwind 生成的每个类都是一样的。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/19.png)

### file 修饰符的顺序颠倒了

自 v3.0.0-alpha.2 版本引入`file`修饰符后的超级小变化--如果你将它与其他修饰符如`hover`或`focus`结合，你需要颠倒修饰符顺序。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/20.png)

在[ordering stacked modifiers](https://tailwindcss.com/docs/hover-focus-and-other-states#ordering-stacked-modifiers)文档中了解更多。

### fill 和 stroke 使用调色板

现在，`fill-{color}`和`stroke-{color}`utilities 默认使用你的`theme.colors`。如果你没有定制你的调色板，这不是一个破坏性的变化，但如果你有而且没有在你自己的调色板中加入`current`，`fill-current`和`stroke-current`类就可能无法工作。

在你的自定义调色板中添加`current`来解决这个问题:

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/21.png)

### 移除负值

像`-mx-4`这样的 utilities 中的负数前缀现在是 Tailwind 的第一类功能，而不是由你的主题驱动的东西，所以你可以在任何支持负数的 utilities 前面添加`-`，它会正常工作。

负值已经从默认主题中移除，所以如果你用`theme()`来引用它们，则在编译 CSS 时会报错。

使用`calc()`函数来更新任何受影响的代码。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/22.png)

### 基础 layer 必须存在

在 Tailwind CSS v3.0 中，`@tailwind base`指令必须存在，以便像 transform、filter 和 shadow 这样的 utilities 能够按预期工作。

如果你之前通过不包括这个指令来禁用 Tailwind 的基础样式，你应该把它加回来，并在你的`corePlugins`配置中禁用`preflight`。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/23.png)

这将禁用 Tailwind 的全局基本样式，而不影响那些依靠添加自己的基本样式来正常工作的 utilities。

### Screens layer 已被重新命名

`@tailwind screens`layer 已更名为`@tailwind variant`。

![](https://cdn.apasser.xyz/articles/tailwindcss%203.x%20%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97/24.png)
