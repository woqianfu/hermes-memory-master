# Neural Memory MCP 对接参考

## 状态: ✅ 已连接 (v4.58.0, 63 tools)

## 概述
Neural Memory 是一个 MCP 记忆引擎，通过 `python3 -m neural_memory.mcp` 提供服务。
安装于系统 Python 3.12 site-packages，通过 Hermes MCP 客户端连接。

## 连接状态
```bash
hermes mcp test neural-memory
# →  ✓ Connected (310ms)
# →  ✓ Tools discovered: 63
```

## 工具分类

| 类别 | 工具 | 说明 |
|------|------|------|
| 记忆存取 | nmem_remember/recall/context/auto | 核心CRUD |
| 项目记忆 | nmem_eternal/recap/session | 项目级持久上下文 |
| 知识图谱 | nmem_explain/causal/hypothesize/evidence | 因果+假设 |
| 代码索引 | nmem_index/train_db | 代码库索引 |
| 质量 | nmem_health/conflicts/stats/evolution | 健康检查 |
| 复习 | nmem_review/forget/consolidate | 遗忘曲线+压缩 |
| 版本控制 | nmem_version/rollback | 快照回滚 |
| 同步 | nmem_sync/export/import/transplant | 跨设备 |
| 预算 | nmem_budget/cache/offload | Token管理 |

## 记忆大师 vs Neural Memory

| 维度 | 记忆大师 | Neural Memory |
|------|---------|---------------|
| 层级 | Layer 2 (用户/云端记忆) | Layer 4 (外部存储引擎) |
| 容量 | ~2,200 chars | 无上限 |
| 检索 | 3通道RRF(向量+关键词+实体) | Spreading Activation |
| 实体 | Entity通道(v2) | 知识图谱+因果链 |
| 维护 | cron + 脚本 | 自动consolidation |
| 同步 | cross-profile 7个 | 云端hub同步 |
| 集成 | Hermes 原生 memory 工具 | MCP 协议 |

## 对接策略

### 1. 热冷分离
```
Layer 1 (自动学习) → 高频事实 → memory 工具 (2,200 chars)
Layer 4 (Neural)  → 海量长期 → nmem_remember (无上限)
                   ↓
自动学习 cron 在写入 memory 的同时，也通过 sync-to-neural-memory.py 备份到 Neural Memory
```

### 2. 检索增强
```
用户提问
  ↓
mm search "查询" (记忆大师·3通道RRF) → 快速命中?
  ├─ 是 → 直接回复
  └─ 否 → nmem_recall "查询" (Neural·spreading activation)
         → 结果整合回复
```

### 3. 项目记忆
```
short-drama-master的项目决策 → nmem_eternal 保存（跨session不丢）
                   → 每次开工 → nmem_recap 恢复上下文
                   → WORKSPACE.md 作为快速摘要（Layer 3）
```

---

## 导入实操: 记忆大师 → Neural Memory

### 有效记忆类型 (必须用以下之一)

| 类型 | 含义 | WF/EF/OB/MM 映射 |
|------|------|-----------------|
| `fact` | 客观事实 | WF |
| `decision` | 决策 | EF/OB |
| `preference` | 偏好 | OB |
| `insight` | 洞察 | OB |
| `instruction` | 指令 | MM |
| `workflow` | 工作流 | EF |
| `reference` | 参考 | WF |
| `boundary` | 边界/红线 | MM |
| `context` | 上下文 | EF |
| `error` | 错误记录 | EF |
| `todo` | 待办 | — |

### 类型映射 (记忆大师 → Neural Memory)

```
[WF] → fact      (世界事实→事实)
[EF] → workflow  (经验事实→工作流)
[OB] → insight   (观察→洞察)
[MM] → boundary  (心智模型→边界)

注意: observation / rule / experience 不是有效类型，用上述映射。
```

### 一次导入 (已执行)
```bash
# 全量导入 (22条, 约60秒)
python3 ~/.hermes/scripts/import-to-neural-memory.py

# 验证
python3 ~/.hermes/scripts/diagnose-search.py
```

### 持续同步 (cron, 已配置)
```bash
# 手动同步
mm nm-sync                    # 仅新/changed条目
mm nm-sync --force             # 全量重导
mm nm-sync --check             # 仅检查差异

# 脚本: ~/.hermes/scripts/sync-to-neural-memory.py
# 原理: 读取 MEMORY.md/USER.md → 解析条目 → 比对哈希缓存(.neural-sync-cache.json)
#       → 仅导入新/修改的条目 → 更新缓存
# cron: 每4h过15分 (job=[JOB_NMSYNC], no_agent=true)
# 无新条目时静默退出, 不产生通知
```

### 缓存去重机制
```
.neural-sync-cache.json
├── synced_hashes: [md5(content) ...]  # 已同步条目的内容哈希
└── last_sync: ISO timestamp            # 上次同步时间

首次: 22条全量导入 → 22个哈希写入缓存
后续: 只同步哈希不在缓存中的条目
--force: 忽略缓存, 全量重导(由Neural Memory自身去重)
```

### 已知限制
- **`nmem_recall` 中文查询不敏感**: 英文词(WeChat/Hermes)能命中中文内容，但纯中文多词查询("微信 记忆 同步")可能无结果
- **变通方案**: 用 `nmem_context` 浏览最近记忆(正常显示中文)，或用 session_search / mm search 补充
- 这是 neural_memory 搜索引擎的中文分词限制，非导入问题

### 脚本清单

| 脚本 | 说明 |
|------|------|
| `~/.hermes/scripts/import-to-neural-memory.py` | 一次导入(解析→nmem_remember) |
| `~/.hermes/scripts/sync-to-neural-memory.py` | 持续同步(缓存去重+cron) |
| `~/.hermes/scripts/diagnose-search.py` | 检索效果诊断 |
| `~/.hermes/scripts/nm-test.py` | MCP协议连通性测试 |
| `~/.hermes/scripts/verify-neural-memory.py` | 导入验证脚本 |
| `~/.hermes/memories/.neural-sync-cache.json` | 同步缓存(自动维护) |

## 注意事项
- Python 3.12 安装，Hermes venv (3.11) 不能直接 import
- MCP 通过 stdio 通信，依赖 `command: python3` 为系统 python
- Gateway 重启后自动重连（有3次重试）
- 首次连接可能需要 ~300ms（热启动后更快）
- `mm nm-sync` 已加入 `~/.local/bin/` PATH
