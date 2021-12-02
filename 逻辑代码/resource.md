# 含有资源定义的代码文件原则

资源声明代码应按其业务领域聚合存放。如果 Module 定义的资源数量较少，或是类型高度内聚 （例如多个属于同一业务领域的资源），可以由一个 `main.tf` 定义；如果涉及多个不同业务领域的资源，需要以能代表其分类的文件名分别存放。模块的核心业务领域应该是中定义在`main.tf`文件中

[示例](https://github.com/cloudposse/terraform-aws-eks-cluster)

## 同一个文件中 `resource` 和 `data` 定义的顺序 。

同一文件中的资源定义，被依赖的资源应定义在前，依赖方资源定义在后。

相互之间有依赖关系的资源，尽量定义在邻近位置。

## 依赖于 `resource` 或 `data` 的 Attribute 的 `local` 定义

对于某些反复出现的表达式，或是比较复杂的表达式，为了代码可读性，我们鼓励将之抽至一个独立的 `local` 变量中引用。

`local` 定义的位置，在表达式所涉及的 `resource` 或是 `data` 中，最重要的那一个的定义块的下方。同一 `resource` 或是 `data` 块下方至多只能由一个 `locals` 块，所有定义于此的 `local` 都应该以字母序定义在该块中。两个 `local` 之间不应空行。

## `resource` 和 `data` 的 Arguments 声明顺序

定义 `resource` 和 `data` 时，Arguments 的声明顺序应保持与文档一致。
