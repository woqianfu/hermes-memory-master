# 全网学习参考资料
# ██████████████████████████████████████████████████████████
# 来源: 记忆大师 20轮深度自检 (2026-06-18)
# 融合: memory-vector.py v2 (Entity), auto-learn-memory.py v4 (多信号)
# ██████████████████████████████████████████████████████████

## 1. Vectorize.io — Hermes 4层记忆架构 (2026)
来源: https://vectorize.io/blog/hermes-memory-architecture
核心发现: Hermes 原生 4 层 vs 记忆大师 3 层的映射关系

### 关键要点
- Layer1 (Prompt Memory): MEMORY.md/USER.md → 我们的 Layer2 用户记忆
- Layer2 (Session Archive): state.db → 我们保持原生 session_search
- Layer3 (Skills): 程序化记忆 → 我们的 Phase2 记忆→技能固化
- Layer4 (External Provider): 7个可选提供者 → 新增 setup 子命令桥接

### 落地产物
- SKILL.md 生态对齐表 (7 提供者 + 4 层映射)
- mm setup 命令检测外部提供者

## 2. Hindsight Blog — LongMemEval 94.6% (2026)
来源: https://hindsight.ai/blog/2026-agent-memory
核心发现: Hindsight 在 LongMemEval 基准达到 94.6%，最高分

### 关键要点
- RRF k=60 是 Hindsight 经验值，我们已采用
- Entity 链接是 Hindsight 核心优势，我们通过 extract_entities() + 第3通道对齐
- Reflect 阶段(写入前反思)已融入 auto-learn-memory.py

### 落地产物
- memory-vector.py RRF 公式 v2: 向量+关键词+实体
- 实体权重: PERSON(1.5) > SKILL(1.4) > PROJECT(1.3) > PLATFORM(1.2) > TECH(1.0) > VALUE(0.8)

## 3. Mem0 — State of Agent Memory 2026
来源: https://www.mem0.ai/blog/state-of-agent-memory-2026
核心发现: 单次 ADD 提取模式 + 多信号检索

### 关键要点
- 最佳实践是"单次提取所有发现"，而非多次 session_search
- 多信号索引: 情绪(affective)、时间(temporal)、关系(relational)
- LongMemEval 67.6% (Mem0 得分低于 Hindsight)

### 落地产物
- cron prompt 改为一次性提取所有信号(减少 token)
- auto-learn-memory.py v4: 信号热力图 + P0/P1/P2 优先级
- 时间衰减: 4h=1.0, 8h=0.8, 24h=0.4, >48h=0.1

## 4. Reddit r/hermesagent — Supermemory 预配置
核心发现: Hermes 最新版已内置 Supermemory（免费每月 1 万次调用）

### 关键要点
- `hermes memory setup` CLI 命令
- Supermemory 无需配置即可使用
- 适合简单记忆场景，不适合复杂实体推理

### 落地产物
- mm setup 检测 `hermes memory setup` 是否可用
- 生态表中 Supermemory 标为"零配置"

## 5. 生态对齐: Hermes 7个原生记忆提供者

| 提供者 | 存储 | 成本 | LongMemEval | 优势 | 适合场景 |
|--------|------|------|-------------|------|---------|
| Hindsight | 本地/云端 | 免费(本地) | 94.6% | 知识图谱+实体+Reflect | 复杂推理 |
| Honcho | 云端 | 按量 | — | 辩证用户建模 | 多面性格 |
| Mem0 | 云端 | 免费+付费 | 67.6% | 服务器端LLM提取 | 成熟稳定 |
| OpenViking | 自托管 | 免费 | — | 文件层级+分层加载 | 文件库 |
| Holographic | 本地SQLite | 免费 | — | 零依赖(纯SQLite+HRR) | 最小化 |
| RetainDB | 云端 | 付费 | — | 混合搜索(向量+BM25+重排序) | 企业级 |
| ByteRover | 本地/云端 | 免费+付费 | — | 预压缩提取+知识树 | 大数据 |
| Supermemory | 云端 | 免费(10K/月) | — | Hermes预配置 | 简单易用 |

## 6. 记忆大师 vs Hermes 原生 4 层

| Hermes 原生 | 记忆大师对应 | 我们的增强 |
|------------|-------------|-----------|
| Layer1: Prompt Memory (MEMORY.md/USER.md) | Layer② 用户记忆 | 自动学习+去重+Reflect |
| Layer2: Session Archive (state.db) | session_search | 向量检索+3通道RRF |
| Layer3: Skills (procedural memory) | Phase2 固化 | 自动固化检测+发散方向 |
| Layer4: External Provider | 可选: Hindsight/Mem0等 | setup检测+桥接表 |

## 7. 记忆大师 RRF 公式演进

v1: TF-IDF + 向量 + 时间衰减
  公式: 1/(60+rank) + (match_ratio*0.5) + time_decay

v2: TF-IDF + 向量 + Entity + 时间衰减 (当前)
  公式: 1/(60+rank) + (match_ratio*0.5) + (entity_score*0.8) + time_decay
  Entity权重: PERSON(1.5) > SKILL(1.4) > PROJECT(1.3) > PLATFORM(1.2) > TECH(1.0) > VALUE(0.8)

## 8. 多信号检索权重公式 (auto-learn v4)

signal_score = time_weight * 0.4 + emotion_strength * 0.3 + 
               topic_coverage * 0.2 + relationship_strength * 0.1

P0 🔴: signal >= 0.7 (情绪强烈+时效高)
P1 🟡: signal >= 0.4 (常规信号)
P2 ⚪: signal < 0.4 (低优先级)
