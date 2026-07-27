# 包身份

Skill package 与 KB package 共用同一套身份规则。回到上层背景见
[Skill Registry](skill-registry.md) 与 [Knowledge Registry](knowledge-registry.md)。

## 完整 package 决定身份

身份不是由某个入口文件决定，而是由 manifest 选中的**全部文件内容**决定。
计算方式是对选中文件的路径与内容做规范化排序后取 canonical hash。

推论：

- 改动 `templates/` 下的任意文件会产生新版本，即使 `SKILL.md` 一字未动。
- 改动未被选中的文件不产生新版本。
- 仅移动文件位置也会改变身份，因为路径参与计算。

## commit 与 package hash 是两个概念

| 概念 | 含义 | 何时变化 |
|---|---|---|
| commit SHA | 仓库某次提交 | 任何文件改动 |
| package hash | 该 package 选中文件的内容身份 | 只有选中文件改动时 |

一次 commit 可能不改变任何 package hash（例如只改了 README），
也可能同时改变多个 package 的 hash。

所以"同步成功但版本没变"通常不是 bug：commit 变了，package hash 没变。

## 为什么不用 commit 作为版本

用 commit 当版本会让无关改动制造出大量语义相同的版本，编译产物与
embedding 都要跟着重建，成本白花。用内容身份可以让"没有实质变化"这件事
在系统里被表达出来。

## 归档格式的坑

从 GitHub / GitLab 拉取 tarball 时有两个真实陷阱：

1. tarball 的首个条目可能是 `pax_global_header` 这类元数据条目，
   把它当成归档根目录会导致"包内出现多个根目录"的误判。
2. 请求归档时 `Accept: application/octet-stream` 会被拒绝，需要使用 `*/*`。

这两点在自建测试归档上都不会出现，只有真实 provider 才暴露。
