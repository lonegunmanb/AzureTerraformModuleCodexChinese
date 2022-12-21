# `local` 原则

为提升代码可读性，可将一些复杂的表达式，或是重复出现的表达式抽取成一个 `local` 定义。凡是不依赖 `resource` 以及 `data` 的 `local` 定义都应集中于 `locals.tf` 文件。

## `locals.tf` 文件中应只包含一个 `locals` 块

`locals.tf` 文件中的所有的 `local` 都应该定义在这唯一的一个 `locals` 块内部

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

## 两行 `local` 之间不应空行

## 对于复杂的 `local` 表达式，应编写相应的[单元测试](../测试代码/单元测试.md)覆盖各种场景。