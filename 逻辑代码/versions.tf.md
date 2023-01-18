# `versions.tf` 原则

`versions.tf` 文件中应仅包含一个 `terraform` 块。

`terraform` 块第一行应按此格式定义 `required_version = ">= 1.1"`

由于 HashiCorp 承诺 1.X 版本的向前兼容，为最大限度地扩展支持的 Terraform 版本，统一规定如无特别理由，`required_version` 的值必须为 `">= 1.1"`。

`terraform` 块中应包含一个 `required_providers` 块，内容是使用的 Provider 的版本约束。Provider 版本约束声明应以字母序排序，块内的赋值也应按照字母序排序。`version` 约束必须使用 `>=`，如无特殊情况，不允许使用其他约束方式。

模块中所用到的所有 Provider 都必须在 `required_providers` 中有定义。

由于 Provider 的 Major 版本升级可能会引入破坏性变更，所以对 Provider 的 Major 版本要求的变更必须伴随 Module 的 Major 版本变更。这意味着，应始终使用以下方式声明 `azurerm` 的版本约束：

```hcl
azurerm = {
  source  = "hashicorp/azurerm"
  version = ">= 3.11, < 4.0"
}
```

## 在 Module 中声明 Provider

[原则](https://www.terraform.io/docs/language/modules/develop/providers.html)上 Module 代码中禁止声明 `provider`，唯一的例外是如果 Module 的确需要同一类 `provider` 的不同实例（例如操作跨 `location` 资源，或是跨账户资源）时，允许在 `versions.tf` 文件的 `terraform` 块下方声明 `provider` 块，并且在 `terraform.required_providers` 中的相应位置使用 `configuration_aliases` 进行关联，详情请见[文档](https://www.terraform.io/docs/language/providers/configuration.html#alias-multiple-provider-configurations)。

Module 中声明的 `provider` 块应仅用作区分 `resource` 和 `data` 使用的 `provider` 实例，禁止在 `provider` 块中声明 `alias` 以外的属性，这会导致 Module 调用者无法在 Module 上使用 `count`、`for_each`、`depends_on`。应由 Module 调用者传入 `provider` 实例配置。
