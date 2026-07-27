# Skill Registry

Skill Registry 是建立在 Git 之上的能力控制面。Git 保存真实的 Skill package，
AgentMate 管理同步、版本、编译产物与检索投影。

关于事实语料应该放在哪里，见 [Knowledge Registry](knowledge-registry.md)；
包身份的判定规则见 [包身份](package-identity.md)。

## 分域原则

Skill 与 Knowledge 是两个独立域，不能互相打包。判断准则只有一条：

> 删掉它之后，agent 是不知道**怎么做**，还是不知道**某个事实**？

不知道怎么做的，属于 Skill；不知道事实的，属于知识库。

这条线之所以重要，是因为两者生命周期完全不同。Skill 随能力演进而改版，
一次改版通常影响一个 package；领域事实随外部世界变化而更新，一次更新可能
影响多个 Skill 的执行结果。混在一起会让两种变更互相牵连。

### 常见误判

- 把 API 字段清单写进 SKILL.md：那是事实，属于知识库。
- 把"先检索再回答"的顺序写进知识库：那是做法，属于 Skill。
- 为每个知识库配一个专属 Skill：两者是多对多，平台提供通用的生命周期
  Skill，每个知识库只需声明式的 profile。

## 同步来源

一个 skill source 是一次注册：仓库地址 + package 路径 + 默认 ref。
同步时先把 ref 解析成不可变 commit，再拉取该 commit 下的 package 目录。

同步是幂等的。相同 commit、相同包内容会返回既有版本而不是新建一个。
这是很多人第一次同步后困惑的地方：改了文件但版本号没变，通常是因为
改动落在未被 manifest 选中的文件上。

## 渐进式披露

检索层只放 L0 card（名称、版本、描述、触发条件），不放完整 instructions。
命中之后按需加载 L1，正文永远不进检索索引。

这样做的收益是上下文可控：agent 先看一批候选卡片，再决定展开哪一个，
而不是一次把所有 Skill 正文塞进上下文。

代价是召回上限被 card 的措辞锁死——`SKILL.md` 正文里的内容检索不到。
这个不对称目前是已知的，见 [领域目录布局](domain-layout.md) 之外的待议清单。

## 编译产物

编译把 package 转成 L0 catalog artifact。编译是离线、确定性的：
同一个 package hash 编译两次得到同样结果，因此产物随时可丢弃重建。

这一点与 wiki 层截然不同。wiki 由 LLM 编译，同一份来源两次编译结果不同，
所以 wiki 必须按带出处的生成物对待，不能当缓存。
