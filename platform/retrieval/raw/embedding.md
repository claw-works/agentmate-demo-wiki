# embedding 配置

semantic 通路的实现。融合背景见 [混合检索](hybrid-search.md)。

## 配置项

| 环境变量 | 默认值 |
|---|---|
| `EMBEDDING_BASE_URL` | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| `EMBEDDING_MODEL` | `text-embedding-v4` |
| `EMBEDDING_DIMENSION` | `1024` |
| `EMBEDDING_ENCODING_FORMAT` | `float` |
| `EMBEDDING_API_KEY` | 无默认值，必须提供 |

走 OpenAI 兼容端点，所以换成其他兼容供应商只需改 base URL 与模型名。

## 维度与向量集合是绑定的

向量集合在创建时就固定了维度。换模型或改维度**不能只改环境变量**，
必须重建集合并全量重新索引。

这是最容易踩的运维坑：改了 `EMBEDDING_MODEL` 但没重建集合，新旧向量混在
同一个集合里，检索结果会变得莫名其妙，而且不报错。

## 失败不是致命的

embedding 调用失败时，对应记录保留失败状态并记录错误，仍可通过 lexical 通路
被检索到。索引过程不会因为单条失败而整体中止，但连续失败超过阈值会中止，
避免在供应商故障时空转。

## 成本与延迟

embedding 是外部 API 调用，是检索路径上唯一的网络往返。实测一个长查询的
端到端耗时主要花在这里，而不是数据库。

这也是 lexical 通路的另一层价值：它完全在本地，供应商故障时仍有检索能力。
