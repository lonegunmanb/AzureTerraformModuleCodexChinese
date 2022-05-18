# `output` 原则

## `output` 应按照字母序排序

## `output` 的命名以及 `description` 应遵守与 `variable` 类似的判例法原则

## `output` 应当包含哪些信息？

应避免无原则地将 Module 中定义的所有 `resource` 或 `data` 作为 `output` 输出的做法。

所有

如果一个 Argument 或是 Attribute 可以通过 `data` 查询到，那么原则上就不应通过 `output` 输出。

## 包含机密数据的 `output` 应声明 `sensitive = true`

## 两个 `output` 之间应空一行

## 对于废弃 `output` 的处理

有时我们会发现某些 `output` 名称已不再合适，我们需要保证在同一 Major 版本中的向前兼容，不允许直接进行修改，而是应将之移动到一个独立的 `outputs-deprecated.tf` 文件中，然后在 `output.tf` 中定义新输出，并且要在代码中做好兼容性逻辑。

可以在 Major 版本升级时清理 `outputs-deprecated.tf` 中的过时输出，以及相关的兼容性逻辑。