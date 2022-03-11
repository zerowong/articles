Yarn workspaces旨在使[monorepos](/advanced/lexicon#monorepository)变得简单，主要用途之一就是以一种更明确的方式解决`yarn link`的一些问题。简而言之，它们允许多个项目在同一个资源库中并存，并且相互引用--对其中一个的源代码的任何修改都会立即应用于其他项目。

首先，解释一些概念：在workspace上下文下，一个 _project_ 构成workspace的整个目录树(通常是资源库本身)。一个 _workspace_ 是一个本地包，由同一项目中你自己的源代码组成。最后，_worktree_ 是所包含的`workspace`的列表。一个项目包含一个或多个worktree，这些worktree本身可以包含任意数量的`workspace`。任何项目都至少包含一个根workspace。

## 如何声明一个 worktree？

worktree通过传统的`package.json`文件定义。它们的特殊之处在于它们有以下属性:

- 它们必须声明一个`workspaces`字段，这个字段应该是一个glob模式的数组，应该用来定位组成worktree的workspace。例如，如果你想让`packages`文件夹中的所有文件夹成为workspace，只需在这个数组中添加`packages/*`。

- 它们必须以某种方式与项目级的`package.json`文件相连。这在典型的workspace设置中并不重要，因为在项目级的`package.json`中通常只定义了一个worktree，但如果你试图设置嵌套的workspace，那么你必须确保嵌套的worktree被定义为其父worktree的有效workspace(否则Yarn不会找到其正确的父文件夹)。

请注意，由于worktree是用普通的`package.json`文件定义的，它们本身也是有效的workspace。如果它们被命名，其他workspace将能够正确地交叉引用它们。

> **注意**
> 
> worktree曾经被要求是私有的(即在其package.json中列出`"private": true`)。这一要求在2.0版本中被删除，以帮助独立项目逐步采用workspace(例如，将其文档网站列为独立的workspace)。

## workspace是什么意思？

workspace有两个重要的属性。

- 只有workspace所依赖的dependencies才能被访问。换句话说，我们严格执行你的workspace的依赖关系。这样做使我们能够干净地将项目相互分离，因为你不需要将所有的依赖关系合并到一个巨大的不可维护的列表中。我们仍然提供工具来同时管理来自多个workspace的依赖，但它们需要显式地使用，并提供更好的集成(例如`yarn add`可以根据其他workspace的使用情况为你的新依赖提供建议，但你可以覆盖它们)。

- 如果包管理器要解析一个workspace可以满足的range，可以的话，它将优先选择workspace的解析而不是远程解析。这就是monorepo方法的核心：项目的包不是使用来自registry的远程包，而是相互连接，并使用存储在你的仓库中的代码。

## workspace range(`workspace:`)

虽然Yarn会在workspace匹配时自动挑选，但有些时候你绝对不想冒险使用来自远程registry的包，即使版本不匹配(例如，如果你的项目实际上并不打算发布，而你只是想使用workspace来更好地划分你的代码)。

对于这些用例，Yarn从v2开始现在支持一个新的解析协议:`workspace:`。当使用这个协议时，Yarn将不会解析到除本地workspace以外的任何其他地方。这个range协议有两种用法。

- 如果是semver range，它将选择与指定版本匹配的workspace。
- 如果是一个项目相关的路径，它将选择与该路径相匹配的workspace **(实验性)**。

请注意，第二种用法是实验性的，我们建议目前不要使用它，因为一些细节可能会在未来发生变化。我们目前的建议是使用`workspace:*`，它几乎总是做你期望的事情。

## 发布workspace

当一个workspace被打包到一个归档文件中时(无论是通过`yarn pack`还是像`yarn npm publish`这样的发布命令)，我们动态地将任何`workspace:`的依赖关系替换为:

- 目标workspace的相应版本(如果你使用`*`，`^`，`~`，或项目相关的路径)
- 相关的semver range(对于任何其他range类型)。

如果我们有以下workspace，其当前版本是`1.5.0`:

```json
{
  "dependencies": {
    "star": "workspace:*",
    "caret": "workspace:^",
    "tilde": "workspace:~",
    "range": "workspace:^1.2.3",
    "path": "workspace:path/to/baz"
  }
}
```

会被转换为:

```json
{
  "dependencies": {
    "star": "1.5.0",
    "caret": "^1.5.0",
    "tilde": "~1.5.0",
    "range": "^1.2.3",
    "path": "1.5.0"
  }
}
```

这个功能允许你不必依赖你的本地workspace以外的东西，同时仍然能够将包发布到远程registry，而不必运行中间的发布步骤--你的用户将能够像使用其他任何包一样使用你发布的workspace，并仍然受益于semver提供的保证。
