# Neural Memory 中文检索增强方案

## 问题
Neural Memory (v4.58.0) 的 `nmem_recall` 搜索引擎不支持 CJK 分词。
纯中文查询("微信规则" "short-drama-master")返回 `No relevant memories found.`
但英文查询("WeChat" "short-drama")和带英文锚点的查询正常命中。

## 根因
底层 embedding/BM25 引擎只识别拉丁字母 token 边界，中文连续字符串不做分词。

## 解决方案: 英文锚点策略 (两腿并行)

### 腿1: 导入时追加 [EN: ...] 附录
`sync-to-neural-memory.py` 在每条中文内容末尾自动追加 `[EN: weixin, WeChat, sync, ...]`
由 `CN_EN_MAP` (约 30+ 条映射) 驱动。

### 腿2: 查询时自动增强
`mm nm-search "微信规则"` 自动扩展为 `"微信规则 rules rule weixin WeChat"`
让引擎能通过拉丁字母 token 命中内容。

## 效果验证 (2026-06-18)

| 原始查询 | 增强后 | 结果 |
|---------|--------|------|
| 微信规则 | 微信规则 rules rule weixin WeChat | ✅ 命中 5+ 条 |
| short-drama-master | short-drama-master short-drama ... duanju-master | ✅ 命中[Author Name]profile |
| 三层记忆架构 | 三层记忆架构 memory architecture ... | ✅ 命中 |
| 安全配置 | 安全配置 security config configuration | ✅ 命中 |
| [Author Name] | [Author Name] wangyan woqianfu | ✅ 命中 |
| 同步到微信 | 同步到微信 weixin WeChat sync synchronize | ✅ 命中 |

## CN_EN_MAP 完整列表 (sync-to-neural-memory.py)

```
CN_EN_MAP_SEARCH = {
    "微信": "weixin WeChat",
    "短剧": "short-drama",
    "short-drama-master": "short-drama-master duanju-master",
    "记忆": "memory",
    "规则": "rules rule",
    "同步": "sync synchronize",
    "架构": "architecture",
    "三层记忆": "three-layer memory",
    "[Author Name]": "wangyan woqianfu",
    "铁律": "iron-rule hard-rule",
    "配置": "config configuration",
    "安全": "security",
    "网关": "gateway",
    "仪表盘": "dashboard",
    "向量": "vector",
    "知识图谱": "knowledge-graph",
    "部署": "deploy setup",
    "更新": "update upgrade",
    "版本": "version release",
    "日报": "daily-report newsletter",
    "产业": "industry",
    "项目": "project task",
    "角色": "character",
    "造型": "styling character-design",
    "脚本": "script screenplay",
    "画质": "video-quality resolution",
    "font": "font",
    "aesthetic": "aesthetic design-style",
    "Neural": "neural-memory nmem",
}
```

## 扩展指南
- 遇到查不到的词 → 往 `CN_EN_MAP_SEARCH` 加一条映射即可
- 映射只在查询时生效，不需要重新导入记忆
- 如果能找到 Neural Memory 支持中文分词的后端配置，可以完全取消此方案
