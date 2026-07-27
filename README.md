# AgentMate Demo Wiki

用于验证 AgentMate Knowledge Registry 端到端链路的示例知识库来源。

## 这个仓库里只有 raw sources

三层模型是 **raw sources / wiki / schema**：

| 层 | 谁拥有 | 存在哪里 |
|---|---|---|
| raw sources | 人 | **本仓库**，immutable，唯一事实源 |
| wiki | 平台侧 LLM 编译 | AgentMate 内部，带出处的生成物 |
| schema | 人与平台共同演进 | 每个 package 的 `KNOWLEDGE.yaml` |

wiki 层（summary / entity / concept 页、`index.md`、`log.md`）由平台侧编译产生，
**不提交到本仓库**。原因是 agent 运行在客户端、行为不可控，不能让它写事实源；
交给平台编译才能保证出处可审计、可复现追溯。

因此这里看不到 `wiki/` 目录——那不是遗漏。人只负责往 `raw/` 里放来源，
编译、交叉引用、矛盾标注、索引与日志都是平台的活。

## 目录规划：领域 / 主题两级

```
platform/               ← 领域（domain）
  registry/             ← KB package
    KNOWLEDGE.yaml
    raw/
  retrieval/            ← KB package
    KNOWLEDGE.yaml
    raw/
product/                ← 领域（domain）
  support/              ← KB package
    KNOWLEDGE.yaml
    raw/
```

**每个二级目录是一个独立 KB package**，注册为独立 knowledge source，
`package_path` 指向该目录（例如 `platform/retrieval`）。

一级目录是领域。AgentMate 从 `package_path` 首段推导 `domain`，用于 catalog 分组
与检索范围收窄。注意：

- `package_path` 只有单段时**没有 domain** —— 扁平 package 并未按领域组织，
  把自身名当领域会凭空造出一个不存在的分组。
- source name 由完整路径段拼接（`platform/retrieval` → `platform-retrieval`），
  所以 `platform/` 与 `product/` 下即使出现同名主题目录也不会互相覆盖。
- domain 不接受注册时传入，一律由 package 位置推导。

## Package 清单

| package_path | 主题 | 文档数 |
|---|---|---|
| `platform/registry` | Skill 与 Knowledge 的注册、包身份、领域布局 | 4 |
| `platform/retrieval` | 混合检索、CJK lexical、embedding、分块 | 4 |
| `product/support` | 常见问题、故障排查、已知限制 | 3 |

## 故意留下的检验点

这些不是疏漏，是用来验证平台行为的：

- `platform/registry/raw/drafts/` 被 `KNOWLEDGE.yaml` 的 exclude 规则排除，
  用于确认未选中的文件既不进入 ingest，也不参与 package identity。
- `product/support/raw/limitations.md` 与 `platform/retrieval/raw/cjk-lexical.md`
  对同一问题的表述存在**时间差**（一处仍称中文检索不可用），用于验证 lint
  能发现过期声明。
- `platform/registry/raw/domain-layout.md` 没有任何入链，用于验证孤立页面检测。
- `product/support/raw/troubleshooting.md` 提到跨知识库的概念但不建链，
  用于确认链接图只在 package 内解析。
