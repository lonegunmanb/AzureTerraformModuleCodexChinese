# 含有资源定义的代码文件原则

资源声明代码应按其业务领域聚合存放。如果 Module 定义的资源数量较少，或是类型高度内聚 （例如多个属于同一业务领域的资源），可以由一个 `main.tf` 定义；如果涉及多个不同业务领域的资源，需要以能代表其分类的文件名分别存放。模块的核心业务领域应该是中定义在 `main.tf` 文件中

[示例](https://github.com/cloudposse/terraform-aws-eks-cluster)

## 同一个文件中 `resource` 和 `data` 定义的顺序 。

同一文件中的资源定义，被依赖的资源应定义在前，依赖方资源定义在后。

相互之间有依赖关系的资源，尽量定义在邻近位置。

## 依赖于 `resource` 或 `data` 的 Attribute 的 `local` 定义

对于某些反复出现的表达式，或是比较复杂的表达式，为了代码可读性，我们鼓励将之抽至一个独立的 `local` 变量中引用。

如果表达式中涉及到 `resource` 或是 `data`，该 `local` 定义的位置，在表达式所涉及的 `resource` 或是 `data` 中最重要的那一个的定义块的下方。同一 `resource` 或是 `data` 块下方至多只能有一个 `locals` 块，所有定义于此的 `local` 都应该以字母序定义在该块中。两个 `local` 之间不应空行。

## `resource` 和 `data` 的 Arguments 声明顺序

定义 `resource` 和 `data` 时，Arguments 的声明顺序应保持与文档一致。

## 使用 `count` 和 `for_each`

我们可以使用 `count` 和 `for_each` 来部署多个资源，但对 `count` 的不正确使用会导致[意外的行为](https://github.com/lonegunmanb/unpredictable_tf_behavior_sample)

只有在创建一组完全一致，或是接近一致的资源时，可以使用 `count`。比如说，如果我们使用 `count` 去迭代一个 `list(string)`，那大概率是错误的，因为修改列表中的元素会导致资源的顺序发生变化，引发难以预知的问题。

另一种使用 `count` 的场景是有条件创建某种资源，例如：

```hcl
resource "azurerm_network_security_group" "this" {
  count               = local.create_new_security_group ? 1 : 0
  name                = coalesce(var.new_network_security_group_name, "${var.subnet_name}-nsg")
  resource_group_name = var.resource_group_name
  location            = local.location
  tags                = var.new_network_security_group_tags
}
```

## Meta-Arguments 的顺序

`resource` 上声明的 Meta-Argument 应以文档中介绍的顺序出现：

* `depends_on`
* `count`
* `for_each`
* `provider`
* `lifecycle`

其中，`lifecycle` 块中的参数应以一下顺序出现：

* `create_before_destroy`
* `prevent_destroy`
* `ignore_changes`

`depends_on` 和 `ignore_changes` 的成员以字母顺序排序。

## 对于可以定义 `tags` 的资源，始终应该通过 `variable` 暴露个 Module 调用者，使其可以设置 `tags`

## 根据输入参数是否为 `null` 来判定是否创建某种资源的场景

有时我们要确保创建的资源符合某种最低限度的合规，例如所有的 `subnet` 都要关联至少一个 `network_security_group`。用户可能会传递一个 `security_group_id` 要求我们关联到一个已经存在的 `security_group` 上，也有可能用户希望我们为其创建安全组。

直觉上我们会直接这样定义：

```hcl
variable "security_group_id" {
  type = string
}

resource "azurerm_network_security_group" "this" {
  count               = var.security_group_id == null ? 1 : 0
  name                = coalesce(var.new_network_security_group_name, "${var.subnet_name}-nsg")
  resource_group_name = var.resource_group_name
  location            = local.location
  tags                = var.new_network_security_group_tags
}
```

这种做法的缺点是如果用户直接在 Root Module 中创建安全组，并将其 `id` 作为 Module 的 `variable` 时，会引发一个问题，决定 `count` 具体值的表达式中出现了另一个 `resource` 的 `attribute`，该值在 Plan 阶段为 Known After Apply，所以 Terraform Core 会认为无法在 Plan 阶段得到一个确定的计划。

这种参数我们推荐使用 `object` 类型进行一次包装：

```hcl
variable "security_group" {
  type = object({
    id   = string
  })
  default     = null
}
```

这样做的优点是将 Known After Apply 值封装在 `object` 内部，而 `object` 本身的引用是可以轻松判定是否为 `null` 的。鉴于一个 `resource` 的 `id` 不可能为 `null`，所以这种用法可以避免这种尴尬的情况。

仅在根据输入参数是否为 `null` 来判定是否创建某种资源的场景下使用该技巧。

## 可选的 Embedded Object Argument 要配合 `dynamic`

一个社区的例子是：

```hcl
resource "azurerm_kubernetes_cluster" "main" {
  ...
  dynamic "identity" {
    for_each = var.client_id == "" || var.client_secret == "" ? [1] : []
    content {
      type                      = var.identity_type
      user_assigned_identity_id = var.user_assigned_identity_id
    }
  }
  ...
}
```

请参照例子中的写法，如果只是想有条件地声明某个 Embedded 块时，请使用：

```hcl
for_each = <condition> ? [1] : []
```

## 为可空的表达式配置默认值时使用 `coalesce`

以下例子可以在 `var.new_network_security_group_name` 为 `null` 或是 `""` 时使用 `"${var.subnet_name}-nsg"`

```hcl
coalesce(var.new_network_security_group_name, "${var.subnet_name}-nsg")
```

## 应使用特性开关保证版本的向前兼容，避免升级引发的令人意外的变更

举例，上一个 release 版本号是 `v1.2.1`，此时我们希望提交一个 Pull Request，添加一个新的资源：

```hcl
resource "azurerm_route_table" "this" {
  location            = local.location
  name                = coalesce(var.new_route_table_name, "${var.subnet_name}-rt")
  resource_group_name = var.resource_group_name
}
```

这样做会导致用户一旦升级了 Module 版本，会惊讶地看到新生成的 Plan 中多了一个新的待创建的 `azurerm_route_table`。

正确的做法是必须为之设计一个特性开关，并且默认关闭：

```hcl
variable "create_route_table" {
  type    = bool
  default = false
}

resource "azurerm_route_table" "this" {
  count               = var.create_route_table ? 1 : 0
  location            = local.location
  name                = coalesce(var.new_route_table_name, "${var.subnet_name}-rt")
  resource_group_name = var.resource_group_name
}
```

同理，如果为 Resource 设置新的 Argument 时，应使它的默认值为 `null`。如果设置一个新的 Embedded Object 时，应使用 `dynamic`，并默认不生成该块。

## 必须引入破坏性变更

如果因为某些原因，必须引入破坏性变更，则需要