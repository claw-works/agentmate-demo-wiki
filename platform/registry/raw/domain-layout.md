# 领域目录布局

Skill package 与 KB package 都按 **领域 / 主题** 两级组织。

## 推导规则

`domain` 从 `package_path` 首段推导，作为一等字段存储，用于 catalog 分组与
检索范围收窄。

| package_path | domain | source name |
|---|---|---|
| `platform/retrieval` | `platform` | `platform-retrieval` |
| `product/support` | `product` | `product-support` |
| `grounded-answer` | 无 | `grounded-answer` |

## 单段路径没有 domain

这是刻意的约定。扁平 package 并未按领域组织，把它自己的名字当作领域，
会在 catalog 里凭空造出一个只含一个成员的分组——那个分组不表达任何真实的
归类关系。未分类记为空值而不是 NULL，避免比较时出现三值逻辑。

## name 必须包含完整路径

knowledge source 的唯一键是账号加名称。如果 name 用路径的最后一段推导，
`platform/retrieval` 与 `product/retrieval` 都会得到 `retrieval`，
第二个注册会静默覆盖第一个。

拼接完整路径段后两者分别是 `platform-retrieval` 与 `product-retrieval`，
不再冲突。

## domain 不接受外部传入

注册时不能声明 domain，只能由 package 位置推导。否则会出现声明的领域与
实际存放位置矛盾的记录，而这种矛盾没有正确的裁决方式。

## 拆分成本

按领域分目录还有一个实际好处：将来某个领域需要独立成仓库时，
把一级目录整体搬走即可，package 内部结构与身份计算都不受影响。
