# `variable` 原则

## `variable` 定义的顺序

输入变量应遵守以下顺序：

1. `resource_group_name` （如果有的话）
2. `location` （如果有的话）
3. 各种必填项(`tags`除外)，以字母序排序
4. 各种选填项(`tags`除外)，以字母序排序
5. `tags` （如果有的话）

## `variable` 原则上遵循 "判例法"

应尽力确保 `variable` 的 `name`、`description`、`validation` 与上下游 `resource`、`data` 定义一致，并且确保不同模块间起相同作用的 `variable` 保持一致。

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

假如一个 `variable` 是用来传值给某个资源的输入参数，那么该 `variable` 的命名应直接使用资源定义的输入参数名称（允许添加资源类型作为前缀）， `description` 应直接拷贝资源定义文档中的相关描述（如果资源定义文档中的描述不包含资源类型，需要在头部使用\`<resource_type>\`标注资源类型）。

假如一个起特定作用的 `variable` 在之前公开发布过的模块中已经存在，那么该 `variable` 的定义应尽量拷贝已经存在的定义，除非这么做会引发明显不合适的结果。

如无必要，尽量不要发明新的 `variable` 。一个新的 `variable` 将会成为后续同类型 `variable` 的模板，需要相当严格的审核。

假如上下游 `resource`、`data` 中相关定义与本规范矛盾，本规范拥有更高优先级。当某个 `variable` 对应的上游 `resource`、`data` 中相关定义与本规范矛盾时，应首先确认该 `variable` 是否已在其他已发布模块中有过定义。如果有，请遵循之前定义的原则，尽量拷贝；如果没有，则视为一个新发明的 `variable`。假如上下游 `resource`、`data` 之间对同一 `variable` 的定义不同，则遵循上述原则；如果没有前例可以拷贝，则可以选择上游下游之中更为妥当的那一个拷贝。

应以某种合适的方式保存所有过去新发明的 `variable` 的相关信息，并合理组织之以便于后续的检索。

## `variable` 的命名必须遵守规范

`variable` 的命名必须遵守 [HashiCorp 定义的命名规范](https://www.terraform.io/docs/extend/best-practices/naming.html)。

用作特性开关的 `variable` 名称使用肯定式：`xxx_enabled` 而不是 `xxx_disabled` 。防止在代码中出现双重否定： `!xxx_disabled` 。

## 每一个 `variable` 必须定义 `description`

`description` 的目标读者是 Module 的调用者。默认情况下要遵守上文定义的判例法原则。

对一个新发明的 `variable` ，它的 `description` 应精确描述该输入变量的目的以及期待的数据种类。`description` 中不应该描述面向模块开发者的信息，此类信息请通过代码注释的方式声明。

## 每一个 `variable` 必须定义合适的 `type`

每一个 `variable` 必须定义 `type`。`type` 应尽可能精确，拥有充分的理由才可定义为 `any`。

* 当要表达 `true`/`false` 时，使用 `bool` 类型而不是 `string` 或者 `number`
* 使用 `string` 存储文本
* 不要滥用 `object` 类型，为 `object` 类型编写校验规则以及文档比较困难
* 假如要定义 `object` 下某个可选字段时，应将该可选字段定义为 `list(sometype)` ，并在 `validation` 中限制其长度为 `0` 或 `1` ，并且应在相应的 `description`以及 错误提示信息中提示调用者该字段为可选字段，可接受的 `list` 长度为 `0` 或 `1` 。

Module 中的代码在使用 `object` 下某个可选字段时，应采用 `argument = try(each.value.optional_property[0], null)` 的方式赋值

## `variable` 的 `validation` 应与下游资源在 Provider 中定义的检查规则保持正交

如果某一 `variable` 在对应的上下游资源中已经有合法性检查校验，那么不应该在 `validation` 中重复定义这些约束。`validation` 应定义针对 Module 设计场景特有的约束，或是为使 Module 代码顺利执行所必须遵守的约束。

## `variable` 的 `validation` 的 `error_message` 应使用完整的句子描述期待的数据所必须要遵守的约束

## 包含机密数据的 `variable` 应声明 `sensitive = true`

## `sensitive = true` 的 `variable` 不允许声明 `default`

## 如果可能， `variable` 应设置符合最佳实践的 `default` 值

例如，某 Module 允许用户使用 `variable` 参数定义网络访问规则:

```hcl
variable "network_security_rules" {
  type = list(object({
    name                       = string
    priority                   = number
    direction                  = string
    access                     = string
    description                = list(string)
    protocol                   = string
    source_port_ranges         = list(string)
    destination_port_ranges    = list(string)
    source_address_prefix      = list(string)
    destination_address_prefix = list(string)
  }))

  validation {
    condition     = alltrue([for rule in var.network_security_rules : (length(rule.description) <= 1)])
    error_message = "`network_security_rules.description` is optional, the length of list should either be `0` or `1`."
  }
  validation {
    condition     = alltrue([for rule in var.network_security_rules : (length(rule.source_port_ranges) > 0)])
    error_message = "`network_security_rules.source_port_ranges` is required, the length of list should greater than `0`."
  }
  validation {
    condition     = alltrue([for rule in var.network_security_rules : (length(rule.destination_port_ranges) > 0)])
    error_message = "`network_security_rules.destination_port_ranges` is required, the length of list should greater than `0`."
  }
  validation {
    condition     = alltrue([for rule in var.network_security_rules : (length(rule.source_address_prefix) <= 1)])
    error_message = "`network_security_rules.source_address_prefix` is optional, the length of list should either be `0` or `1`."
  }
  validation {
    condition     = alltrue([for rule in var.network_security_rules : (length(rule.destination_address_prefix) <= 1)])
    error_message = "`network_security_rules.destination_address_prefix` is optional, the length of list should either be `0` or `1`."
  }

  default = [
    {
      name                       = "ssh"
      priority                   = 4096
      access                     = "Deny"
      direction                  = "In"
      description                = ["no remote connection"]
      protocol                   = "Tcp"
      source_port_ranges         = ["*"]
      destination_port_ranges    = ["22", "3389"]
      source_address_prefix      = ["*", "**"]
      destination_address_prefix = ["*"]
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
  source_address_prefix       = try(each.value.source_address_prefix[0], null)
  destination_address_prefix  = try(each.value.destination_address_prefix[0], null)
}
```

应提供尽力确保符合安全合规的 `default` 值。

## 两个 `variable` 块之间应空一行
