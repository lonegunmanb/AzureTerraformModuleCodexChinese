## Module 的逻辑代码

Module 的逻辑代码应由以下几部分组成

* 一组输入参数 —— `variables.tf`
* 一组输出值 —— `outputs.tf`
* 所有不依赖于 `resource` 以及 `data` 的，包含计算逻辑的表达式，均以 `local` 的形式定义于 `locals.tf` 文件
* 包含有资源定义的代码文件
* 其他
