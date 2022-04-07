# `local` 原则

为提升代码可读性，可将一些复杂的表达式，或是重复出现的表达式抽取成一个 `local` 定义。凡是不依赖 `resource` 以及 `data` 的 `local` 定义都应集中于 `locals.tf` 文件。

## `locals.tf` 文件中应只包含一个 `locals` 块

所有的 `local` 都应该定义在这唯一的一个 `locals` 块内部

## `local` 应按照字母序排列

## `local` 应尽可能使用精确的类型

例如：`object({ name=string, age=number })` 的类型：

```hcl
{
  name = "John"
  age  = 52
}
```

就优于 `map(string)` 的类型：

```hcl
{
  name = "John"
  age  = "52"
}
```

## `local` 命名不需要严格遵守判例法原则

我们鼓励为 `local` 取名时遵守判例法原则，然而由于 `local` 被封装于 Module 内部，所以这并不是一个强制原则。

## 两行 `local` 之间不应空行
