# `output` 原则

## `output` 应按照字母序排序

## `output` 的命名以及 `description` 应尽量与 `resource` 文档中相关参数的 `description` 保持一致

## 包含机密数据的 `output` 应声明 `sensitive = true`

## 两个 `output` 之间应空一行

## 对于废弃 `output` 的处理

有时我们会发现某些 `output` 名称已不再合适，我们需要保证在同一 Major 版本中的向前兼容，不允许直接进行修改，而是应将之移动到一个独立的 `deprecated-outputs.tf` 文件中，然后在 `output.tf` 中定义新输出，并且要在代码中做好兼容性逻辑。

可以在 Major 版本升级时清理 `deprecated-outputs.tf` 中的过时输出，以及相关的兼容性逻辑。