# 🧠 Hermes Memory Master

> **7-Layer Autonomous Memory Architecture for Hermes Agent**
> **Hermes Agent 多层自进化记忆系统**

---

## Overview 概览

Hermes Memory Master is a **complete memory management system** for [Hermes Agent](https://github.com/NousResearch/hermes-agent). It transforms Hermes from a session-memory-only agent into a **persistent, self-learning, cross-session intelligent assistant** that truly "remembers" you.

Hermes 记忆大师是一个为 Hermes Agent 打造的**完整记忆系统**。它将 Hermes 从"仅会话记忆"升级为**持久化、自学习、跨会话的智能助手**，真正"记住"用户。

### What you get 你能得到什么

| Feature | English | 中文 |
|---------|---------|------|
| 🤖 Auto-Learning | Scans every conversation every 4h, extracts preferences/rules/facts | 每4h自动扫描对话，提取偏好/规则/事实 |
| 🔍 Hybrid Search | Vector + keyword + entity 3-channel RRF search | 向量+关键词+实体 3通道混合检索 |
| 🏗️ 7-Layer Architecture | Session→Facts→User→Creative→Soul→Rules→Ground Truth | 会话→事实→用户→创意→灵魂→规则→Ground Truth |
| 🧬 Ground Truth Priority | authoritative > normal > fallback tier marking | 权威/普通/后备三级优先级标记 |
| 🧠 Neural Memory | External MCP engine with 63 tools for graph/causal search | 63工具外部知识图谱引擎 |
| 🔄 Cross-Profile Sync | Sync memory across 7+ Hermes profiles | 跨7个+ Hermes 实例同步记忆 |
| 📊 Dashboard | HTML dashboard with 7 browse-only tabs | HTML 仪表盘，7个浏览选项卡 |
| ⏰ Cron Guardians | 6 cron jobs: auto-learn + forget + dashboard + audit + NM sync + learn | 6条守护进程 |
| 🚒 Circuit Breaker | Auto-clean, merge, alert when memory is full | 记忆满时自动清洗/合并/告警 |
| 🗑️ Auto-Forget | TTL-based expiry + superseded cleanup (no_agent, 20:30 daily) | TTL到期+superseded自动清理(每日20:30) |
| 📦 Export/Import | Full memory backup with version diff | 完整记忆打包+版本差异 |
| 🔬 A/B Testing | Compare search quality with/without memory | 对比有/无记忆的检索质量 |
| 🌐 Bilingual | Chinese-English keyword anchoring for cross-language search | 中英文锚点支持跨语言检索 |
| 🧪 Self-Test | `mm self-test` — 6-dimension health check suite | 自评测套件：6维健康检测 |

---

## 7-Layer Architecture 七层架构

```
Layer ①  Auto-Learning Cloud     every 4h → auto-learn-memory.py
Layer ②  Persistent User Memory  2,200 chars → MEMORY.md / USER.md
Layer ③  Project Workspace       per-project → WORKSPACE.md
Layer ④  Creative Layer          TTL 7d → CREATIVE.md
Layer ⑤  Soul Layer              permanent → SOUL.md
Layer ⑥  Rules Layer             per-skill → SKILL.md + AGENTS.md
Layer ⑦  Ground Truth Priority   ⭐ > ✓ > ⚪ → injection-filter.py
```

---

## Scripts 配套脚本 (25 total)

| Category | Script | Lines | CLI |
|:---------|:-------|:-----:|:----|
| **Core Memory** | | | |
| | `auto-learn-memory.py` | 387 | `mm learn` |
| | `memory-vector.py` | 466 | `mm search / index / status / setup` |
| | `memory-pre-retrieval.py` | 215 | manual call with query |
| | `memory-injection-filter.py` | 128 | stdin pipe |
| | `memory-auto-forget.py` | 118 | `mm forget [--dry-run]` |
| | `memory-changelog.py` | 126 | `mm changelog` |
| | `memory-conflict.py` | 168 | `mm conflict` |
| | `memory-diff.py` | 107 | `mm diff` |
| | `memory-tag-validator.py` | 97 | `mm validate` |
| **Ops Tools** | | | |
| | `mm` | 141 | unified entry for 23 subcommands |
| | `memory-dashboard.py` | 389 | `mm dashboard` |
| | `memory-to-skill.py` | 149 | `mm skill` |
| | `cross-profile-sync.py` | 185 | `mm sync` |
| | `memory-migrate.py` | 101 | `mm export / import` |
| | `memory-editor.py` | 183 | `mm editor` |
| | `memory-forecast.py` | 131 | `mm forecast` |
| | `memory-workspace.py` | 42 | `mm workspace [name]` |
| | `memory-self-test.py` | 145 | `mm self-test [--quick]` |
| **Quality** | | | |
| | `memory-abtest.py` | 153 | `mm abtest <query>` |
| | `memory-review.py` | 156 | `mm review` |
| | `memory-bilingual.py` | 131 | `mm bilingual` |
| **External** | | | |
| | `sync-to-neural-memory.py` | 319 | `mm nm-sync` |
| | `import-to-neural-memory.py` | 143 | manual |
| | `verify-neural-memory.py` | 46 | manual |

---

## Quick Start

```bash
# Clone
git clone https://github.com/woqianfu/hermes-memory-master.git
cd hermes-memory-master

# Copy scripts to your Hermes scripts directory
cp -r scripts/* ~/.hermes/scripts/
cp SKILL.md ~/.hermes/skills/hermes/memory-master/

# Run health check
python3 ~/.hermes/scripts/mm self-test

# Generate your dashboard
python3 ~/.hermes/scripts/mm dashboard

# Set up auto-learn cron (every 4h)
hermes cron create --schedule "0 */4 * * *" \
  --prompt "run: python3 ~/.hermes/scripts/auto-learn-memory.py" \
  --deliver local

# Run auto-forget daily (20:30)
hermes cron create --schedule "30 20 * * *" \
  --script ~/.hermes/scripts/memory-auto-forget.py \
  --no-agent
```

---

## Requirements

- Python 3.12+ (chromadb dependency)
- Hermes Agent (any version that supports skills)
- Optional: Neural Memory MCP for graph search

```
pip install chromadb sentence-transformers
hermes mcp install neural-memory  # optional, for graph search
```

---

## File Structure

```
hermes-memory-master/
├── SKILL.md                    # Main skill (7-layer architecture, 1,371 lines)
├── .gitignore
├── README.md
├── HIGHLIGHTS.md
├── assets/
│   └── dashboard.html          # Generated HTML dashboard (run mm dashboard)
└── references/
    ├── memory-drift-recovery.md
    ├── memory-os-integration.md
    ├── neural-memory-chinese-search.md
    ├── neural-memory-integration.md
    ├── placeholder-to-real-layer.md
    ├── supermemory-integration.md
    ├── ten-round-self-inspection-2026-06-21.md
    ├── web-learning-synthesis.md
    └── web-to-code-fusion.md
```

---

## Key Design Decisions 设计决策

| Decision | Rationale |
|:---------|:----------|
| Zero Docker | All scripts are pure Python; no Docker dependency |
| chromadb over Qdrant | Simpler local deployment, zero external services |
| LLM-driven auto-learn | The LLM judges what to write; not rule-based extraction |
| 6-month principle > 7-day principle | When conflicting, long-term value trumps short-term freshness |
| no_agent for auto-forget | Pure TTL scan — no LLM cost, runs 20:30 daily |
| SOUL.md as behaviour layer | Values > rules: "remember correctly > remember more" |

---

## License

MIT
