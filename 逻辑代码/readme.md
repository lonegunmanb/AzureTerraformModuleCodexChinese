## Module 的逻辑代码
<!-- TODO:待 《Terraform Style Guide》正式发布后补充此处链接 -->
Module 的所有 `.tf` 代码都必须首先遵循 [Terraform Style Guide]() 的规定。

Module 的逻辑代码应由以下几部分组成

* 一组输入参数 —— `variables.tf`
* 一组输出值 —— `outputs.tf`
* 所有不依赖于 `resource` 以及 `data` 的，包含计算逻辑的表达式，均以 `local` 的形式定义于 `locals.tf` 文件
* 包含 `terraform` 块的 `versions.tf` 文件
* 包含有资源定义的代码文件
* (可选) 包含废弃输入参数和输出值的 `variables-deprecated.tf` 和 `outputs-deprecated.tf`
* 其他

## 为什么需要一个独立的 `locals.tf` 文件？

对于某些不依赖于 `resource` 以及 `data` 的，包含计算逻辑的表达式，其逻辑可能相当复杂，我们可以为之编写一些单元测试，这种单元测试将利用软链接在测试目录下引用 `locals.tf` 文件。我们将在测试章节中详细介绍。
