# `variable` 原则

## `variable` 定义的顺序

输入变量应遵守以下顺序：

1. 各种必填项，以字母序排序
2. 各种选填项，以字母序排序

定义了 `default` 的 `variable` 为必填项，反之则为选填项。

## `variable` 原则上遵循 "判例法"

在易于理解的前提下确保 `variable` 的 `name`、`description`、`validation` 与上下游 `resource`、`data` 定义尽可能一致，并且确保不同模块间起相同作用的 `variable` 保持一致。

允许出于区分的目的为 `variable` 添加以 `_` 结尾的前缀。例如：

```hcl
resource "azurerm_linux_virtual_machine" "webserver" {
  name                = "webserver"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  source_image_id     = var.webserver_source_image_id
  ...
}
```

在这里 `webserver_source_image_id` 中，`source_image_id` 与 `resource` 中的 Argument 保持了一致，`webserver_` 作为前缀是合法的。

假如一个 `variable` 是用来传值给某个资源的输入参数，那么该 `variable` 的命名应直接使用资源定义的输入参数名称（允许添加前缀）， `description` 应尽量拷贝资源定义文档中的相关描述。

## `variable` 的命名必须遵守规范

`variable` 的命名必须遵守 [HashiCorp 定义的命名规范](https://www.terraform.io/docs/extend/best-practices/naming.html)。

用作特性开关的 `variable` 名称使用肯定式：`xxx_enabled` 而不是 `xxx_disabled` 。防止在代码中出现双重否定： `!xxx_disabled` 。

请使用 `xxx_enabled` 而非 `enable_xxx` 作为 `variable` 名称。

## 每一个 `variable` 必须定义 `description`

`description` 的目标读者是 Module 的调用者。

对一个新发明的 `variable` （例如用来开关 `dynamic` 块的 `variable`），它的 `description` 应精确描述该输入变量的目的以及期待的数据种类。`description` 中不应该描述面向模块开发者的信息，此类信息请通过代码注释的方式声明。

对于类型为 `object` 的 `variable`，可以采用 HEREDOC 格式编写 `description`：

```hcl
variable "test_obj" {
  type = object({
    id   = string
    name = string
  })
  default     = null
  description = <<-EOF
  {
    id   = "the id of this object"
    name = "the name of this object"
  }
  EOF
}
```

## 每一个 `variable` 必须定义合适的 `type`

每一个 `variable` 必须定义 `type`。`type` 应尽可能精确，拥有充分的理由才可定义为 `any`。

* 当要表达 `true`/`false` 时，使用 `bool` 类型而不是 `string` 或者 `number`
* 使用 `string` 存储文本

## `variable` 的 `validation` 的 `error_message` 应使用完整的句子描述期待的数据所必须要遵守的约束

## 包含机密数据的 `variable` 应声明 `sensitive = true`

假如 `object` 中有一个成员会被作为 `sensitive` 的参数使用，那么整个 `variable` 必须声明为 `sensitive = true`。

## 如果可能，尽可能声明 `nullable = false`

## 不要声明 `nullable = true`

## `sensitive = true` 的 `variable` 不允许声明 `default`，除非 `default = null` 或是 `default = []` 这样代表关闭的默认值。

## `variable` 的 `default` 值设定

一个 `variable` 的 `default` 值的选定按照如下顺序：

1. 有充分的原因使用某个值，例如该值是为本 Module 所设计的场景而特定的
2. 使用相关 `resource` 的 Schema 种使用的默认值。没有默认值的，`default = null`

例如，某 Module 允许用户使用 `variable` 参数定义网络访问规则:

```hcl
variable "network_security_rules" {
  type = list(object({
    name                         = string
    priority                     = number
    direction                    = string
    access                       = string
    description                  = string
    protocol                     = string
    source_port_ranges           = optional(list(string))
    destination_port_ranges      = optional(list(string))
    source_address_prefixes      = optional(list(string))
    destination_address_prefixes = optional(string)
  }))

  default = [
    {
      name                         = "ssh"
      priority                     = 4096
      access                       = "Deny"
      direction                    = "In"
      description                  = ["no remote connection"]
      protocol                     = "Tcp"
      source_port_ranges           = ["*"]
      destination_port_ranges      = ["22", "3389"]
      source_address_prefixes      = ["*", "**"]
      destination_address_prefixes = ["*"]
    }
  ]
}

resource "azurerm_network_security_rule" "example" {
  for_each                    = toset(var.network_security_rules)
  resource_group_name         = var.resource_group_name
  network_security_group_name = azurerm_network_security_group.example.name
  name                        = each.value.name
  priority                    = each.value.priority
  direction                   = each.value.direction
  access                      = each.value.access
  protocol                    = each.value.protocol
  description                 = try(each.value.description[0], null)
  source_port_ranges          = each.value.source_port_range
  destination_port_ranges     = each.value.destination_port_ranges
  source_address_prefixes     = each.value.source_address_prefixes
  destination_address_prefix  = try(each.value.destination_address_prefix[0], null)
}
```

应提供尽力确保符合安全合规的 `default` 值。

## 两个 `variable` 块之间应空一行

## 对于废弃 `variable` 的处理

有时我们会发现某些 `variable` 名称已不再合适，或是数据类型需要改变，我们需要保证在同一 Major 版本中的向前兼容，不允许直接进行修改，而是应将之移动到一个独立的 `deprecated_variables.tf` 文件中，然后在 `variable.tf` 中定义新变量，并且要在代码中做好兼容性逻辑。

废弃的 `variable` 必须在 `description` 的开头标注 `DEPRECATED`，并指示新的替代 `variable` 名称，例如：

```hcl
variable "enable_network_security_group" {
  type        = string
  default     = null
  description = "DEPRECATED, use `network_security_group_enabled` instead; Whether to generate a network security group and assign it to the subnet. Changing this forces a new resource to be created."
}
```

可以在 Major 版本升级时清理 `deprecated_variables.tf` 中的过时变量，以及相关的兼容性逻辑。