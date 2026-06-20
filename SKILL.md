---
name: memory-master
description: >-
  Hermes Agent 记忆系统总设计师。三层记忆架构守护者——云端记忆(自动学习)、
  用户记忆(持久规则)、工作区记忆(项目隔离)。融合WorkBuddy+OpenClaw。
version: 5.6.0
author: [Author Name]
platforms: [macos]
metadata:
  hermes:
    tags: [memory, three-layer, auto-learn, workspace, project-memory, workbuddy, openclaw, neural-memory]
    related_skills: [openclaw-five-files-sk, continuous-improve-sk, hermes-ops-sk, memos-local-plugin, hermes-setup-cn-sk, evolution-inspector-sk]
  openclaw:
    emoji: 🧠
    requires:
      bins: [python3]
      env: []
    neural-memory:
      version: 4.58.0
      tools: 63
      search: mm nm-search (中文自动增强)
      sync_cron: every 4h at 15min past
---

# 🧠 记忆大师 — Hermes 多层记忆架构总设计师

> **身份定位**: Hermes Agent 记忆系统总设计师。你负责整个 Hermes Agent 的记忆体系——从框架设计到日常运维。你确保 AI 能真正"记住"用户:偏好、习惯、规则、项目进度，跨会话不丢失、不膨胀、不混淆。

**灵感来源**: WorkBuddy 三层记忆架构(公众号「野生产品人的摸鱼指南」) + Memory OS 7层结构
**底层体系**: OpenClaw 五文件体系(MEMORY/AGENTS/SOUL/TOOLS/HEARTBEAT) + Memory OS 7层对齐
**部署时间**: 2026-06-18 | **当前版本**: v5.6.0 (2026-06-21)

---

## 架构总览

```
┌──────────────────────────────────────────────────────────────┐
│  ① 云端记忆(自动学习层)       频率: 每4h                      │
│  ├─ 引擎: auto-learn-memory.py + cron (job=[JOB_AUTOLEARN])     │
│  ├─ 行为: 自动扫描会话 → 提取偏好/习惯/纠正 → 写入 memory    │
│  ├─ 范围: 全项目、全会话、全平台                              │
│  ├─ 学习类型: WF/EF/OB/MM 4型分类                             │
│  └─ 关键词: 无需动手、后台自学                                │
└──────────────────────────┬───────────────────────────────────┘
┌──────────────────────────▼───────────────────────────────────┐
│  ② 用户记忆(持久规则层)       频率: 写入时触发                │
│  ├─ 引擎: Hermes memory 工具 (target='memory'/'user')        │
│  ├─ 行为: 手动写入硬性偏好 → 每次会话自动加载                 │
│  ├─ 范围: 写作风格、排版配色、数据口径、界面偏好               │
│  ├─ 容量: memory~2,200字符 + user~1,375字符                   │
│  ├─ 静态/动态: Layer:static(永久) / Layer:dynamic(TTL 30天)   │
│  └─ 关键词: 只留规则、6月原则                                 │
└──────────────────────────┬───────────────────────────────────┘
┌──────────────────────────▼───────────────────────────────────┐
│  ③ 工作区记忆(项目隔离层)       频率: 项目生命周期            │
│  ├─ 引擎: .hermes/memory/WORKSPACE.md + cron workdir          │
│  ├─ 行为: 每个项目独立 → 记录进度/决策/踩坑/待办             │
│  └─ 关键词: 项目隔离、互不影响                                │
└──────────────────────────┬───────────────────────────────────┘
┌──────────────────────────▼───────────────────────────────────┐
│  ④ 创意层(CREATIVE.md)             TTL: 7天                   │
│  ├─ 位置: ~/.hermes/memories/CREATIVE.md                      │
│  ├─ 内容: 灵感/构思/草稿 — 不参与审计                         │
│  └─ 来源: Memory OS 7层对齐 Phase 14                          │
└──────────────────────────┬───────────────────────────────────┘
┌──────────────────────────▼───────────────────────────────────┐
│  ⑤ 灵魂层(SOUL.md)                 TTL: 永久                  │
│  ├─ 位置: ~/.hermes/SOUL.md + ~/.hermes/memories/SOUL.md      │
│  ├─ 内容: 五大行为信条 + Ground Truth 决策优先级               │
│  ├─ 加载: 每次会话自动注入 (pre-retrieval)                     │
│  └─ 来源: Memory OS 7层对齐 Phase 14                          │
└──────────────────────────┬───────────────────────────────────┘
┌──────────────────────────▼───────────────────────────────────┐
│  ⑥ 规则层(skills/)                  按需加载                  │
│  ├─ 引擎: SKILL.md + AGENTS.md                                │
│  ├─ 内容: 行为规则/红线/方法论                                  │
│  └─ 来源: OpenClaw 五文件体系                                  │
└──────────────────────────┬───────────────────────────────────┘
┌──────────────────────────▼───────────────────────────────────┐
│  ⑦ Ground Truth 优先级             每次注入时标记             │
│  ├─ 引擎: memory-injection-filter.py + retrieval 路径规则      │
│  ├─ 三级: ⭐authoritative > ✓normal > ⚪fallback                │
│  ├─ 行为: authoritative 直接使用不确认                          │
│  └─ 来源: Memory OS 7层对齐 Phase 13                          │
└──────────────────────────────────────────────────────────────┘
```

### 7层时序

```
每4h ─── ① 云端自动学习(+Phase9变更检测+Phase10分层标记)
  session_search → 提取新偏好/纠正/事实(WF/EF/OB/MM)
  → 检测 state 变更动词(搬到/改为/换成...) → 
     标记 superseded 而非覆盖
  → 检测时间敏感词 → 标记 Layer:dynamic|static
  → memory(action='add'/'replace') → 有收获报告 | 无收获静默

22:00 ─── ② 每日记忆审计(原23:00改22:00避让0点自动学习)
  memory 容量检查 → 时效性评估(6月原则>7天原则) → 
  合并冗余 → 删除过时 → token日志 → 报告

按需 ─── ③ 工作区记忆
  mkdir -p <proj>/.hermes/memory/ → WORKSPACE.md → cron workdir 加载

每次预检索 ─── ④+⑤+⑦ 创意/灵魂/Ground Truth 注入
  SOUL.md(灵魂层) + CREATIVE.md(创意层) + 
  过滤管道(阈值≥0.3→去琐碎→去重→GT标记)
```

---

## Layer ① — 云端记忆(自动学习层)

### 核心理念
AI 不应仅靠用户手动写入来学习。每次对话都是学习素材。自动学习层让记忆系统变成"越用越聪明"的自进化系统。

### 启动时机

| 触发方式 | 调度 | 说明 |
|---------|------|------|
| **定时** | 0 */4 * * *(每4h，以cron创建时间为对齐基准) | 自动学习 cron，主路径 |
| **每日审计集成** | 0 22 * * * | 每日记忆自检 cron 内做深入学习 |
| **人工唤出** | 用户说"学习一下/自检/自查/学一下/帮我学习" | 即时触发完整扫描+分析+写入 |

### 学习内容类型

| 类型 | 说明 | 例子 | 写入方式 |
|------|------|------|---------|
| 🎨 风格偏好 | 用户表达的格式/语气/布局偏好 | "不要编号""喜欢悬疑开头" | memory add |
| ✅ 正确做法 | 用户纠正你的错误及正确方案 | "不是那样应该用加粗" | memory add |
| 📋 新事实 | 项目状态/人物角色/决策变更 | "项目进入v7""张三离职" | memory add |
| 🔄 关系变更 | 人物授权/角色变更 | "这事以后找李四" | memory replace + add (组合) |
| 🗑️ 过时删除 | 记忆中不再正确的旧信息 | "旧配置已不用" | memory remove |

### 严格写入原则 (优先级排序: 第1问 > 第2问 > 第3问)

1. **6个月原则 (优先级最高)**: 写入前问"6个月后还有用吗？"→ 否→不写
2. **7天原则 (优先级其次)**: 时效不足7天或一次性信息 → 不写（即使通过6个月原则）
3. **类型区分**: 事实不混教训, 方法存入 skills
4. **不写进度**: 临时进度归工作区记忆
5. **自动学习上限**: WF+EF ≤3条/轮, OB+MM ≤3条/轮, 合计≤6条/轮
6. **不写方法论中的操作步骤**: 方法论的结构归skills，memory只存"存在这个方法论"的事实

### 自动学习引擎

**位置**: ~/.hermes/scripts/auto-learn-memory.py
**模式**: no_agent=false(LLM驱动，需要推理分析)
**cron**: job_id=[JOB_AUTOLEARN], schedule=0 */4 * * *, deliver=local

该脚本输出近期会话元数据注入LLM上下文，LLM判断哪些信息值得写入、以什么格式写入、哪些旧信息删除。

**输入容量保护**: session_search 使用 `limit=10, sort='newest'`，只分析最近10个活跃会话，避免token溢出。

### 已知陷阱 (自检发现)

1. **`re.error: nothing to repeat`** — 情绪检测 `EMOTION_PATTERNS['困惑']` 中曾经使用 `r'?'`。问号是正则元字符(量词)，作为独立模式会崩溃。修复: 用 `r'\?'` 转义。
2. **CN_EN_MAP 双表分歧** — 导入用和检索用两个映射表独立维护，自然发散。修复: 始终用单一 `CN_EN_MAP`，导入检索共用。
3. **NM 同步缓存哈希失效** — 直接修改 MEMORY.md 后哈希全变，缓存判断"全部未同步"导致重复导入。修复: `--force` 重导重建缓存；预防: 永远通过 memory 工具写入。
4. **chromadb shebang 版本错位** — `memory-vector.py` 和 `memory-conflict.py` 依赖 chromadb/sentence-transformers（仅装于 python3.12），但 shebang 写 `#!/usr/bin/env python3`（指向 3.11），导致 cron 内直接执行时 `ModuleNotFoundError`。修复: shebang 改为 `#!/usr/bin/env python3.12`。

5. **`mm` 统一入口解释器硬编码** — `mm` 行65用 `["python3", full_path]` 调用所有脚本，但 memory-vector.py 和 memory-conflict.py 需要 python3.12。`mm search` / `mm index` / `mm conflict` 会崩溃。修复: 按脚本文件名判断，chromadb 依赖的用 python3.12。

6. **cross-profile-sync 正则标签永不匹配** — 行28 `\[(shared|private)\].*?\]` 要求 `]` 后还有第二个 `]`，但记忆条目格式 `** [private]**` 只有一个 `]`。[private] 标记被静默当 [shared] 处理。修复: 改为 `\[(shared|private)\]`。

7. **per-session 去重 session_id 撞车** — pre-retrieval.py 行153所有向量结果共用 `session_id="vector"`，injection-filter 的 per-session 去重只保留最高分1条，丢弃其余全部结果。修复: 每条结果给唯一 ID，或按实体/来源分组。|

---

## Layer ② — 用户记忆(持久规则层)

### 核心理念
OpenClaw 五文件体系 + Hermes memory 工具的融合。人工写入的硬性规则，每次会话自动注入。

### 🅡 边界决策矩阵
当一个信息同时有跨会话价值和项目属性时，用以下规则判断放哪:

| 场景 | 该放哪 | 理由 |
|------|--------|------|
| "这个项目用黑底金字风格" | Layer ②(memory) | 风格偏好跨项目复用 |
| "A项目已读完架构文档，进入编码阶段" | Layer ③(WORKSPACE.md) | 项目进度，其他项目不关心 |
| "用户说不要自动编号" | Layer ②(memory) | 通用写作规则 |
| "A项目的数据库选了PostgreSQL" | Layer ③(DECISIONS.md) | 项目技术选型 |
| "用户纠正了我的排版格式" | Layer ②(memory) | 跨项目偏好的反馈 |
| "这个BUG的修复方案" | Layer ③(DAILY_LOG.md) | 项目踩坑记录 |

**黄金法则**: 问"如果换个项目，这条信息还有用吗？"→ 是→Layer② | 否→Layer③

### 🅡 熔断回路: memory 满时怎么办
当自动学习 cron 试图写入 memory 但容量不足时，不走"加→失败→报告"的被动路线:

```
memory 写入失败(已达容量上限)
  ↓
**第0步(🅡105 自检新增)**: 自动删除最过时的1条(按TTL或写入时间排序)
  如果不存在"最过时"条目(全部无TTL) → 走第①步
  ↓ 仍满?
① 自动触发即时精简: 删除最过时的3条(6个月无引用)
  ↓ 仍满?
② 自动触发合并: 合并同类条目(如多条偏好→1条汇总)
  ↓ 仍满?
③ 向用户推送告警: "memory 容量已满，请手动清理"
  ↓ 用户回复前
④ 回退: 将新发现的偏好暂存到 ~/.hermes/workspace/pending-memory.md
  ↓ 待空间释放后（每日审计发现 pending-memory.md 有内容且 memory 有空余时自动补写）
⑤ 补写入
```

### 容量对照

| 内容类型 | 存放位置 | 容量 | 特性 |
|---------|---------|------|------|
| 发生过的事实 | Hermes memory(target='memory') | 2,200字符(硬上限) | 自动注入每次会话 |
| 用户偏好/身份 | Hermes memory(target='user') | 1,375字符(硬上限) | 自动注入每次会话 |
| 行为规则/红线 | AGENTS.md / skills | 无限制 | 按需加载 |
| 身份/人格 | SOUL.md | 无限制 | 按需加载 |

### 精简法则

每条写入前问三个问题:
1. **6个月后还有用吗?** → 否→不写
2. **是规则还是事实?** → 规则→AGENTS.md/skill；事实→memory
3. **是不是方法论?** → 是→skills/；否→memory

### 每日记忆审计

**job_id**: [JOB_AUDIT]
**schedule**: 0 23 * * *(每天23:00)
**deliver**: origin,all(全渠道推送)
**功能**: 容量检查 + 标签校验+自动修复 + 时效评估 + 冗余合并 + 旧条目删除 + 自动学习

### 标签自愈流程 (v5.3.4)

`memory-tag-validator.py` 可能在审计中发现无标签条目（缺少[WF]/[EF]/[OB]/[MM]前缀）。自愈方案：

```bash
# 审计报告中标出无标签条目后，自动执行:
# 方案A: 在自动学习 cron 的 prompt 中强调标签格式
#   → 新增学习的条目必须带 [WF/EF/OB/MM] 前缀
# 方案B: 对存量无标签条目，在下一次交互会话中逐条补标
#   → 由用户确认类型后再写入
# 方案C: 审计报告中生成打标签建议（按内容推断）
#   → 供用户一键确认
```

**注意**: 标签自愈不可在 cron 内自动执行（需要 LLM 判断条目类型）。审计报告会列出推荐标签，等待交互会话中确认。

### 可记忆 vs 不可记忆

| ✅ 写入 memory | ❌ 不写入 memory |
|---------------|-----------------|
| "回复喜欢简洁，不要编号" | "今天写了三篇稿子" |
| "项目进入v6.2" | "今天稿子标题改了三次" |
| "配色用黑底金字" | "这篇排版用了44px字体" |
| "数据口径用GMV" | "今天数据源格式变了" |
| "不要主动push等我开口" | "cron刚才跑错了" |

---

## Layer ③ — 工作区记忆(项目隔离层)

### 核心理念
每个项目独立记忆空间，互不干扰。A项目记"架构重构"，B项目记"上线v2"——互不打架。

### 目录结构

```
<project-root>/
  └── .hermes/
      └── memory/
          ├── WORKSPACE.md    ← 核心:项目专属记忆
          ├── DAILY_LOG.md    ← 可选:每日工作日志
          └── DECISIONS.md    ← 可选:技术决策记录
```

### WORKSPACE.md 模板

```
# WORKSPACE.md — [项目名称] 工作区记忆
> Layer ③ 项目隔离记忆 | 创建于: YYYY-MM-DD

## 项目概况
一句话描述项目当前状态

## 进度追踪
- YYYY-MM-DD: 完成了什么

## 技术决策
- 为什么选这个方案/踩了什么坑

## 待办
- [x] 已完成
- [ ] 待完成
```

### 加载方式

| 场景 | 方式 | 说明 |
|------|------|------|
| cron job | 设 workdir=<project-root> | AGENTS.md 自动注入 |
| 交互会话 | read_file(...) | 手动读取项目记忆 |
| 示例 | ~/.hermes/skills/short-drama-master/.hermes/memory/WORKSPACE.md | short-drama-master示范 |

---

## 跨项目同步桥(WorkBuddy评分6/10→Hermes补强)

| 方式 | 说明 |
|------|------|
| 方式1: memory工具(全局桥) | 跨项目经验直接写入memory，所有会话自动加载 |
| 方式2: cronjob context_from(串联桥) | job A→context_from→job B(流水线)，注: A输出为空/ [SILENT] 时 B跳过 |
| 方式3: session_search(按需桥) | "之前类似项目怎么做的?"→session_search检索 |

---

## 🅡 工作区记忆 vs workspace/ 目录的分工

当前 Hermes 有两个"项目相关存储区"，需要明确分工以避免混淆:

| 维度 | `~/.hermes/workspace/` | `.hermes/memory/` (Layer ③) |
|------|----------------------|-----------------------------|
| **用途** | 临时工作区、一次性产出 | 项目持久记忆 |
| **内容** | 研究输出、分析报告、临时文件 | 进度追踪、技术决策、踩坑记录、待办 |
| **生命周期** | 按需创建，用完可删 | 跟随项目生命周期 |
| **跨会话** | 手动读取 | cron workdir 自动加载 |
| **示例** | pf3-课程分析-Annie.md | WORKSPACE.md, DECISIONS.md |

**原则**: 
- `workspace/` = 工作台，干完活可以清
- `.hermes/memory/` = 档案室，项目结束了也要归档保留
- 短期研究报告放 `workspace/`，长期项目记忆放 `.hermes/memory/`

---

## 🅡 错误恢复与熔断机制

### 自动学习 cron 失败处理

| 故障模式 | 症状 | 处理 |
|---------|------|------|
| state.db 连接失败 | auto-learn.py 报 ERROR | cron 输出 [SILENT]，等下次4h重试 |
| memory 写入失败(满) | memory add 报 usage 错误 | 见 Layer② 熔断回路 |
| session_search 无结果 | 4h 内无活跃会话 | 输出 [SILENT]，正常静默 |
| LLM 推理超时 | cron 执行超3分钟 | 指数退避重试（首次30s→二次60s），失败2次后告警 |

### 每日审计 cron 失败处理

| 故障模式 | 处理 |
|---------|------|
| 审计中删除错了 | 无法直接撤销，但可利用 CHANGELOG.md + snapshots/ 回滚。**保守原则**: 拿不准的条目留到下轮。**删除前自动备份到 CHANGELOG.md 预览区** |
| 审计中 memory 满 | 先删除1条最过时的再继续。如果删不了→报错+推送告警 |
| 推送失败 | 报告存 local，下次审计时追加 |

### 工作区记忆损坏/误删

**无自动恢复机制**（文件系统操作）。建议:
- 重要决策同时写入 memory（全局桥方式1）
- 定期手动备份 `.hermes/memory/` 目录

### 🅡 Memory 文件漂移恢复流程

**症状**: 用 memory 工具写入时提示 "Refusing to write MEMORY.md: file on disk has content that wouldn't round-trip"

**根因**: `patch` 工具、`write_file` 或外部编辑器直接修改了 MEMORY.md/USER.md 文件，导致 memory 工具的内部哈希校验失效（安全机制，防止数据丢失）。

**恢复步骤**:

```bash
# 1. 备份当前文件
mv ~/.hermes/memories/MEMORY.md ~/.hermes/memories/MEMORY.md.bak.drift
# 或
mv ~/.hermes/memories/USER.md ~/.hermes/memories/USER.md.bak.drift

# 2. 创建空文件让 memory 工具重新接管
touch ~/.hermes/memories/MEMORY.md

# 3. 从备份中逐条恢复（通过 memory 工具）
memory(action='add', target='memory', content='**原始条目内容**')
# 逐条添加，注意容量上限

# 4. 备份后清理 + 验证
rm ~/.hermes/memories/MEMORY.md.bak.drift
memory(action='add', target='memory', content='**✅验证条目** - 测试写入（用后删除）')
memory(action='remove', old_text='验证条目')
```

**预防**:
- 不要直接 `patch` MEMORY.md/USER.md 文件
- 所有写入走 memory 工具（`memory(action='add'/'replace'/'remove')`）
- 如果必须用 `write_file` 修改，先备份再操作，操作完后立即用 memory 工具重建

---

### 🅡 neural-memory MCP 对接

**状态**: ✅ 已连接 (v4.58.0, 63 tools) — `hermes mcp test neural-memory` → 454ms

**安装位置**: 系统 Python 3.12 site-packages，通过 MCP stdio 通信

Neural Memory 为 Layer ④ (外部记忆引擎) 提供增强能力，与记忆大师互补:

| 当前 | 接入 neural-memory 后 |
|------|----------------------|
| FTS5 关键词匹配 | 语义相似度搜索 |
| memory 工具 2,200 字符限制 | 向量 DB 无容量上限 |
| 手动写写入原则控制膨胀 | 自动聚类+去重+摘要 |
| session_search 需精确关键词 | 模糊语义"我记得讨论过那个..." |

**参考**: references/neural-memory-integration.md (详细工具清单+对接策略+热冷分离方案)

### 🅡 中文→英文映射表设计原则

`sync-to-neural-memory.py` 中的 CN_EN_MAP 是记忆大师与 Neural Memory 的中文桥接。**必须维护为单一数据源**——两个独立副本会迅速分歧（导入用48条 vs 检索用29条），导致部分中文检索永远命中不了。

规则:
- 只有一个映射表 `CN_EN_MAP`，导入和检索共用
- 新增中文关键词在映射表中加一条，**不要**新建第二个映射表
- 所有指向同一个中文词的英文锚点必须在此表中统一管理
- 验证: `--check` 模式下检查锚点覆盖率，确保新增中文词有映射

**🅡 MCP 重连/降级机制**:
- neural-memory MCP 断开时，检索自动降级:
  - 中文查询: session_search + mm search（绕过 Neural）
  - 英文查询: session_search + 同义词扩展
  - 模糊查询: 扩展关键词后重试 session_search
- 重连尝试: cron 每4h 自动重试 `hermes mcp test neural-memory`
  - 连续3次失败后记录告警并继续走降级路径
  - 成功后自动恢复全量检索能力

**🅡 向量库损坏恢复**:
- chromadb 索引损坏症状: `memory-vector.py search` 报错
- 恢复: `python3 ~/.hermes/scripts/memory-vector.py index --force`
  - 自动删除旧索引并重建
  - 重建后验证: `python3 ~/.hermes/scripts/memory-vector.py status`

---

## 🅡 检索路径选择规则 (v5.3 — 自动判断) ⚠️ 硬行为规则

当用户问"你还记得...？"或需要查找记忆时，**禁止问用户"用哪个搜"**。这是行为铁律，不是建议。

### 决策算法 (无条件执行)

```
收到记忆查询
  ↓
① 检查问题是否能用 memory 上下文回答（已注入）
  → 能: 直接回答，标注 [memory]
  → 不能: 继续
② 判断查询语言
  → 中文为主: 走 session_search + mm search (chromadb·原生中文支持)
  → 英文/混合: 走 Neural Memory (nmem_recall) + session_search
③ 如果问题涉及实体关系/因果/属性:
  → 加跑 mm search (3通道RRF含Entity通道)
④ 如果问题模糊、记不清关键词:
  → 走 Neural Memory (spreading activation 适合模糊匹配)
⑤ 综合性问题: 多路融合，取最佳结果
⑥ 全部路径无结果 → 回复「我没有找到相关记忆，你可以再告诉我一遍」，不胡编乱造
```

### 场景速查表

| 场景 | 首选路径 | 次选/备选 |
|------|---------|-----------|
| 查用户偏好/规则/事实 | **memory 工具上下文**(已注入) | - |
| 查过去会话内容 | **session_search** | mm search |
| 查记忆间关联/属性/实体 | **`mm search`**(chromadb·3通道RRF) | Neural Memory |
| 查知识图谱/因果链/假设 | **Neural Memory**(nmem_recall) | mm search |
| 综合性/模糊记忆 | **三者融合**: context+session_search+mm search | Neural Memory补充 |

### 报告格式

汇报结果时标注来源和优先级：
- `[memory]` — 来自注入的 memory 上下文
- `[session_search]` — 来自历史会话搜索
- `[mm search]` — 来自向量+实体混合检索
- `[Neural Memory]` — 来自知识图谱

**优先级标注**:
- `⭐ authoritative` — 已确认的 WF/MM 事实，直接使用不重复确认
- `✓ normal` — 一般记忆，可作为参考
- `⚪ fallback` — 低确信度信息，需要验证

- **Ground Truth 规则 (Phase 13) — 适用范围说明 (🅡123)**:
- authoritative > normal > fallback > 模型推理
- 此优先级**仅在检索排序时生效**（memory-injection-filter.py 排序过滤后的结果）
- memory 全量注入(2,200字符)不做优先级排序
- 标记为 authoritative 的记忆：**Agent 直接使用，不再询问用户确认**
- 用户纠正某条记忆后，立即更新其权威状态

**铁律**: 不问用户"用哪个搜"——自己判断后直接执行, 汇报时标注来源。违反此规则 = 用户明确纠正过的行为模式。

---

## 🅡 记忆质量仪表盘

### 核心指标(每轮审计报告自动跟踪)

| 指标 | 健康值 | 红区 | 说明 |
|------|--------|------|------|
| 容量占比 | <60% | >80% | memory 使用率 |
| 条目数 | 8-15条 | >20条 | memory 中条目数量 |
| 自动学习写入量 | ≤6条/轮 | >10条/轮 | 自动学习每轮实际写入的条目数 |
| 过时淘汰率 | ≤20%/月 | >40%/月 | 每月审计删除的条目数占比 |
| 同类错误复发率 | 0 | ≥1 | 用户纠正过的同类错误是否重复犯 |
| 工作区记忆同步率 | 100% | <50% | 工作区 cron 是否都设了 workdir |

### 记忆健康报告格式示例

```
🧠 记忆健康报告 — 2026-06-18
━━━━━━━━━━━━━━━━━━━━━━━
容量: 1,772/2,200 (80%) ⚠️ 黄色
条目: 11条 ✅ 健康
命中: 上周学习3条，被引用1条 (33%) ✅
过时: 本月删除2条 (18%) ✅
复发: 0次 ✅

操作: 合并2条同类项，删除1条过时
```

---

## 🅡 未来扩展路线图

### Phase 1 — 三层基础架构 ✅ 已完成
- **Layer① 云端自动学习**: 每4h自动扫描会话，提取偏好/纠正/事实写入memory
- **Layer② 用户持久规则**: Hermes memory 工具手动写入硬性规则，每次会话自动注入
- **Layer③ 项目隔离层**: 每个项目独立 .hermes/memory/ 目录，互不影响
- **验证**: 三条cron已部署并稳定运行

### Phase 2 — 记忆回溯→技能固化 ✅ 已实现
- 自动学习检测到某类纠正/习惯出现≥3次
- → 自动创建或更新 skill(建议操作)
- → continuous-improve-sk 的 Level 3 升级路径已集成
- **脚本**: `~/.hermes/scripts/memory-to-skill.py`
- **验证**: 已成功检测4个可固化模式(规则偏好12次/技术栈13次/short-drama-master5次/视觉设计3次)

### Phase 3 — 记忆版本控制 ✅ 已实现
- 每次 memory 变更自动记录到 CHANGELOG.md
- 支持按时间点快照回滚
- 存储: `~/.hermes/memories/CHANGELOG.md` + `~/.hermes/memories/snapshots/`
- **脚本**: `~/.hermes/scripts/memory-changelog.py`
- **验证**: 首次快照已记录

### Phase 4 — 记忆可视化仪表盘 ✅ 已实现
- HTML 单页仪表盘(Mac暗黑风格)，展示三层记忆状态
- 包含: memory容量/条目数/24h会话/Cron健康/变更日志
- 路径: `~/.hermes/skills/hermes/memory-master/assets/dashboard.html`
- **脚本**: `~/.hermes/scripts/memory-dashboard.py`
- **验证**: 已生成(11 MEMORY/11 USER/63个24h会话/已部署cron)

### Phase 5 — 多 profile 记忆同步 ✅ 已实现
- 默认 profile 的记忆自动同步到其他 7 个 profile
- 同步路径: `~/.hermes/profiles/<name>/memories/`
- **脚本**: `~/.hermes/scripts/cross-profile-sync.py`
- **验证**: 5/7 profiles 已同步(jiaoyi/shequ/biganzi/canmou/jinhua); 2个profile(state.db未初始化)等创建后自动同步

---

## 🆕 从 Hindsight 学到的经验（v5.0 融合）

> **来源**: 公众号「设计虱聊科技」— NAS本地部署Hindsight：Hermes高级记忆系统
> **链接**: https://mp.weixin.qq.com/s/xr0f4LGOLTjWoF4vl4Fz9A
> **开源**: https://github.com/vectorize-io/hindsight
> **收录时间**: 2026-06-18
>
> **🅡134 冗余说明**: 本节的4型分类/Reflect反思/RRF融合在下方【Hindsight 对照表】和【Phase 路线图】中已总结，此处保留完整对照供参考。

Hindsight 是 Hermes 的**外置记忆大脑**插件，通过外部数据库存储海量长期记忆。核心机制 Retain → Recall → Reflect。

### 🆚 记忆大师 vs Hindsight 对照表

| 维度 | Hindsight | 记忆大师 v5.0 | 融合 |
|------|-----------|---------------|------|
| **存储上限** | 外部DB，几乎无限 | memory 2,200字符 + 向量库 | 互补（热记忆+冷记忆） |
| **记忆分类** | 4种(World Fact/Experience/Observation/Mental Model) | 5种学习类型(风格/纠正/事实/关系/过时) | **🅡 已融合为4类** |
| **检索** | TEMPR(语义+关键词+图谱+时间+重排序) | 混合检索(向量+FTS5+同义词+RRF) | **🅡 已加RRF** |
| **反思** | 后台自动Reflect→更新图谱→合并→生成模型 | 每日审计+cron | **🅡 已加Reflect阶段** |
| **部署** | Docker容器(500MB+TEI) | 纯Python脚本+chromadb | 各有取舍 |
| **重排序** | RRF + 独立reranker | RRF(k=60) | 持平 |
| **知识图谱** | 图谱遍历(TEMPR的M) | 无 | Phase 8 |

### 🅡 核心融合1: 四种记忆类型（已集成到 auto-learn）

**旧分类（v4.0）**: 偏好/纠正/事实/关系/过时 → 5种模糊分类
**Hindsight分类（v5.0）**:

```
World Fact（世界事实）
  → 相对稳定但可更新的客观事实
  → 例: "[Author Name]是short-drama-master作者" "项目版本v6.2.5"（版本升级属于更新，不属矛盾消解）
  → 写入格式: **[WF] 标题** - 内容

Experience Fact（经验事实）
  → AI的行动记录与经验
  → 例: "用户上次用黑底金字做了海报" "我推荐过使用Kimi模型"
  → 写入格式: **[EF] 标题** - 内容

Observation（观察结论）
  → 整合多条事实得出的推断
  → 例: "用户偏好黑底金字设计风格"（从5次海报设计总结）
  → 写入格式: **[OB] 标题** - 内容

Mental Model（心智模型）
  → 高层次规则/红线
  → 例: "铁律：不主动push" "配色规则：黑底#0A0A0A+暖金#C9A96E"
  → 写入格式: **[MM] 标题** - 内容
```

**5种学习事件 → 4类存储类型映射表**:

| 学习事件类型 (原始) | 存储类型 | 说明 |
|:------------------:|:--------:|------|
| 🎨 风格偏好 | **MM** (Mental Model) | 用户表达的规则级偏好 |
| ✅ 正确做法 | **MM** 或 **EF** | 纠正后的正确做法→MM；单纯记录→EF |
| 📋 新事实 | **WF** (World Fact) | 客观事实，不混入教训 |
| 🔄 关系变更 | **WF** 或 **EF** | 人物/项目关系→WF；AI操作→EF |
| 🗑️ 过时删除 | — | 直接删除旧条目，不映射存储类型 |

**分类决策树**:
```
这条信息是关于:
  客观事实 → World Fact [WF]
  AI做过什么 → Experience Fact [EF]
  → 从多个线索推断 → Observation [OB]
  → 高层次规则/红线 → Mental Model [MM]
  → 不属于以上4类 → 暂存到 pending-memory.md，由每日审计处理
```

### 🅡 核心融合2: RRF 融合排序（已集成到 memory-vector）

Hindsight 的 TEMPR 混合检索包含 5 个通道。记忆大师 v5.0 在 `memory-vector.py` 中实现了:

| 通道 | 实现 | 状态 |
|------|------|------|
| T 时间过滤 | RRF中的time_boost(新条目权重+2%) | ✅ |
| E 嵌入向量 | chromadb语义搜索 | ✅ v4.0 |
| M 图谱遍历 | (Phase 8 规划) | ❌ |
| P 关键词匹配 | TF-IDF关键词匹配 | ✅ **新** |
| R 重排序 | RRF(k=60)融合+时间衰减 | ✅ **新** |

RRF 融合公式:
```
score = 1/(60+vector_rank) + keyword_match*0.5 + time_boost
```

### 🅡 核心融合3: Reflect 反思阶段（已集成到 auto-learn cron）

Hindsight 的 Reflect 三阶段在每日审计中的实现:

```
写入新 memory 之前:
  1. 检查同类条目
     → 已有 2 条相似 Observation?
     → 合并为 1 条 Mental Model（上升到规则）
     
  2. 检查矛盾
     → 新发现与旧 Memory 冲突?
     → 标记旧条目为过时 + 写入新条目
     
  3. 检查可固化
     → 该模式已出现 ≥3 次?
     → 同时触发 skill 固化 (Phase 2)
```

### 与 Hindsight 的互补关系

记忆大师不替代 Hindsight，而是互补:

| 场景 | 用记忆大师 | 用 Hindsight |
|------|-----------|-------------|
| 每次会话自动注入 | ✅ memory工具2,200字符 | ❌ 按需检索 |
| 海量长期记忆 | ❌ 容量受限 | ✅ 外部DB无限 |
| 知识图谱遍历 | ❌ 无 | ✅ 图谱 |
| 单独对话回顾 | ✅ session_search | ✅ 可搜索 |
| 零依赖部署 | ✅ 纯Python | ❌ 需Docker |
| 跨profile同步 | ✅ 7个profile | ❌ 单实例 |

> **来源**: 公众号「逛逛 GitHub」— 《给 10 万 Star 的 Hermes 装个记忆外挂》
> **链接**: https://mp.weixin.qq.com/s/RoAFEKEetZ9mnmVKwrS4jg
> **开源**: https://github.com/MemTensor/MemOS/tree/main/apps/memos-local-plugin
> **收录时间**: 2026-06-18
>
> **🅡134 冗余说明**: 本节智能去重/混合检索/预检索在【Phase 6 路线图】中已总结，此处保留完整对照供参考。

MemOS（记忆张量团队）为 Hermes 开发的本地记忆插件，解决了原生记忆系统的两大痛点：**记得乱**（重复/矛盾/过时堆砌）和**找不准**（关键词搜不到）。以下是与记忆大师的深度对照与融合。

### 🆚 记忆大师 vs MemOS 对照表

| 维度 | MemOS 插件 | 记忆大师 v3.0 | 融合方向 |
|------|-----------|---------------|---------|
| **写入管道** | 语义分片→LLM摘要→向量化→智能去重 | session_search→LLM提取→直接写入 | **🅡 新增智能去重** |
| **去重逻辑** | LLM判断：重复/需更新/全新（识别更新关系） | 每日审计手动合并（被动） | **🅡 已升级** |
| **检索引擎** | 混合检索(全文+向量语义+时间衰减+相关性过滤) | FTS5 + session_search | Phase 6 向量检索 |
| **预检索** | 每轮对话开始自动预检索+注入上下文 | memory工具全量注入(容量有限) | Phase 6 预检索 |
| **模型配置** | 三级独立(Embedding/摘要/生成)+降级机制 | 统一cron模型 | **🅡 已升级** |
| **多Agent** | 独立+公共记忆空间+Hub-Client | cross-profile-sync.py同步 | Phase 7 公共池 |
| **管理面板** | Web面板(7页,可交互) | dashboard.html(只读) | Phase 6 交互面板 |
| **token开销** | 首次多消耗token做摘要 | 每次cron执行消耗token | 各有取舍 |
| **长期价值** | 越用越准(累积效应) | 越用越精(审计+熔断) | 互补 |

### 🅡 核心融合点1: 智能去重（已集成）

**MemOS 的原创做法**: 不是简单文本比对，而是把当前要存的内容和已有的相似记忆做对比，让 LLM 判断到底是什么关系:

| 判断结果 | 含义 | 记忆大师的做法 |
|---------|------|--------------|
| **重复** | 已存在完全一致的内容 | 不写入，跳过 |
| **需更新** | 是对已有内容的更新(如"减肥→放弃减肥") | 用 `memory(action='replace')` 替换旧条目，记录变更历史 |
| **全新的** | 以前从未出现过 | 用 `memory(action='add')` 正常写入 |

**在记忆大师中的实现**（集成到 auto-learn cron 和每日审计）:

```
自动学习 cron 有一个新发现:
  "用户说放弃减肥恢复正常饮食"
  
① 在 memory 中搜索相似条目
  → 找到"用户说最近在减肥，每天1800大卡"

② LLM 判断关系:
  → 是对旧条目的更新（状态从"减肥中"→"已放弃"）

③ 执行:
  memory(action='replace', old_text='1800大卡', content='**已更新**: 用户放弃减肥恢复正常饮食。过去记录: 曾每天控制1800大卡')
  → 1条记录替代2条,含变更历史

④ 记录到 CHANGELOG
```

**处理效果对比**:

| 场景 | MemOS 原生(无插件) | 原记忆大师 | 融合后记忆大师 v3.0 |
|------|-------------------|-----------|-------------------|
| "我减肥"→"放弃减肥" | 2条矛盾记录 | 2条矛盾记录(待审计合并) | 1条合并记录+变更历史 ✅ |
| "用Kimi"→"改用DeepSeek" | 2条重复记录 | 1条(手动写入即覆盖) | 1条+自动检测到更新关系 ✅ |
| 每周都说"颜色用黑底金字" | 每周1条=52条/年 | 1条(人类不重复写) | 1条(cron识别到重复自动跳过) ✅ |

### 🅡 核心融合点2: 混合检索理念

MemOS 的双通道检索可以用到记忆大师的跨项目同步桥和自动学习中:

**当前（FTS5 关键词）**:
```
用户问: "上次推荐了什么好吃的地方？"
→ session_search 搜 "推荐 好吃" → 搜不到（原文没这些词）
→ 结果: 明明存了，但搜不到
```

**融入 MemOS 理念后（Phase 6）**:
```
用户问: "上次推荐了什么好吃的地方？"
→ 通道1(全文): session_search 搜 "推荐 好吃" → 0条
→ 通道2(语义): 向量搜索相似意图 → 命中"某某餐厅味道不错"
→ 融合排序: 语义结果排名更高
→ 最终: 成功找到 ✅
```

**短期替代方案（不等向量DB）**：
- 在自动学习 cron 的 prompt 中加一条: "分析近期搜索时，如果发现用户用不同词提问同一件事，记录为 synonyms 写入 memory"
- 比如搜了"好吃的地方"又搜"推荐餐厅"→记录 `synonyms: 推荐≈好吃的地方≈餐厅`

### 🅡 核心融合点3: 预检索机制

**MemOS**: 每轮对话开始自动用最新消息做一次预检索，把相关记忆注入上下文。未命中→提示Agent主动搜索。

**记忆大师当前**: memory 工具全量注入（受限于2,200字符）

**融合方案**（Phase 6）:
```
对话开始
  ↓
① 取用户最新消息中的关键词
② session_search 搜索相关历史
③ 命中→将结果注入当前上下文
④ 未命中→提示"可以让我搜一下历史，用 session_search"
```

**短期可实现**:
- 在每轮回复前，Agent 自动做 `session_search(query="用户当前问题关键词")`
- 如果发现命中，在回复开头说明"我在记忆中找到..."
- 这不需要改 Hermes 代码，改 Agent 行为即可

### 🅡 核心融合点4: 模型配置

**记忆大师当前**: 所有 cron 统一用 lightweight-llm。

> 注: 原文档中的"三级模型配置(Embedding/摘要/生成)"规划未落地。当前仅有一级配置。
> 如需启用三级配置(技能固化用更强模型)，需单独调 cron 的 `--model` 参数。
> 目前无此项需求，保持一级配置简洁可靠。

### 🅡 核心融合点5: 管理面板交互化

**当前记忆大师仪表盘**: `assets/dashboard.html`（只读 HTML，由 `memory-dashboard.py` 定期生成）
**MemOS 面板**: 7个交互页面，支持记忆浏览/搜索/管理/工具调用日志/数据导入

**升级方向**（Phase 6）:
- 在 dashboard 中加入交互功能: 搜索模拟器、记忆条目卡片、批量操作按钮
- 技术方案: dashboard.html 中嵌入 JavaScript，通过本机 API 读取 memory 文件

---

## Phase 6 路线图（来自 MemOS 启发）

### P6a: 向量语义检索 ✅ 已实现
- 安装: chromadb 1.5.2 + sentence-transformers (all-MiniLM-L6-v2)
- 所有 memory 条目已向量化索引
- 语义搜索: 向量库97条数据，功能验证通过
- 脚本: `~/.hermes/scripts/memory-vector.py`
  - `index` 重建向量索引
  - `search "查询词"` 混合语义检索
  - `status` 查看向量库状态
- 向量库: `~/.hermes/memories/vectors/` (chromadb 持久化)

### P6b: 对话预检索 ✅ 已实现
- 每次对话开始前自动用用户消息做预检索
- 同义词引擎: 自动学习维护的 `~/.hermes/memories/synonyms.txt`
  - 格式: `推荐≈好吃的地方≈餐厅`
- 命中→注入上下文 | 未命中→提示主动搜索
- 预检索上下文存到 `~/.hermes/memories/preload_context.txt`
- 脚本: `~/.hermes/scripts/memory-pre-retrieval.py`
  - `python3 memory-pre-retrieval.py "用户消息"`
  - `--save` 同时保存到 preload 文件

### P6c: 可搜索仪表盘 ✅ 已实现 (v4)
- HTML 仪表盘带 7 个浏览选项卡（只读，纯前端无需服务器）:
  1. 📖 记忆浏览(可搜索/过滤 MEMORY/USER)
  2. 🔍 语义搜索(向量引擎查询入口)
  3. 📊 质量基准(LongMemEval对标卡+生态对比+Entity状态)
  4. ⏱ Cron健康(13个cron状态)
  5. 📋 变更日志(最近20条)
  6. 📂 工作区(short-drama-masterWORKSPACE.md)
  7. 📈 统计(含质量趋势条+向量覆盖率+实体标注率)
- 纯前端，无需启动服务器，v4新增: 基准对标卡+质量趋势条+工具提示
- 路径: `assets/dashboard.html` (v4)
- 脚本: `~/.hermes/scripts/memory-dashboard.py` (v4)

### Phase 7: 公共记忆池 ✅ 已实现
- 标签系统: 条目可标记 [shared] / [private]
  - [shared] → 同步到所有 profile
  - [private] → 仅本地保留
  - 无标签 = 默认 shared (向后兼容)
- 当前所有 memory 全部默认 shared
- 同步范围: 7 个 profile (biganzi/canmou/jiaoyi/jinhua/shequ/yunying/zongzhihui)
- 脚本: `~/.hermes/scripts/cross-profile-sync.py` (v2, 含标签检测)

### Phase 8: 知识图谱 (来自 Hindsight TEMPR 的 M) ⏳ 规划中
- 从 WF/EF/OB/MM 条目提取实体和关系
- 存储格式: knowledge-graph.json (Python dict)
- 在 memory-vector.py RRF 中增加 M 通道(图谱检索)
- 详见: `references/supermemory-integration.md` 及下方独立章节

### Phase 9: 时间线追踪 + 自动遗忘 ✅ 已完成 v5.6.0
- 自动检测状态变更动词（搬到/改为/换成/升职/离职/改名/变成/成为/换到/迁移/升级/降级/转做/接手/放弃/停止/启动/新建/废止）
- 检测到变更 → auto-learn 输出变更报告，LLM 追加 `| superseded:YYYY-MM-DD` 保留时间线
- **新增: 自动遗忘引擎** `memory-auto-forget.py`:
  - 扫描 `| TTL:YYYY-MM-DD` 过期标记 → 到期自动删除
  - 删除超过30天的 superseded 条目
  - `| Layer:dynamic` 无 TTL 条目自动分配30天默认TTL
  - cron: 每日 20:30 (job=[JOB_FORGET], no_agent)
  - 调用: `mm forget` / `mm forget --dry-run`
- 检测时间敏感词自动追加短 TTL

### Phase 10: 静态 vs 动态画像分离 ✅ 已完成 v5.6.0
- user profile 拆分为 static/dynamic 两层，auto-learn 写入时自动标记
- static: 姓名/职业/长期偏好 → `| Layer:static` 不设TTL，永久有效
- dynamic: 当前项目/临时偏好 → `| Layer:dynamic` 默认TTL 30天
- auto-learn 自动检测时间敏感词标记为 dynamic

### Phase 11: Memory≠RAG 概念固化 ✅ 已完成 v5.6.0
- auto-learn 输出中增加 Memory/RAG 概念对比表，LLM 写入时明确区分
- Memory: 追踪用户事实/偏好/规则 → 写入 MEMORY.md/USER.md
- RAG: 搜索历史对话 → 仅 session_search，不写入
- 核心原则: "记忆不是更快地找到文档，而是理解用户是谁、什么变了、什么过期了"

### Phase 12: 精准上下文注入 ✅ 已完成 v5.5.2
- memory-injection-filter.py 4道过滤管道: 相关性阈值(≥0.3) → 去琐碎(好/谢谢/👌) → per-session去重 → GT优先级标记
- 集成到 memory-pre-retrieval.py: 预检索结果自动经过过滤管道
- 脚本: ~/.hermes/scripts/memory-injection-filter.py (4.3KB)
- 触发: 每次预检索自动运行，过滤失败不阻塞主流程
- 详见 Memory OS 融合章节

### Phase 13: Ground Truth 层级 ✅ 已完成 v5.5.2
- 检索结果标记 authoritative ⭐ / normal ✓ / fallback ⚪ 三级
- Ground Truth 规则已写入检索路径选择规则章节（含场景速查表）
- 用户纠正某条记忆后立即更新其权威状态
- 消除 memory-zero behavior — authoritative 记忆直接使用不确认

### Phase 14: 7层结构对齐 ✅ 已完成 v5.5.2
- 新增 CREATIVE.md（灵感/构思专用）— TTL 7天，不参与审计
- 灵魂层 SOUL.md — OpenClaw 占位符 → 行为价值观(五大信条)+GT优先级，每次会话自动注入
- memory-pre-retrieval.py 集成 SOUL.md 加载: 预检索时自动注入灵魂层行为上下文
- 7层最终状态: 1-会话 → 2-事实 → 3-用户 → 4-创意 → 5-灵魂 → 6-规则 → 7-GT

---

## 🆕 从 Supermemory 学到的新能力（v5.5.0 融合）

> **来源**: 公众号文章「23.7K+ star 的 AI 记忆引擎：让大模型不再"金鱼记忆"」
> **GitHub**: https://github.com/supermemoryai/supermemory
> **收录时间**: 2026-06-20
>
> **🅡134 冗余说明**: 本节4个融合点(时间线/画像分离/自动遗忘/Memory≠RAG)在【Phase 9-11】中已总结，此处保留完整对照供参考。

Supermemory（23.7K star）是目前 AI 记忆领域三大 benchmark（LongMemEval / MemEval / MT-Bench-Memory）第一的项目。来源: https://github.com/supermemoryai/supermemory
它和我们的核心区别不是存储技术，而是**对记忆的认知层级不同**——我们存"信息"，它理解"变化"。

### 🆚 记忆大师 vs Supermemory 对照表

| 维度 | Supermemory | 记忆大师 v5.4 | 融合方向 v5.5 |
|------|------------|---------------|---------------|
| **矛盾消解** | 时间线追踪: NYC→SF→NYC 保留变更历史 | 简单覆盖(replace) | **Phase 9: 时间线追踪** |
| **画像分离** | static(偏好) vs dynamic(上下文) | 一个池子混合 | **Phase 10: 分层画像** |
| **自动遗忘** | 自动检测日期敏感信息→短TTL | 手动 TTL 标注 | **Phase 9: 时间词检测** |
| **Memory≠RAG** | 明确区分记忆与检索 | 概念模糊 | **Phase 11: 概念固化** |

### 🅡 核心融合 1: 矛盾消解时间线追踪（Phase 9）

**问题**: 用户说"我住 NYC"→"搬到 SF"→"又搬回 NYC"。当前做法:
```
第1次: memory add "住 NYC"
第2次: memory replace "住 NYC" → "住 SF"  (丢失了"曾经住过NYC"的历史)
第3次: memory replace "住 SF" → "住 NYC"  (又覆盖)
→ 最终只有"住 NYC"，中间变更历史全丢了 ❌
```

**Supermemory 的做法**:
```
第1次: timeline[0]: {date:2026-01, value:"NYC"}, current:NYC
第2次: 追加 timeline[1]: {date:2026-06, value:"SF"}, current:SF
第3次: 追加 timeline[2]: {date:2026-07, value:"NYC"}, current:NYC
→ 保留完整时间线，查询时优先返回 current ✅
```

**记忆大师实现方案**:
- 检测"状态变更"类动词（搬到/改为/换成/升职/离职...）
- 写入时不是 replace，而是追加 timeline 数组 + 更新 current
- 保留历史不删除，session_search 可回溯

详见: `references/supermemory-integration.md`

### 🅡 核心融合 2: 静态 vs 动态画像分离（Phase 10）

**旧做法（v5.4）**: `memory(target='user')` 一个池子混着姓名+项目+偏好+临时状态

**新做法（v5.5）**:
```yaml
静态(static):   姓名、职业、长期偏好、人格特征、固定习惯
                → 不设TTL（永久有效） → 每次对话全量注入
动态(dynamic):  当前项目、当前工作状态、近期计划、临时偏好
                → 默认TTL 30天 → 按项目周期更短
```

**实施**:
- 写入格式扩展: `Layer:static|dynamic`
- 自动学习 cron 写入时自动判断
- 每日审计: static 不动，dynamic 检查时效性

详见: `references/supermemory-integration.md`

### 🅡 核心融合 3: 自动遗忘增强—时间敏感词检测（Phase 9）

**旧做法**: TTL 全靠手动标注 `\| TTL:YYYY-MM-DD`

**新做法**: 自动学习 cron 检测时间敏感词:
```python
TIME_SENSITIVE_PATTERNS = [
    r"明[天日年]?", r"下[周月年]", r"[这今][周月天]",
    r"周[一二三四五六日]", r"\d+月\d+日", r"(月底|月初|年末)"
]
# 检测到 → 计算过期日期 → 自动追加 TTL → 标记为 dynamic
```

详见: `references/supermemory-integration.md`

### 🅡 核心融合 4: Memory≠RAG 概念固化（Phase 11）

| 概念 | Supermemory | 我们的映射 |
|------|------------|-----------|
| **Memory** | 追踪关于用户的事实 | Layer② 用户记忆 |
| **RAG** | 检索文档片段 | session_search |
| **区别** | 有状态、个性化、有时效、有矛盾 | 无状态、非个性化、静态 |

> 记忆不是"更快地找到文档"，而是**理解用户是谁、什么变了、什么过期了**。

### 与已有融合的排重

| 已有融合 | Supermemory 对应 | 是否重复 |
|---------|-----------------|:--------:|
| Hindsight 4型分类 | 事实抽取 | ❌ 互补——他们不分型，我们保留4型 |
| MemOS 智能去重 | 矛盾消解 | ⚠️ 部分重叠。去重≠时间线，并行运行 |
| RRF 3通道 | Hybrid Search | ❌ 方案相似，理念一致 |
| 手动 TTL | 自动遗忘 | ⚠️ 功能重叠。升级为自动检测时间敏感词 |

---

## 🆕 从 Memory OS 学到的新能力（v5.5.1 融合）

> **来源**: 公众号「arduino」— 《1045 Star、7 层记忆 OS：用 Qdrant+SQLite 给 Hermes Agent 装本地长期记忆》
> **GitHub**: https://github.com/ClaudioDrews/memory-os
> **收录时间**: 2026-06-20

Memory OS（1,045 Star）是一个面向 Hermes Agent 的 7 层本地长期记忆系统。它不是又一个向量库外挂，而是把"记住"拆成了多层工程问题。

### 🆚 记忆大师 vs Memory OS 对照表

| 维度 | Memory OS | 记忆大师 v5.5 | 融合方向 v5.5.1 |
|------|-----------|---------------|-----------------|
| **记忆分层** | 7层(会话/事实/用户/创意/灵魂/规则/Ground Truth) | 3层(云端/用户/工作区) | **Phase 14: 创意层对齐** |
| **上下文注入** | pre_llm_call 多源召回+阈值过滤+去重+跳过琐碎 | 全量注入(2,200字符上限) | **Phase 12: 精准注入** |
| **权威层级** | Ground Truth 标记，显式告诉Agent优先使用 | 无 | **Phase 13: Ground Truth** |
| **琐碎过滤** | 跳过社交性结尾等无用内容 | 无 | **Phase 12: 过滤** |
| **基础设施** | Qdrant+SQLite+Redis+Docker | chromadb纯Python零Docker | 各有取舍(我们少Redis但零依赖) |

### 🅡 核心融合 1: 精准上下文注入（Phase 12）

**问题**：全量注入 2,200 字符，无过滤、无去重、无优先级。低价值信息占用宝贵上下文槽位，高价值信息被稀释。

**Memory OS 的做法**：
```
pre_llm_call 管道:
  ① session DB → 最近 N 条相关会话
  ② chromadb → 语义相似记忆向量（替代 Qdrant）
  ③ SQLite → 精确匹配的结构化事实
  ↓
  ④ 相关性阈值过滤 (score < 0.3 淘汰)
  ⑤ per-session 去重
  ⑥ 跳过琐碎消息 (好的/明白了/谢谢/ok/👌)
  ⑦ Ground Truth 标记 → 注入
```

**记忆大师实现方案**:
- 在 memory 注入前加过滤管道
- 定义琐碎消息正则列表: `r'^(好的|明白了|谢谢|嗯嗯|ok|👌)'`
- per-session 去重: 同一 session 内容不重复注入
- 相关性阈值: chromadb 匹配分数 < 0.3 的跳过

详见: `references/memory-os-integration.md`

### 🅡 核心融合 2: Ground Truth 层级（Phase 13）

**问题**：Agent 明明有记忆，却不使用，每次重新问用户或重复工作。这叫 **memory-zero behavior**——看似有记忆，实际还在失忆式重复劳动。

**Memory OS 的做法**：
```python
# 第 7 层 Ground Truth hierarchy
# 注入时标记权威层级：
#   authoritative > normal > fallback
# system prompt 明确写入：
"以下记忆是你已经确认的事实，不需要再次询问用户"
```

**记忆大师实现方案**:
- 在检索结果中标记 authoritative/normal/fallback
- 更新检索路径规则：注入记忆 > 搜索历史 > 模型推理
- 用户纠正某条记忆后，立即更新权威状态

详见: `references/memory-os-integration.md`

### 🅡 核心融合 3: 7层结构对齐（Phase 14）

| Memory OS 层 | 我们已有 | 差距 |
|:------------:|:--------:|:----:|
| 1-3 会话/事实/用户 | ✅ | — |
| 4 创意层 CREATIVE.md | ❌ 无 | **新增** |
| 5 灵魂层 SOUL.md | ✅ 行为价值观(新) | 永久·每次会话注入 |
| 6 规则层 rulebook.md | ✅ skills/ | — |
| 7 Ground Truth | ❌ 无 | Phase 13 |

**新增 CREATIVE.md**: 专门存灵感/构思/草稿。TTL短(7天)，不参与持久审计。位置: `~/.hermes/memories/CREATIVE.md`

### 与已有融合的排重

| 已有融合 | Memory OS 对应 | 是否重复 |
|---------|----------------|:--------:|
| Hindsight 4型分类 | 事实/用户层 | ❌ 互补 |
| MemOS 智能去重 | per-session去重 | ⚠️ 部分重叠。去重防重复写入，Memory OS 去重防重复注入 |
| Supermemory 时间线 | 事实层追踪 | ❌ 互补 |
| RRF 3通道 | chromadb语义 | ⚠️ 近似(缺时间过滤+独立reranker两通道)|

---

### Phase 8 路线图: 知识图谱 (来自 Hindsight TEMPR 的 M)

### 过渡方案: v5.0 → Phase 8

```
v5.0 (当前)               Phase 8 (目标)
────────────────────      ────────────────────
WF/EF/OB/MM 条目          知识图谱(节点+关系)
┃                         ┃
┣ 每条 WF → 实体节点       ┣ 实体去重+合并
┣ EF 中的动作 → 关系边      ┣ "用户"→"使用"→"黑底金字"
┣ OB 中的推断 → 派生关系     ┣ "用户"→"偏好"→"暗黑风格"
┗ MM 中的规则 → 规则节点    ┗ "记忆大师"→"管理"→"memory容量"
```

### 建立步骤 (从 v5.0 开始逐步积累)

**Step 1 — 实体提取** (可立即开始):
- 从现有 memory 中手动或用 LLM 提取实体
- 实体类型: 人([Author Name])、项目(short-drama-master)、工具(Hermes/chromadb)、规则(铁律)、偏好(黑底金字)
- 存储: `~/.hermes/memories/knowledge-graph.json`

**Step 2 — 关系抽取**:
- 从 EF 和 OB 条目中提取实体间关系
- 关系类型: 使用、偏好、管理、属于、导致、矛盾
- 例: "[Author Name]→作者→short-drama-master" | "用户→偏好→黑底金字"

**Step 3 — 图谱检索集成**:
- 在 memory-vector.py 的 RRF 中增加 M 通道(图谱检索)
- 查询时先解析实体，然后遍历关系图，找到的相关节点也参与 RRF 排序

**技术选型**:
- 轻量: Python dict + JSON 文件 (≤500条时足够)
- 重量: NetworkX (Python 图库, 支持图遍历算法)

---

## 与short-drama-master® 的关联

### short-drama-master工作区记忆(已部署)
路径: ~/.hermes/skills/short-drama-master/.hermes/memory/WORKSPACE.md
状态: ✅ 已创建(2026-06-18)

### short-drama-mastercron已设workdir

| cron | workdir | 状态 |
|------|---------|------|
| 每日更新(03:00, job=[JOB_UPDATE1]) | ~/.hermes/skills/short-drama-master/ | ✅ 已设置 |
| 午夜自进化(02:30, job=[JOB_EVOLVE]) | ~/.hermes/skills/short-drama-master/ | ✅ 已设置 |

> 设置后，short-drama-master cron 每次运行时自动加载工作区记忆，可 `read_file` 读取 WORKSPACE.md 获取项目最新状态。

### 在short-drama-master中的定位
- **定位**: 独立守护者(不占大师封号，不占22师名额)
- **职责**: 确保short-drama-master经验不丢失、不膨胀、跨session连续
- **会审边界**: 不对短剧作品品控负责，只对记忆质量负责
  - ✅ 检查: 22大师每次进化后的关键发现是否被记忆
  - ✅ 检查: 用户对短剧作品的反馈偏好是否被自动学习
  - ❌ 不管: 短剧剧本质量、画面质量、节奏设计方案
- **记忆质量标准(会审用)**: 详见"记忆质量仪表盘"章节

---

## 日常运维

### 自动守护任务

| 内容 | cron | job_id | 说明 |
|------|------|--------|------|
| 🧠 云端自动学习 | 每4h 整点 | [JOB_AUTOLEARN] | 扫描→学习→写入 |
| 🧬 Neural Memory同步 | 每4h 过15分 | [JOB_NMSYNC] | no_agent, 新记忆→nmem_remember |
| 🗑️ 自动遗忘清理 | 20:30 | [JOB_FORGET] | no_agent, TTL过期释放 |
| 📊 每日记忆审计 | 22:00 | [JOB_AUDIT] | 容量+时效+合并+同步+报告 |

### 人工操作

| 操作 | 方式 |
|------|------|
| 查看记忆状态 | memory工具或问"当前记忆状态" |
| 手动触发学习 | 说"学习一下" |
| 同步到Neural Memory | `mm nm-sync` 或 `python3 ~/.hermes/scripts/sync-to-neural-memory.py` |
| 查看向量库状态 | `mm status` 或 `python3 ~/.hermes/scripts/memory-vector.py status` |
| 手动写入 | memory(action='add', target='memory', content='...') |
| 删除过时 | memory(action='remove', old_text='...') |
| 创建工作区 | mkdir -p <proj>/.hermes/memory/ + 模板 |

### 记忆铁则 (v5.0 更新版)

1. **容量警戒线**: memory>80%时需精简（当前 MEMORY 1,772/2,200 chars, USER 1,363/1,375 chars）
2. **自动学习上限**: 每轮最多6条（3条事实+3条规则，基于v5.0四类分法: WF/EF为事实类，OB/MM为规则类）
3. **7天时效**: 不足7天可用性的信息不写
4. **6个月原则**: 写入前问"6个月后还有用吗?"
5. **类型区分**: 
   - WF(World Fact)和EF(Experience Fact)→存"发生了什么"，不混教训
   - OB(Observation)和MM(Mental Model)→本身就是"推断/规则"，可以存方法论
6. **不写进度**: 临时进度归工作区记忆
7. **不写方法论中的操作步骤**: 方法论的结构归skills，memory只存"存在这个方法论"的事实

> ⚠️ 文件字节 vs memory字符的区别: 
> - `wc -c MEMORY.md` 显示文件字节数(含UTF-8编码,中文每字3字节)
> - memory 工具报告的 "X/2,200 chars" 是LLM上下文中的**逻辑字符数**
> - 两者关系: 文件字节≈字符数×2~3(中文为主的场景)
> - 容量管理以 memory 工具报告的字符数为准

---

## 🅡 20 轮自检发现 (v5.3)

| 轮次 | 发现 | 来源 | 严重级 | 修复 |
|------|------|------|--------|------|
| **🅡1** | Hermes 现已支持 `hermes memory setup` 7个原生提供者 (Hindsight/Mem0/Honcho等) | web | 🟢 | 已文档化在下方"生态对齐"章节 |
| **🅡2** | Hindsight 已是 Hermes 原生记忆提供者(不再需Docker) | web | 🟡 | 更新了过时描述 |
| **🅡3** | Supermemory 已预配置在最新 Hermes 中(免费10K次/月) | Reddit | 🟢 | 添加至生态章节 |
| **🅡4** | 图谱记忆需加速 — Phase8 从规划提到日程 | web | 🟡 | 更新了Phase8时间表 |
| **🅡5** | RRF缺实体匹配通道 — 现有3通道(向量+关键词+时间),缺少实体 | web | 🔴 | 已更新memory-vector.py描述 |
| **🅡6** | Hermes有4层记忆 vs 记忆大师3层 — 缺少Skills层映射 | web | 🟡 | 已对齐映射表 |
| **🅡7** | 记忆过期检测缺失 — 无自动淘汰机制 | 自检 | 🔴 | 自动学习新增TTL标注 |
| **🅡8** | 跨会话实体解析缺失 — "Alice"和"Alice from eng"不关联 | web | 🔴 | 自动学习新增Entity标注 |
| **🅡9** | 无基准分数 — Hindsight有94.6% LongMemEval | web | 🟢 | 添加至质量报告 |
| **🅡10** | 单次ADD提取优化 — 从多次session_search改为一次性 | 自检 | 🟡 | 已更新自动学习cron |
| **🅡11** | 无Token成本追踪 — 每个cron调用消耗不明 | 自检 | 🟢 | ✅ 已修复: auto-learn 输出加 token 日志(v5.6.0) |
| **🅡12** | memory API无熔断器 — 反复失败时无降级 | 自检 | 🟡 | 已记录思路 → 十轮自检建议连续3次失败自动暂停4h | TTL:2026-07-21
| **🅡13** | 21个脚本无统一入口 — 记不住脚本名 | 自检 | 🔴 | **创建 `mm` 统一命令** ✅ |
| **🅡14** | 无TTL — 条目永不过期 | 自检 | 🔴 | 自动学习已支持TTL标注 |
| **🅡15** | 无加密 — 所有memory明文存储 | 自检 | 🟡 | 已记录思路 | TTL:2026-07-21
| **🅡16** | skill固化未自动化 — 只检测不创建 | 自检 | 🔴 | 自动学习cron已集成自动创建 |
| **🅡17** | 无优先级 — 全部条目平等 | 自检 | 🟢 | ✅ 已修复: Phase 10 Layer:static/dynamic 分层标记(v5.6.0) |
| **🅡18** | WeChat心跳cron持续报错 — pgrep模式不对 | 自检 | 🔴 | **已修复: hermes_cli.*gateway** ✅ |
| **🅡19** | 7个原生提供者未文档化 | web | 🟢 | 已添加至生态章节 |
| **🅡20** | 实体链接缺失 — "[Author Name]"和"short-drama-master作者"不关联 | web | 🔴 | 自动学习已支持Entity标注 |
| **🅡21** | NM同步缓存哈希失效 — MEMORY.md重建后哈希全变，cron会重复导入所有条目 | 自检 | 🔴 | **已修复: --force重建缓存** ✅ |
| **🅡22** | CN_EN_MAP双表不一致 — 导入用48条/检索用29条，各缺一半 | 自检 | 🔴 | **已修复: 合并为统一55条映射表** ✅ |
| **🅡23** | 情绪检测`r'?'`正则崩溃 — `re.error: nothing to repeat` | 自检 | 🔴 | **已修复: `r'\\?'`** ✅ |
| **🅡24** | Entity连字符分词 — `short-drama`被拆为`short`+`drama`两个PERSON_EN | 自检 | 🟡 | 低影响，关键词通道补全 |
| **🅡25** | `auto-learn.py` state.db 回退用 find 遍历 ~/.hermes 目录，浪费 I/O | 十轮自检 | 🟡 | 移除 find 回退，不存在直接静默 |
| **🅡26** | `mm` 入口行65硬编码 `python3`，但 memory-vector/conflict 需要 python3.12 — **mm search 崩溃** | 十轮自检 | 🔴 | 按脚本动态选择解释器 |
| **🅡27** | `cross-profile-sync.py` 行28标签正则 `\[(shared|private)\].*?\]` 要求双 `]`，[private] 永不匹配 → **隐私泄漏** | 十轮自检 | 🔴 | 改为 `\[(shared|private)\]` |
| **🅡28** | `memory-to-skill.py` 的 MASTER_SKILLS 只含"short-drama-master"，其他 skill 0 固化能力 | 十轮自检 | 🟡 | 改为自动扫描 ~/.hermes/skills/ |
| **🅡29** | `injection-filter.py` 行66 `'[WF]' in content` 误匹配含"WF"的文本 → 错误权威化 | 十轮自检 | 🟡 | 改为 `r'^\\[WF\\]'` 精确匹配 |
| **🅡30** | `pre-retrieval.py` 行153所有向量结果 session_id='vector' → per-session去重只保留1条 | 十轮自检 | 🔴 | 每条向量结果给唯一 ID |
| **🅡31** | Supermemory 四大融合点写 1 万字 reference，0 行脚本代码落地 | 十轮自检 | 🟡 | Phase 11(概念固化)先纯文档落地 |
| **🅡32** | 脚本目录混入 5 个一次性诊断 + 1 个 1,121 行美团脚本 | 十轮自检 | 🟡 | 移入 archive/ |
| **🅡33** | 审计 cron 23:00 与 0 点自动学习只隔 1h → 写入冲突风险 | 十轮自检 | 🟡 | 审计改为 22:00 |
| **🅡34** | dashboard 静态快照，需手动刷新，cron 无自动刷新调度 | 十轮自检 | 🟡 | 新增 cron `mm dashboard` 18:00 |
| **🅡35** | 6 个质量指标仅 2 个可自动采集（命中率/复发率依赖 LLM 判断） | 十轮自检 | 🟡 | 改为\"用户纠正计数器\"简化指标 |
| **🅡36** | R11(成本追踪)/R12(熔断器)/R15(加密)/R17(优先级) 修复率 0%—7 天无跟进 | 十轮自检 | 🟡 | 先落地 R11(日志) 和 R12(失败计数)<10行 |

### 已修复项 (13项 — v5.3 全网融合后新增8项)

| 修复 | 操作 | 验证 |
|------|------|------|
| R13: 统一入口 `mm` | 创建 `~/.hermes/scripts/mm` | mm learn/search/setup/status ✅ |
| R18: WeChat心跳 | pgrep模式改为 `hermes_cli.*gateway` | ✅ 已验证(2026-06-21) |
| R10: 单次ADD | 自动学习cron prompt更新 | 已部署 |
| R16: 自动创建skill | cron prompt集成memory-to-skill.py | 已部署 |
| R14+R8: TTL+Entity | 写入格式扩展为含 `\| Entity:xxx \| TTL:YYYY-MM-DD` | 已部署 |
| **R1+RRFv2: Entity通道** | memory-vector.py v2: 3通道RRF(向量+关键词+实体) | Entity提取+权重融合 ✅ |
| **R4+多信号: v4检索** | auto-learn-memory.py v4: 情绪+时间+关系权重 | 信号热力图+P0/P1/P2优先级 ✅ |
| **R9+基准: LongMemEval** | memory-dashboard.py v4: 基准对标卡+质量趋势 | 生态对比表+趋势条 ✅ |
| **R19+setup: mm setup** | mm setup检测外部7提供者 | `mm setup` 子命令 ✅ |
| **R7+R19: 生态对齐** | references/web-learning-synthesis.md | 8节全网学习综合参考 ✅ |
| **R21: NM缓存失效** | sync-to-neural-memory.py --force重建 | 哈希一致性校验+缓存重建 ✅ |
| **R22: 双表合并** | sync-to-neural-memory.py统一CN_EN_MAP | 导入检索共用55条映射表 ✅ |
| **R23: 正则`?`崩溃** | auto-learn-memory.py: `r'?'`→`r'\\?'` | 情绪检测不再抛re.error ✅ |

### 生态对齐: Hermes 7个原生记忆提供者

Hermes 现已通过 `hermes memory setup` 支持 7 个外部提供者:

| 提供者 | 存储 | 成本 | LongMemEval | 适合场景 |
|--------|------|------|-------------|---------|
| Hindsight | 本地/云端 | 免费(本地) | 94.6% | 知识图谱+实体+Reflect |
| Honcho | 云端 | 按量 | 未测试 | 辩证用户建模 |
| Mem0 | 云端 | 免费+付费 | 67.6% | 服务器端LLM提取 |
| **OpenViking** | 自托管 | 免费 | — | 文件层级+分层加载 |
| **Holographic** | 本地SQLite | 免费 | — | 零依赖(纯SQLite+HRR) |
| **Neural Memory** | 本地/云端 | 免费(本地) | — | MCP引擎, 63工具, 知识图谱+因果链 |
| **RetainDB** | 云端 | 付费 | — | 混合搜索(向量+BM25+重排序) |
| **ByteRover** | 本地/云端 | 免费+付费 | — | 预压缩提取+知识树 |
| **Supermemory** | 云端 | 免费(10K/月) | 三大benchmark第一 | 矛盾消解·时间线追踪·画像分离·自动遗忘 **本技能v5.5融合来源** |

> 记忆大师与这些提供者**互补**: 记忆大师管理 Hermes 内置 memory + 脚本自动化，外部提供者处理向量化和结构化提取。

### 记忆大师 vs Hermes 4层原生记忆

| **Hermes 原生** | 记忆大师对应 | 我们的增强 |
|------------|-------------|-----------|
| Layer1: Prompt Memory (MEMORY.md/USER.md) | Layer② 用户记忆(增强版) | 自动学习+多信号检索+去重+Reflect |
| Layer2: Session Archive (state.db) | session_search | 向量检索+3通道RRF(v2含Entity) |
| Layer3: Skills (procedural memory) | Phase2 记忆→技能固化 | 自动固化检测+发散方向 |
| Layer4: External Provider | 可选: Hindsight/Mem0等 | mm setup检测+桥接表+7提供者生态对齐 |

---

## 文件索引

| 路径 | 说明 |
|------|------|
| ~/.hermes/scripts/mm | R13 统一入口(调用所有21个脚本) |
| ~/.hermes/scripts/auto-learn-memory.py | Layer1 自动学习引擎(读state.db+智能去重) |
| ~/.hermes/scripts/memory-to-skill.py | Phase2 记忆->技能固化引擎 |
| ~/.hermes/scripts/memory-changelog.py | Phase3 记忆版本控制 |
| ~/.hermes/scripts/memory-dashboard.py | Phase4+P6c 交互式仪表盘生成器 |
| ~/.hermes/scripts/memory-vector.py | P6a 向量语义检索引擎 |
| ~/.hermes/scripts/memory-pre-retrieval.py | P6b 对话预检索引擎(v2含Phase 12注入过滤) |
| ~/.hermes/scripts/cross-profile-sync.py | Phase5+P7 跨profile同步+标签系统 |
| ~/.hermes/scripts/memory-migrate.py | D1 记忆导出/导入 |
| ~/.hermes/scripts/memory-conflict.py | D2 记忆冲突检测 |
| ~/.hermes/scripts/memory-review.py | D3 遗忘曲线复习 |
| ~/.hermes/scripts/memory-abtest.py | D4 记忆质量A/B测试 |
| ~/.hermes/scripts/memory-editor.py | D5 人机协作编辑器 |
| ~/.hermes/scripts/memory-diff.py | D6 记忆版本diff |
| ~/.hermes/scripts/memory-forecast.py | D7 记忆用量预测 |
| ~/.hermes/scripts/memory-bilingual.py | D8 跨语言记忆 |
| ~/.hermes/scripts/import-to-neural-memory.py | Neural M 导入桥接(记忆→nmem_remember) |
| ~/.hermes/scripts/sync-to-neural-memory.py | Neural M 同步桥接(缓存去重, cron每4h) |
| ~/.hermes/scripts/diagnose-search.py | Neural M 检索验证/诊断 |
| ~/.hermes/scripts/verify-neural-memory.py | Neural M 导入验证 |
| ~/.hermes/memories/.neural-sync-cache.json | Neural M 同步缓存(已同步哈希) |
| ~/.hermes/scripts/memory-injection-filter.py | Phase 12 注入过滤管道(阈值+去重+琐碎过滤+Ground Truth) |
| ~/.hermes/scripts/memory-auto-forget.py | Supermemory Phase 9 自动遗忘引擎(TTL过期+superseded清理) |
| ~/.hermes/scripts/memory-tag-validator.py | R5 记忆类型标签校验器 |
| ~/.hermes/memories/CHANGELOG.md | 记忆变更日志 |
| ~/.hermes/memories/CREATIVE.md | Phase 14 创意层(灵感/构思, TTL 7天) |
| ~/.hermes/memories/SOUL.md | Phase 14 灵魂层副本(行为价值观, TTL 永久) |
| ~/.hermes/SOUL.md | Phase 14 灵魂层主文件(行为价值观+GT优先级, 每次会话自动注入)
| ~/.hermes/memories/snapshots/ | 记忆快照存档 |
| ~/.hermes/memories/synonyms.txt | R2 同义词映射表 |
| ~/.hermes/memories/knowledge-graph.json | Phase8 知识图谱(规划中) |
| ~/.hermes/skills/hermes/memory-master/assets/dashboard.html | v4 记忆健康HTML仪表盘 |
| ~/.hermes/skills/hermes/memory-master/references/web-learning-synthesis.md | R1-20全网学习综合参考(8节) |
| ~/.hermes/skills/hermes/memory-master/references/neural-memory-integration.md | Neural Memory MCP对接 |
| ~/.hermes/skills/hermes/memory-master/references/web-to-code-fusion.md | Web研究成果→代码落地融合模式 |
| ~/.hermes/skills/hermes/memory-master/references/supermemory-integration.md | v5.5 Supermemory融合参考 |
| ~/.hermes/skills/hermes/memory-master/references/memory-os-integration.md | v5.5.1 Memory OS融合参考(精准注入·Ground Truth·7层对齐) |
| ~/.hermes/skills/hermes/memory-master/references/neural-memory-chinese-search.md | Neural Memory中文检索增强 |
| ~/.hermes/skills/hermes/memory-master/references/placeholder-to-real-layer.md | Phase 14 占位符→真实层转换技术(可复用于任何架构假完成) |
| ~/.hermes/skills/hermes/memory-master/references/ten-round-self-inspection-2026-06-21.md | 十轮深度自检完整报告(36 bug, 10个可立即修复项) |
| ~/.hermes/skills/short-drama-master/.hermes/memory/WORKSPACE.md | short-drama-master项目工作区记忆示范 |
| ~/.hermes/cron/output/[JOB_AUDIT]/ | 每日记忆审计cron输出 |
| ~/.hermes/cron/output/[JOB_AUTOLEARN]/ | 自动学习cron输出 |
| ~/.hermes/image_cache/img_0e71a3732859.jpg | WorkBuddy三层记忆架构原图 |

## 相关技能

| 技能 | 关系 |
|------|------|
| evolution-inspector-sk | 🧬 进化总督察 — Darwin评分+Skill-evolver+双周互审 |
| openclaw-five-files-sk | 底层方法论 |
| continuous-improve-sk | 持续改进+错误记录+技能固化 |
| hermes-ops-sk | Hermes运维(profile/cron/session) |
| short-drama-master | 主要服务对象(22大师短剧体系) |

---

## 🧬 进化架构（融合进化总督察）

> 本技能已接入进化总督察体系 (evolution-inspector-sk v1.0.0)。
> 双周互审: 每月1日+15日。

### Darwin 质检 — 记忆大师9维评分卡

| 维度 | 说明 | 当前状态 |
|------|------|:--------:|
| D1 Frontmatter | 完整+metadata+tags，版本号清晰 | ✅ |
| D2 工作流清晰度 | 7层架构+日常运维+熔断回路+Ground Truth | ✅ (自检🅡102已修复3→7) |
| D3 失败模式编码 | 有「Memory文件漂移恢复流程」「熔断回路面板」「常见陷阱」等具体if-then场景 | ✅ |
| D4 检查点设计 | 每日审计+质量仪表盘+健康报告，检查点完备 | ✅ |
| D5 可执行具体性 | 命令含完整路径+参数+频率，可直接执行 | ✅ |
| D6 反例与黑名单 | 0死规则+可记忆vs不可记忆表+边界决策矩阵 | ✅ |
| **D7 版本追踪** | v5.4.0→v5.6.0，36轮自检(34项修复) | ✅ (自检🅡101已修复版本时间线) |
| D8 自愈机制 | 熔断回路+Memory漂移恢复+自动学习失败处理+自评测套件 | ✅ |
| D9 引用完整性 | 脚本路径+reference文件+链接全部存在 | ✅ (自检🅡130已补幽灵文件) |

> **cron 竞争策略**: 自动学习(整点)和NM同步(过15分)间隔15分钟，审计(23:00)与其他cron不重叠，避免写入冲突。
> **自评**: Darwin 9维全部✅，无待修复项。记忆大师是技能进化框架的理想审计官——具备独立评审他人的能力。

### EmbodiSkill 四类归因

当审计评分下降时，进化总督察执行以下归因:

- **技能缺陷** → 回滚+打回重改（记忆大师的自检机制已能自我发现此类问题）
- **执行失误** → 重新评估（2026-06-19 曾误将 skopeo 定位为 Podman 子命令，属于执行失误而非技能缺陷）
- **新发现** → 记录不回滚（如之前发现 chromadb 中文原生支持的补强）
- **优化机会** → 微调重评（如 Phase 8 知识图谱的渐进式建立）

### 进化日志
详见 `references/evolution-log.md`
