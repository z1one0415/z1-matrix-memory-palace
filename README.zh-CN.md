# Z1 矩阵记忆宫殿 2.0

**英文名：Z1 Matrix Memory Palace 2.0**

一套面向多智能体工作流的文件驱动记忆架构，内置向量检索、记忆脱水与严格的上下文预算纪律。

## 概述
Z1 矩阵记忆宫殿是一个开源蓝图，用于为自主智能体工作流构建长期、高密度的记忆系统。

本项目不把记忆视为临时检索层，而是将其视为结构化运行层：
- 原始产出存放于文件中
- 任务通过显式路径流转
- 项目结果被代谢为可复用的原则
- 持续编译的维基式层作为唯一事实来源

**宫殿 2.0** 引入三大核心升级：
1. **Watchdog（看门狗）** – 轻量向量路由器，用 `query_memory_palace` 返回最相关的 Top‑k 摘要。
2. **Archivist（档案员）** – 脱水流水线，将已完成的任务链压缩为电报体摘要并提炼为可复用逻辑。
3. **上下文预算宪法** – 硬性限制历史上下文注入，防止 token 浪费与“迷失在中段”的漂移。

## 灵感来源
本项目明确受到以下启发：
- **Andrej Karpathy 的 llm‑wiki 理念** – 知识应持续编译为持久、结构化的中间层。
- **Jeff Pierce 的 memory‑palace 方法** – 长期记忆应作为持久、连接的系统层存在于模型之外。
- **BAAI 的 bge‑m3 嵌入模型** – 领先的多语言嵌入模型，支持高效的本地语义搜索。

## 核心合成
宫殿 2.0 将五个理念融合为一个运行时架构：
- **记忆宫殿作为空间骨架**
- **LLM 维基作为编译灵魂**
- **Watchdog 作为向量路由器**
- **Archivist 作为记忆脱水器**
- **文件驱动宪法作为运行纪律**

简言之：
- 宫殿定义了记忆的存放位置以及工作在不同房间之间的流转路径
- 维基定义了如何将工作产物转化为稳定的知识
- Watchdog 确保智能体在深入原始文件之前先找到正确的摘要
- Archivist 持续将长任务链脱水为可复用资产
- 宪法严格控制 token 消耗

## 架构
系统分为四层：

1. **原始层** – 工作产物、日志、任务输出。
2. **宫殿层** – 房间、路径、项目容器、调度记录。
3. **Watchdog 层** – 高密度摘要的向量索引，服务于 `query_memory_palace`。
4. **反思层** – 原则、提示内核、失败模式、思考路径、宪法候选。

### 宫殿拓扑
建议的核心空间：
- 大厅（控制面板、概览页）
- 智能体室（各智能体专属索引）
- 项目室（各项目的索引、链图、回填）
- 调度走廊（已完成的任务卡）
- 反思翼（电报、原则、内核、模式）
- 档案地下室（冷存储，极少访问的原始日志）

每个房间不仅是文件夹隐喻，它代表一个语义域、路由规则、维护责任和生命周期规则。

## 为何文件驱动？
本项目采用文件驱动宪法：
- 文件优先，而非聊天密集型协作
- 默认单主执行链
- 结果后报告，而非在每个中间状态都报告
- 工作区作为状态面板
- 只有被写下来才算完成

## 宫殿 2.0 组件

### 1. Watchdog – 向量路由器
Watchdog 是记忆宫殿的守门单元。它不生成文本、不总结、不仲裁。它的唯一工作是将智能体的查询路由到最相关的 1–3 个高密度摘要页。

**职责：**
- 将查询和摘要页转换为嵌入向量。
- 维护轻量本地向量索引（如 SQLite、ChromaDB 或简单的 JSONL 近似最近邻）。
- 返回 Top‑k 候选路径，可附带房间/类型标签。
- 强制上下文预算入口：注入的历史总量不超过 1200 token。

**技术栈（alpha 阶段）：**
- 嵌入模型：`BAAI/bge‑m3`（多语言，1024 维，Apache‑2.0 许可证）。
- 索引：本地向量存储，余弦相似度。
- 索引范围：仅“热区”摘要页（大厅控制面板、项目室索引、反思回填等）。

**默认查询流：**
```python
# 伪代码
results = watchdog.query("我们是如何处理大文件读取限制的？")
# 返回 [(路径, 得分), ...]
```

### 2. Archivist – 记忆脱水器
Archivist 是一个后台编译器，将已完成的任务链压缩为电报体摘要，并在适当时将其提升为可复用的反思资产。

**职责：**
- 读取已完成的走廊卡片及对应的项目室材料。
- 将长链脱水为 150–400 token 的电报体（存放于 `reflection_wing/telegrams/`）。
- 判断电报是否具有跨任务价值；若有，则提升为原则、提示内核、失败模式、思考路径或宪法候选。
- 删除叙述泡沫：中间闲聊、礼貌性确认、低增量试错描述。

**脱水模板（保留四部分）：**
1. 背景
2. 失败尝试及其原因
3. 最终有效解法
4. 可泛化原则

### 3. 上下文预算宪法
一套硬性规则集，防止历史记忆挤压当前任务。

**默认预算：**
- 系统/人格/宪法：500–800 token。
- 当前任务描述：800–1500 token。
- Watchdog 注入的历史：≤1200 token。
- 原始文件深读：按明确路径逐行读取，不批量灌入。

**前线智能体读取纪律：**
1. 读取大厅控制面板。
2. 读取相关项目室/智能体室索引。
3. 读取反思页。
4. 若仍不足，调用 `read_drawer_file(确切路径, 起始行, 结束行)`。

### 4. 查询协议

#### `query_memory_palace`
- 当智能体缺乏项目历史、旧决策或矩阵上下文时，首先调用的检索接口。
- 返回 Top‑3 相关摘要，总量 ≤500 token。
- 若摘要不足，智能体可调用 `read_drawer_file`。

#### `read_drawer_file`
- 在 `query_memory_palace` 之后的精确深读后门。
- 接受确切文件路径和行范围。
- 禁止用作全仓库扫描工具。

#### 大文件读取限制
- 超过阈值的文件触发“文件过大，请改用 `query_memory_palace`”响应。
- 迫使智能体养成“先检索，后局部深读”的习惯。

## 本地向量模型：bge‑m3 部署

### 模型卡片
- **名称：** BAAI/bge‑m3
- **维度：** 1024
- **许可证：** Apache‑2.0
- **语言：** 多语言（中英优化，支持 100+ 语言）
- **性能：** 在 MTEB 和 C‑MTEB 基准测试中达到领先水平。

### 安装
```bash
# 安装 sentence‑transformers（或其他你偏好的嵌入库）
pip install sentence‑transformers

# 可选：安装本地向量存储（ChromaDB、FAISS 等）
pip install chromadb
```

### 快速启动脚本
```python
from sentence_transformers import SentenceTransformer
import numpy as np
import json
import os

# 加载模型（首次下载后会缓存到本地）
model = SentenceTransformer('BAAI/bge-m3')

# 索引时编码摘要
summaries = ["..."]  # 摘要文本列表
paths = ["..."]       # 对应文件路径
embeddings = model.encode(summaries, normalize_embeddings=True)

# 存储嵌入向量和路径（最简单的例子：JSONL）
with open("memory/palace/watchdog_index.jsonl", "w") as f:
    for path, emb in zip(paths, embeddings):
        f.write(json.dumps({"path": path, "embedding": emb.tolist()}) + "\n")

# 查询
query = "我们如何处理上下文预算违规？"
query_emb = model.encode([query], normalize_embeddings=True)[0]

# 加载索引并计算余弦相似度（小规模集合可用线性扫描，大规模请换用 ANN 库）
best_score = -1
best_path = None
with open("memory/palace/watchdog_index.jsonl", "r") as f:
    for line in f:
        item = json.loads(line)
        score = np.dot(query_emb, np.array(item["embedding"]))
        if score > best_score:
            best_score = score
            best_path = item["path"]

print(f"最佳匹配：{best_path}（得分：{best_score:.3f}）")
```

### 注意事项
- 示例使用简单的 JSONL 索引以便理解，生产环境可使用 ChromaDB、FAISS 或 SQLite 向量存储。
- 保持索引轻量，仅索引“热区”摘要，不索引原始日志。
- 嵌入生成可批量处理并缓存；每当写入新的高密度摘要时更新索引。

## 仓库结构
```text
z1‑矩阵‑记忆‑宫殿/
├─ README.md
├─ README.zh‑CN.md
├─ LICENSE
├─ NOTICE（第三方模型署名声明）
├─ docs/
│  ├─ architecture/
│  │  ├─ palace‑2.0‑overview.md
│  │  ├─ watchdog‑protocol.md
│  │  └─ archivist‑protocol.md
│  ├─ deployment/
│  │  ├─ local‑vector‑model.md
│  │  └─ quick‑start.md
│  └─ references/
├─ memory/
│  └─ palace/
│     ├─ grand_hall/
│     ├─ chambers/
│     ├─ project_rooms/
│     ├─ dispatch_corridor/
│     ├─ reflection_wing/
│     │  ├─ telegrams/
│     │  ├─ principles/
│     │  ├─ prompt_kernels/
│     │  ├─ failure_patterns/
│     │  └─ thinking_paths/
│     └─ archive_basement/
├─ examples/
├─ templates/
└─ scripts/
```

## MVP
首个开源版本（2.0‑alpha）聚焦于：
- 带宫殿 2.0 房间约定的仓库骨架。
- 使用 `BAAI/bge‑m3` 和本地向量存储的 Watchdog 原型。
- Archivist 脱水流水线（电报生成与提升逻辑）。
- 上下文预算宪法与查询协议。
- 示例项目页和走廊卡片。

## 非目标
本项目至少在 MVP 阶段不是：
- 一个生产就绪的向量数据库服务。
- 一个完整的内存即服务平台。
- 一个聊天日志转储工具。
- 一个通用智能体框架。
- 一个精致的 SaaS 产品。

## Archivist（Alpha 阶段）
在 alpha 阶段，Archivist 角色临时分配给一个专用智能体（例如 04_Code）定期执行。它不参与前线工作；它只处理已在文件中验证的内容。

## 开源说明
本仓库刻意避免私有本地文件系统路径、私有身份、内部操作痕迹以及任何密钥。它保留结构、协议和示例，同时剥离环境特定细节。

## 路线图
1. 稳定宫殿 2.0 房间命名、路由和反思协议。
2. 提供开箱即用的 Watchdog 脚本，集成 `BAAI/bge‑m3` 与 ChromaDB。
3. 增加带类型边的语义检索（因果、矛盾、强化）。
4. 探索服务/插件集成（如 Slack、Discord、飞书）以实现自动记忆摄入。

## 许可证
MIT

## 署名
向量检索组件依赖 `BAAI/bge‑m3` 嵌入模型，该模型基于 Apache‑2.0 许可证。完整署名信息请见 `NOTICE` 文件。
