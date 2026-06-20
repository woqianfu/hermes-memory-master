# Memory OS 融合参考 — 7层记忆 · 精准注入 · Ground Truth

> **来源**: 公众号「arduino」— 《1045 Star、7 层记忆 OS：用 Qdrant+SQLite 给 Hermes Agent 装本地长期记忆》
> **GitHub**: https://github.com/ClaudioDrews/memory-os
> **收录时间**: 2026-06-20
> **融合目标**: memory-master v5.5.1+

---

## 一、Memory OS 核心架构（7层）

Memory OS 把"记住"拆成 7 层工程问题：

| 层 | 文件 | 用途 | 我们是否有 |
|:--:|------|------|:----------:|
| 1 会话层 | 聊天记录 | 原始对话 | ✅ session日志 |
| 2 事实层 | MEMORY.md | 结构化持久事实 | ✅ Layer② user memory |
| 3 用户层 | USER.md | 用户画像偏好 | ✅ 4型WF/EF/OB/MM |
| 4 创意层 | CREATIVE.md | 灵感/想法/构思 | ❌ 缺 |
| 5 灵魂层 | SOUL.md | Agent身份/人格+行为价值观 | ✅ 行为价值观 |
| 6 规则层 | rulebook.md | 硬规则/红线 | ✅ 在skills中 |
| 7 Ground Truth层 | — | 权威上下文指令 | ❌ 缺 |

### 最有价值的 3 个设计

---

## 二、融合点 1：精准上下文注入（Phase 12）

### Memory OS 的做法

`pre_llm_call` 从多个来源召回：

```
session DB → 最近 N 条相关会话
Qdrant   → 语义相似的记忆向量
SQLite   → 精确匹配的结构化事实
Wiki     → 项目知识文档
```

然后：
- **相关性阈值过滤** — 低分结果剔除，不喂
- **per-session 去重** — 同一段上下文不反复塞
- **跳过琐碎消息** — 社交性结尾（"好的""明白了""谢谢"）不存
- **Ground Truth 标记** — 注入时标记"此为权威上下文，优先使用"

### 记忆大师当前状态（v5.5.0）

- memory 工具全量注入（2,200字符硬上限）
- 无相关性过滤
- 无琐碎消息过滤
- 无显式权威指令

### 融合方案（Phase 12 — v5.5.1+）

```python
# 注入前过滤管道
def pre_inject_filter(candidates):
    # 1. 相关性阈值过滤 (score < 0.3 淘汰)
    candidates = [c for c in candidates if c['score'] >= 0.3]
    
    # 2. per-session 去重 (同一 session 不重复注入)
    seen = set()
    unique = []
    for c in candidates:
        if c['session_id'] not in seen:
            seen.add(c['session_id'])
            unique.append(c)
    
    # 3. 跳过琐碎消息
    TRIVIAL_PATTERNS = [r'^好的', r'^明白了', r'^谢谢', r'^嗯嗯', r'^ok', r'^👌']
    candidates = [c for c in candidates if not any(
        re.match(p, c['content']) for p in TRIVIAL_PATTERNS
    )]
    
    # 4. 标记 Ground Truth
    for c in candidates:
        c['authoritative'] = True
    
    return candidates
```

**实施步骤**：

| 步骤 | 内容 | 难度 |
|:----:|------|:----:|
| 1 | 定义琐碎消息过滤正则列表 | 🟢 易 |
| 2 | 自动学习 cron 写入时忽略琐碎消息 | 🟡 中 |
| 3 | 注入时添加相关性阈值 | 🟡 中 |
| 4 | 添加 per-session 去重 | 🟢 易 |

---

## 三、融合点 2：Ground Truth 层级（Phase 13）

### Memory OS 的做法

第 7 层 **Ground Truth hierarchy**：**被注入的记忆是权威上下文，Agent 应优先使用，而不是再去重复查询一遍。**

核心机制：
- 每次注入时标记这些记忆的优先级（authoritative > normal > fallback）
- 在 system prompt 中明确写入指令：**"以下记忆是你已经确认的事实，不需要再次询问用户或做额外搜索"**
- 解决 **memory-zero behavior**：看似有记忆，实际还在失忆式重复劳动

### 记忆大师当前状态（v5.5.0）

- 没有权威层级概念
- Agent 可能知道某条事实存在但不去用（每次重新问用户）

### 融合方案（Phase 13 — v5.5.1+）

```python
# System prompt 注入段
"""
## 📋 已确认记忆（权威上下文）
以下信息是你已经知道的事实。遇到相关问题时**直接使用这些信息**，
不要重复询问用户，不要额外搜索确认。

{injected_memories}

## 优先级规则
1. 注入记忆 > 搜索历史 > 模型推理
2. 有争议时以注入记忆为准
3. 如果用户明确纠正了某条记忆，立刻更新它
"""
```

**实施步骤**：

| 步骤 | 内容 | 难度 |
|:----:|------|:----:|
| 1 | 在 retrieval 结果中标记权威层级 | 🟢 易 |
| 2 | 更新检索路径规则添加"已确认记忆优先" | 🟢 易 |
| 3 | 在每日审计中标记"已确认"条目 | 🟢 易 |

---

## 四、融合点 3：7层结构对齐（Phase 14）

### Memory OS 的 7 层 vs 我们的 3 层

| Memory OS 层 | 我们已有 | 缺口 |
|:------------:|:--------:|:----:|
| 1 会话层 | ✅ session_search | — |
| 2 事实层 | ✅ MEMORY.md (WF/EF) | — |
| 3 用户层 | ✅ USER.md (OB/MM) | — |
| 4 创意层 | ❌ 无专门位置 | 需要 CREATIVE.md |
| 5 灵魂层 | ✅ SOUL.md (行为价值观) | 永久·每次会话注入 |
| 6 规则层 | ✅ skills/ | — |
| 7 Ground Truth | ❌ 无 | Phase 13 |

**创意层（CREATIVE.md）** — 目前我们没有专门存灵感和构思的地方：
- 用户临时冒出的想法 → 现在混在 MEMORY.md 或 Layer③ 工作区
- 应该有个独立区域存"创意草稿"，TTL短、不参与持久记忆审计
- 可以放在 `~/.hermes/memories/CREATIVE.md`，自动学习 cron 检测创意类内容写入这里而不是 MEMORY.md

**对齐方案**：

```yaml
# 对齐映射表
记忆大师 → Memory OS
━━━━━━━━━━━━━━━━━━━━━━━━━
Layer① 云端自动学习 → Layer 1/2 会话+事实
Layer② 用户持久规则 → Layer 3/5/6 用户+灵魂+规则
Layer③ 工作区项目隔离 → —（Memory OS没有等价物）
—                     → Layer 4 创意层 💡 NEW
Phase 13 Ground Truth → Layer 7 Ground Truth
```

---

## 五、与已有融合的排重

| 已有融合 | Memory OS 对应 | 是否重复 |
|---------|----------------|:--------:|
| Hindsight 4型分类 | 事实/用户层 | ❌ 互补 |
| MemOS 智能去重 | per-session 去重 | ⚠️ 部分重叠。我们去重防重复写入，Memory OS 去重防止重复注入，并行运行 |
| Supermemory 时间线 | 事实层追踪 | ❌ 互补 |
| RRF 3通道 | Qdrant 语义搜索 | ❌ 方案相似，不需同质化 |

---

## 六、实施优先级

| 优先级 | 融合点 | 预期效果 | 工作量 |
|:------:|--------|---------|:------:|
| 🔴 P0 | 跳过琐碎消息 | 减少30%+无用记忆写入，节省容量 | 半天 |
| 🔴 P0 | Ground Truth 层级 | 消除"有记忆但不用"的重复劳动 | 半天 |
| 🟡 P1 | 相关性阈值过滤 | 注入质量提升，token更省 | 1天 |
| 🟢 P2 | 创意层 CREATIVE.md | 灵感/构思有专用空间 | 0.5天 |
