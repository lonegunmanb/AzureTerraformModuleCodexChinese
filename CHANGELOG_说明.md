# CHANGELOG 说明

我们使用 CHANGELOG.md 文件辅助测试以及其他一些自动化流程，CHANGELOG 使用 Github Action 自动生成，自动生成代码可以参考 [`terraform-azurerm-aks` 的例子](https://github.com/Azure/terraform-azurerm-aks/blob/master/.github/workflows/update-changelog.yaml)。

我们使用 CHANGELOG.md 的名字来帮助代码确认下一个准备发布的版本的 Major Version。例如一个代码仓库中包含以下几个文件：

```text
.
├── CHANGELOG-v4.md
├── CHANGELOG-v5.md
├── CHANGELOG.md
```

过去发布的 Major 最高版本号为 `v5`，所以当前预备发布的版本的 Major Version 为 `6`。假如当前目录下只有 `CHANGELOG.md` 文件，则 Major Version 为 `0`。

[版本升级测试](测试代码/版本升级测试.md)的工作也是基于该原理，它会确认当前要发布的版本的 Major Version，然后尝试从 Github 中克隆该 Major Version 下最新的版本代码，与当前 PR 版本的代码进行升级测试。如果当前要发布的 Major Version 还没有已发布的 tag，则会跳过版本升级测试。

CHANGELOG 自动生成会在代码仓库默认分支收到 `push` 推送时执行，它会从当前 Major Version 的第一个版本开始计算，将所有的 Pull Request 信息汇编成 CHANGELOG。

当我们合并一个具有 Breaking Change 效应的变更后，我们要做的事有：

1. 复制当前的 `CHANGELOG.md` 文件，新文件名为 `CHANGELOG-v<CURRENT_MAJOR_VERSION>.md`， 例如，当前 Major Version 是 `5`，那么我们这时应该有一个 `CHANGELOG-v4.md` 以及 `CHANGELOG.md` 文件。我们复制 `CHANGELOG.md` 到新文件 `CHANGELOG-v5.md`
2. 删除新文件中标明 `Unreleased` 的部分（这一部分尚未发布的 PR 将在下一个 Major Version 中被发布）
3. 抹除 `CHANGELOG.md` 文件的内容
4. 提交推送变更到代码仓库，在流水线自动更新完新的 `CHANGELOG.md` 的内容后，重新 `git pull`