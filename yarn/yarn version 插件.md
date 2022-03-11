原文: https://yarnpkg.com/features/release-workflow

> **实验性**
>
> 这项功能仍在孵化中，我们可能会根据你的反馈意见来改进它。

> **插件**
>
> 要访问这个功能，首先安装 `version` 插件: `yarn plugin import version`。

当使用 monorepos 时，一项艰巨的任务往往是在开始一个新的版本时弄清楚哪些包应该接受新版本。Yarn 提供了一些工具，旨在使这个工作流程更容易，而不需要第三方工具（当然，有可能你会更喜欢不同实现提供的工作流程！）。

## 自动更新的依赖关系

当运行`yarn version`命令来升级一个 workspace 的版本时，其他所有通过基本 semver range（`^x.y.z`, `~x.y.z`, ...）依赖于某一个 workspace 的其它 workspace 的依赖将被自动更新以引用新版本。例如:

```
/packages/common (1.0.0)
/packages/server (depends on common@^1.0.0)
/packages/client (depends on common@^1.0.0)
```

在 2.0 之前，升级 `common` 需要你在那里运行命令，然后进入 `server` 和 `client` 手动升级它们的依赖关系以引用新版本。但现在不一样了! 如果我们只需要在 `common` 中运行 `yarn version 1.1.1 `:

```
/packages/common (1.1.1)
/packages/server (depends on common@^1.1.1)
/packages/client (depends on common@^1.1.1)
```

当然，当 monorepo 中的包总是要作为 monorepo 的一部分来使用时，这并不重要，但当你处理要发布的多个包时，它将很有用。如果你忘了更新你的依赖包的 range，你的用户就有可能下载一个旧版本的 `common`，而这个版本与新版本不兼容。

## 推迟版本

从 2.0 版本开始，`yarn version`命令现在接受一个新的标志：`--deferred`。当设置该标志时，该命令不会立即改变本地 manifest 中的 `verion` 字段，而是在内部记录一个条目，说明当前包需要在下一个发布周期中接受升级。例如:

```bash
yarn version minor --deferred
```

它不会导致 `package.json` 文件的改变! 相反，Yarn 会在 `.yarn/versions` 目录下创建一个文件（在一个分支内会复用），这个文件将记录所要求的升级。

```yaml
releases:
  my-package@1.0.0: minor
```

之后，一旦你准备好了，只要运行`yarn version apply`。Yarn 就会找到它之前保存的所有升级记录，并一次性应用它们（包括像我们看到的那样，处理好相互依赖关系的升级）。

## 签入的延迟记录

我们在上一节中看到，`yarn version patch` 可以将未来的版本存储在一个内部文件夹中(`.yarn/versions`)。但这是为什么呢？它有什么好处？为了回答这个问题，考虑一个通过 monorepo 开发的流行开源项目。这个项目收到许多外部拉动请求，但它们并不立即发布--它们通常是作为批次的一部分被发布。每隔一段时间，主要的维护者就会把所有的变化，转换为新的版本，然后开始部署。

让我们把注意力集中在必须转换为版本的更改中。那要怎么做呢？这并不容易。以 Lerna 为例（最流行的 monorepos 版本管理工具），你有两种解决方案。

- 在固定模式下，你的所有包都有一个单一的版本。因此，它们会被一次性地升级。

- 在独立模式下，你可以为每个来源发生变化的包选择一个版本。

然而，一个关键的问题仍然存在：即使你使用独立模式，你怎么知道哪些包要被升级？而且，同样关键的是，它们应该是 patch 版本？还是 minor 版本？很难知道--大型项目每周可能会收到几十个 PR，跟踪哪些单元需要发布，发布到哪个版本是一个相当困难的任务。

然而，有了 Yarn 的工作流程，这一切就变得非常容易了，由于升级被保存在一个文件中，并且这个文件被神奇地绑定到一个 Git 分支上，所以这只是一个提交发布文件夹的问题--所有预期的发布将成为项目历史的一部分，直到 `yarn version apply` --然后 Yarn 将消耗所有单独的记录，然后合并（所以一个 minor 的 PR 将比一个 patch 的 PR 拥有更高的优先权），并同时应用它们。

你甚至可以为典型的 PR 审查的同时来审查包的升级。这可以将更多的权力下放给你的社区，同时能够确保每个人都遵守规则。

## 确保版本被 bumped（CI）

然而，提交延迟发布的一个问题是：确保你收到的 PR 包含正确的包发布定义。例如，你应该能够相信定义中包含了每个修改的工作区的发布策略（patch, minor, major, ......）。

为了以自动化的方式解决这个问题，出现了 `yarn version check` 命令。当运行时，该命令将找出哪些包发生了变化，以及它们是否被列在发布定义文件中。如果没有则会报错。如果你把它集成到 CI 系统中，比如 GitHub Actions，PR 作者将被要求填写发布定义文件。

编写这个文件可能很繁琐。幸运的是 `yarn version check` 实现了一个非常方便的标志 `--interactive` 。当设置(`yarn version check --interactive`)时，Yarn 将显示一个终端界面，列出所有改变的文件，所有改变的工作区，所有相关的依赖工作区，以及每个条目的复选框，允许你为每个工作区挑选你想设置的发布策略。

[`changesetIgnorePatterns`](/configuration/yarnrc#changesetIgnorePatterns)配置选项可用于在检查哪些文件发生变化时忽略文件。它对于排除那些不影响发布过程的文件（例如测试文件）很有用。

### 注意事项

#### 提交历史

`version`插件需要访问提交历史，以便能够正确推断哪些包需要发布规范。特别是，当使用`actions/checkout@v2`或更高版本的 GitHub Actions 时，默认行为是 Git 只获取正在检查的版本，这将导致一些问题。为了解决这个问题，你需要覆盖`fetch-depth`配置值，以获取整个提交历史:

```yaml
- uses: actions/checkout@v2
  with:
    fetch-depth: 0
```
