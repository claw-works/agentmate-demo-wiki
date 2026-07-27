# Knowledge Registry

Knowledge Registry 保存领域事实语料，与 [Skill Registry](skill-registry.md) 是两个
独立的域。包身份规则两者共用，见 [包身份](package-identity.md)。

## 最小 KB 单元

一个 KB package 是一个目录，根上必须有 `KNOWLEDGE.yaml`。这个 manifest 声明：

| 字段 | 作用 |
|---|---|
| `name` | 集合的展示名 |
| `description` | 用于 catalog 卡片与库级选择 |
| `profile` | 声明这个库遵循哪套编译与引用约定 |
| `language` | 主语料语言，影响检索策略 |
| `citation_policy` | 回答时是否强制引用 |
| `include` / `exclude` | 选中哪些文件进入 ingest |

### package identity 语义

manifest 本身与 manifest 选中的文件共同参与 canonical hash。未被选中的文件
既不进入 ingest，也不参与身份计算——所以往 `drafts/` 里加文件不会产生新版本。

这条规则让草稿目录可以安全地留在仓库里。

## 三层模型

- **raw sources**：人放进仓库的来源文档，immutable，唯一事实源。
- **wiki**：平台侧 LLM 编译出的综合页面，带出处，可追溯，但**不可重现**。
- **schema**：`KNOWLEDGE.yaml` 与 profile，规定编译与引用的约定。

wiki 层由平台编译而非客户端 agent 生成。agent 运行在客户端、行为不可控，
让它写事实源等于把数据质量外包给一个无法约束的进程。

### 为什么 wiki 不是缓存

Skill 的编译产物是离线确定性的，删了重跑得到同样结果，可以随便丢。
LLM 编译不是：同一份 raw 编译两次，结果不同。

因此每次编译必须记全出处——raw 的 package hash、compiler 版本、模型、
prompt 版本、输出快照——并且 wiki 快照要 immutable、可 diff、可导出。
它是客户数据，不能按缓存处理。

## 检索与记忆的分工

知识库存放**经人工审校的领域事实**；memory 存放运行时产生的事实
（本次会话的目标、遇到的问题、得出的结论）。

memory 晋升进知识库必须同时满足使用信号门槛与人工审批，禁止自动写入。
这道门守护的是知识库作为事实源的纯度。详见产品侧的常见问题。
