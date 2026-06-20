# Memory 文件漂移恢复流程

## 症状

memory 工具写入时提示：

```
Refusing to write MEMORY.md: file on disk has content that wouldn't
round-trip through the memory tool (likely added by the patch tool,
a shell append, a manual edit, or a concurrent session).
```

## 根因

`patch` 工具、`write_file` 或外部编辑器直接修改了 MEMORY.md/USER.md 文件，
导致 memory 工具的内部哈希校验失效。这是 Hermes 的安全机制，防止数据丢失。

## 标准恢复流程

### Step 1: 备份 + 清理

```bash
mv ~/.hermes/memories/MEMORY.md ~/.hermes/memories/MEMORY.md.bak.drift
touch ~/.hermes/memories/MEMORY.md
```

同样处理 USER.md 如果有类似错误。

### Step 2: 从备份逐条恢复

通过 memory 工具逐条添加：

```python
memory(action='add', target='memory', content='**[WF] 标题** — 内容')
```

关注容量上限（MEMORY ~2,200 chars, USER ~1,375 chars）。
空间不够时先合并同类项。

### Step 3: 清理备份

```bash
rm ~/.hermes/memories/MEMORY.md.bak.drift
```

## 预防措施

- ❌ 不要直接 `patch` 或 `write_file` 修改 MEMORY.md/USER.md
- ✅ 所有写入走 memory 工具（add/replace/remove）
- ⚠️ 如果必须文件级操作，先备份，操作后立即用 memory 工具重建

## 影响

漂移后：
1. memory 工具可正常读文件，但无法写
2. 自动学习 cron 会运行但写不了新条目（不报错，无收获）
3. 每日审计可以读但写不了清理结果
4. Neural Memory 同步缓存哈希失效 → 下次 cron 重复全量导入
