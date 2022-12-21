# 含有资源定义的代码文件原则

资源声明代码应按其业务领域聚合存放。如果 Module 定义的资源数量较少，或是类型高度内聚 （例如多个属于同一业务领域的资源），可以由一个 `main.tf` 定义；如果涉及多个不同业务领域的资源，需要以能代表其分类的文件名分别存放，例如：`virtual_machines.tf`, `network.tf`, `storage.tf` 等。模块的核心业务领域应该是中定义在 `main.tf` 文件中.

## 同一个文件中 `resource` 和 `data` 定义的顺序 。

同一文件中的资源定义，被依赖的资源应定义在前，依赖方资源定义在后。

相互之间有依赖关系的资源，尽量定义在邻近位置。

## 依赖于 `resource` 或 `data` 的 Attribute 的 `local` 定义

对于某些反复出现的表达式，或是比较复杂的表达式，为了代码可读性，我们鼓励将之抽至一个独立的 `local` 变量中引用。

如果表达式中涉及到 `resource` 或是 `data`，该 `local` 定义的位置，在表达式所涉及的 `resource` 或是 `data` 中最重要的那一个的定义块的下方。同一 `resource` 或是 `data` 块下方至多只能有一个 `locals` 块，所有定义于此的 `local` 都应该以字母序定义在该块中。两个 `local` 之间不应空行。

`locals.tf` 文件因为可能会被用于单元测试，涉及 `resource` 与 `data` 的 `local` 不应该被声明在 `locals.tf` 文件中。

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

## `resource` 块与 `data` 块的内部顺序

`resource` 和 `data` 块中的赋值语句分为三种：Argument 、Meta-Argument与 Nested Block。Argument 的赋值表达式为参数名后接 `=` 的，例如：

```hcl
location = azurerm_resource_group.example.location
```

又或者：

```hcl
tags = {
  environment = "Production"
}
```

Nested Block 的赋值表达式为参数名后接一个 `{}` 块，例如：

```hcl
subnet {
  name           = "subnet1"
  address_prefix = "10.0.1.0/24"
}
```

Meta-Argument 为所有 `resource` 与 `data` 块都可以声明的赋值语句，有：

* `count`
* `depends_on`
* `for_each`
* `lifecycle`
* `provider`

`resource` 与 `data` 块内部赋值语句的声明顺序应为：

以下 Meta-Argument 在 `resource` 和 `data` 块中应声明在最上方，按顺序排列：

1. `provider`
2. `count`
3. `for_each`

然后是必填的 Argument，以字母顺序排列

然后是选填的 Argument，以字母顺序排列

然后是必填的 Nested Block，以字母顺序排列

然后是选填的 Nested Block，以字母顺序排列

以下 Meta-Argument 在 `resource` 块中应声明在最下方，按顺序排列：

1. `depends_on`
2. `lifecycle`

其中，`lifecycle` 块中的参数应以以下顺序出现：

* `create_before_destroy`
* `ignore_changes`
* `prevent_destroy`

`depends_on` 和 `ignore_changes` 的成员以字母顺序排序。

Meta-Argument、Argument、Nested Block 之间，以空行分隔。

以 `dynamic` 形式出现的  Nested Block，以 `dynamic` 后的名字作为排序依据，例如：

```hcl
  dynamic "linux_profile" {
    for_each = var.admin_username == null ? [] : ["linux_profile"]

    content {
      admin_username = var.admin_username

      ssh_key {
        key_data = replace(coalesce(var.public_ssh_key, tls_private_key.ssh[0].public_key_openssh), "\n", "")
      }
    }
  }
```

该 `dynamic` 块视同一个 `linux_profile` 块进行排序，Meta-Argument、Argument、Nested Block 之间，以空行分隔。

在 Nested Block 中的代码也按照上述规则排序，

## `module` 块的内部顺序

以下 Meta-Argument 在 `resource` 块中应声明在最上方，按顺序排列：

1. `source`
2. `version`

然后是

3. `count`
4. `for_each`

`source`、`version` 与 `count`、`for_each` 之间应以空行分隔。

然后是必填的 Argument，以字母顺序排列

然后是选填的 Argument，以字母顺序排列

以下 Meta-Argument 在 `resource` 块中应声明在最下方，按顺序排列：

1. `depends_on`
2. `providers`

Argument 部分与 Meta-Argument 之间应以空行分隔。

## 传递给 `provider`, `depends_on`, `lifecycle` 块中的 `ignore_changes` 中的值，不允许使用双引号

## 对于可以定义 `tags` 的资源，始终应该通过 `variable` 暴露给 Module 调用者，使其可以设置 `tags`

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

## 可选的 Nested Object Argument 要配合 `dynamic`

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

请参照例子中的写法，如果只是想有条件地声明某个 Nested 块时，请使用：

```hcl
for_each = <condition> ? [<some_item>] : []
```

## 为可空的表达式配置默认值时使用 `coalesce`

以下例子可以在 `var.new_network_security_group_name` 为 `null` 或是 `""` 时使用 `"${var.subnet_name}-nsg"`

```hcl
coalesce(var.new_network_security_group_name, "${var.subnet_name}-nsg")
```