# Web 研究成果 → 代码落地融合模式
## 适用于: 将"已文档化但未编码"的知识转化为可操作代码

## 核心问题
AI 很容易把全网搜索到的信息**写进文档**但不**写进代码**。
知识停留在 SKILL.md 的表格里，不改变运行时行为。

## 融合三步法

### Step 1: 评估差距
```markdown
| 来源 | 学到什么 | 当前状态 | 应该是什么 |
|------|---------|---------|-----------|
| Vectorize.io | 4层架构 | 文档一行 | 运行时 `mm setup` 输出 |
| Hindsight | RRF+Entity | 记录为"P8待办" | 实际 Entity 通道代码 |
| Mem0      | 多信号检索 | 一行文字 | auto-learn.py v4 算法 |
```

### Step 2: 判断融合类型
| 类型 | 例子 | 落地动作 |
|------|------|---------|
| **算法变更** | RRF从2通道→3通道 | 改 memory-vector.py 的排序公式 |
| **新子命令** | mm setup | 改 mm 脚本 + 加 memory-vector.py 函数 |
| **CRON升级** | 多信号检索 | 改 auto-learn-memory.py 的输出格式 |
| **仪表盘** | LongMemEval基线 | 改 memory-dashboard.py 的HTML模板 |
| **参考资料** | 8节综合参考 | references/web-learning-synthesis.md |

### Step 3: 端到端验证
每项融合必须跑真实输出证明生效：
```bash
# Entity 通道验证
mm search "[Author Name] short-drama-master" → 看到 [E:100%] 标记

# mm setup 验证
mm setup → 看到 Neural Memory MCP 状态 + 4层对齐表

# 多信号验证
python3 auto-learn-memory.py | head -5 → 看到 "v4 多信号" 标题

# 仪表盘验证
python3 memory-dashboard.py → dashboard.html 更新
```

## 戒律
- ❌ **不**把知识只写入文档就标记完成
- ❌ **不**报告"已记录思路"当最终状态
- ✅ **必须**产生可运行的代码变更 + 真实输出验证
