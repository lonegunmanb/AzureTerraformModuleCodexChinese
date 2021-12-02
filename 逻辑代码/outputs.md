# `output` 原则

## `output` 应按照字母序排序

## `output` 应遵守与 `variable` 类似的判例法原则

<!-- TODO: 存疑。。。 -->
## `output` 仅应输出足以使用 Module 的信息

应避免无原则地将 Module 中定义的所有 `resource` 或 `data` 作为 `output` 输出的做法。

通过 `output` 输出的信息应仅够 Module 调用者将 Module 与其他 Module 组合使用（例如输出聚合根 `resource` 的 `id`）。

通过 `variable` 输入的信息如果未经处理，不应通过 `output` 输出，因为这部分信息是 Module 调用者提供的。

对于 `resource` 中的细节信息，原则上不通过 `output` 输出，Module 的调用者应尽力自行获取细节信息（例如通过 `data` 查询）。

## `output` 的 `description` 应遵循与 `variable` 的 `description` 类似的原则

## 包含机密数据的 `output` 应声明 `sensitive = true`

## 两个 `output` 之间应空一行
