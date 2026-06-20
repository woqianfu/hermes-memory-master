# 🧠 Memory Master — Highlights

> v5.6.0 | 25 scripts · 6 cron jobs · 7-layer architecture · 4 external integrations

---

## Core Concepts 核心理念

1. **Three layers → seven layers**: evolved from WorkBuddy's 3-layer architecture to Memory OS's 7-layer alignment (Session→Facts→User→Creative→Soul→Rules→Ground Truth)
2. **Auto-learning**: every 4h the cron scans sessions, extracts WF/EF/OB/MM memories, and writes them automatically — no manual entry needed
3. **Memory is not RAG**: memory tracks who you are and what changed; RAG searches conversation text. They are complementary, not interchangeable.
4. **Forgetting is a feature**: TTL-based auto-forget runs daily at 20:30 (no_agent, zero LLM cost). `mm forget --dry-run` previews what will be cleaned.
5. **Remember correctly > remember more**: one wrong memory is worse than no memory. Ground Truth priority (authoritative > normal > fallback) ensures high-confidence facts get used directly.
6. **Do not ask "which search method"**: the agent auto-selects the retrieval path based on query language and type. This is enforced as a hard rule.
7. **Soul layer as behavior constitution**: SOUL.md defines 5 core values (remember correctly, forget > fabricate, remember to not ask again, automate > manual, project isolation) that constrain all memory decisions.
8. **Zero Docker**: all 25 scripts are pure Python with chromadb. No containers, no docker-compose.

---

## Core Modules 核心模块

9. **auto-learn-memory.py** (387 lines): session scanning → multi-signal detection (emotion/time/relationship) → change verb detection (19 verbs: 搬到/改为/换成/...) → time sensitivity detection → LLM instruction output with Phase 9/10/11 markers
10. **memory-vector.py** (466 lines): chromadb vector storage, 3-channel RRF (vector + keyword + Entity), Chinese/English entity extraction, external provider detection
11. **memory-pre-retrieval.py** (215 lines): synonym expansion + vector search + SOUL.md context injection + Phase 12 injection filter pipeline
12. **memory-injection-filter.py** (128 lines): 4-stage filter — relevance threshold (≥0.3) → trivial message removal (好的/谢谢/👌) → per-session dedup → GT priority marking
13. **memory-auto-forget.py** (118 lines): TTL expiry scanner + superseded (>30d) cleanup + Layer:dynamic default TTL assignment. Runs daily 20:30 via no_agent cron.
14. **mm** (141 lines): unified entry point for 23 subcommands. Automatically selects python3.12 for chromadb-dependent scripts (memory-vector.py, memory-conflict.py) and python3 for others.
15. **memory-self-test.py** (145 lines): 6-dimension health check — Python env, vector DB, syntax, cron health, Neural Memory status, SOUL.md integrity
16. **memory-workspace.py** (42 lines): auto-generates WORKSPACE.md with current date for new projects
17. **memory-dashboard.py** (389 lines): generates a complete HTML dashboard with 7 browse tabs (memory browser, semantic search, quality benchmark, cron health, changelog, workspace, statistics)
18. **memory-to-skill.py** (149 lines): scans memory for ≥3x repeated patterns, auto-discovers all installed skills via `~/.hermes/skills/` walk, suggests skill creation
19. **cross-profile-sync.py** (185 lines): cross-profile sync with [shared]/[private] tag support. Corrected regex in v5.6.0 to properly detect private tags.
20. **sync-to-neural-memory.py** (319 lines): bidirectional sync with Neural Memory MCP, 55-entry CN_EN_MAP for Chinese-English keyword anchoring, hash-based dedup cache
21. **memory-changelog.py** (126 lines): memory version control, CHANGELOG.md + snapshot directory, rollback support
22. **memory-conflict.py** (168 lines): contradiction detection between memory entries, links to auto-resolve

---

## Rules 规则体系

23. **6-Month Principle (highest)**: don't write if it won't be useful in 6 months
24. **7-Day Principle (second)**: don't write if <7d usefulness (even if it passes 6-month)
25. **Type Separation**: WF/EF store "what happened"; OB/MM store "inferred patterns/rules"
26. **No Progress in Memory**: task progress goes to Layer 3 workspace, not Layer 2 memory
27. **Auto-Learn Cap**: WF+EF ≤3/round, OB+MM ≤3/round, total ≤6/round
28. **No Methodology in Memory**: methodology structure belongs to skills
29. **GT Tag Required**: every auto-learn output must begin with [WF]/[EF]/[OB]/[MM]
30. **Layer Tag**: auto-detect time sensitivity → mark Layer:static (permanent) or Layer:dynamic (TTL 30d)
31. **Phase 9 Priority**: change verb detection output > regular memory extraction. If user says "搬到杭州", mark old location as superseded rather than overwrite.
32. **Retrieval Path Auto-Select**: Chinese → session_search + mm search; English → Neural Memory + session_search; Fuzzy → spreading activation
33. **Memory Drift Recovery**: 4-step process (backup → create empty → restore via memory tool → verify) for when MEMORY.md hash becomes out of sync

---

## Script Tooling 配套工具

34. `mm` → 23 subcommands: learn, search, index, status, setup, nm-sync, nm-search, diff, forecast, review, conflict, export, import, editor, bilingual, abtest, dashboard, sync, skill, validate, changelog, forget, workspace, self-test
35. All scripts live in `~/.hermes/scripts/` — cron workdir compatible
36. Auto-python3.12 for chromadb scripts; python3 for all others (v5.6.0 fix from 10-round self-inspection)

---

## Methodology 设计决策

37. **Zero Docker**: chromadb + sentence-transformers over Qdrant/Redis/Docker stack. Reduced operational complexity at the cost of embedding quality.
38. **LLM-driven auto-learn over rule-based extraction**: auto-learn presents structured session data + decision trees to the LLM, letting it judge what to write. More flexible than regex rules, more token-costly.
39. **Single-thread cron design**: auto-learn (:00), NM sync (:15), auto-forget (20:30), dashboard (18:00), audit (22:00) — all time-staggered to avoid state.db contention.
40. **Circuit breaker over fail-loud**: when memory is full, auto-forget runs first before alerting the user. Step 0 deletes the oldest stale entry, steps 1-2 do progressive cleanup, step 3 only alerts as last resort.
41. **Simplified timeline over full version history**: instead of maintaining a full timeline array (Supermemory approach), detection of change verbs appends `| superseded:YYYY-MM-DD` — storing the superseded date inline is 90% of the value at 10% of the complexity.

---

## Benchmarks 量化基准

42. Vector DB: 97 entries, 100% entity-tagged (as of v5.6.0)
43. chromadb 1.5.2 + all-MiniLM-L6-v2 embedding model
44. Neural Memory MCP: 63 tools, 454ms average response time
45. RRF formula: `score = 1/(60+vector_rank) + keyword_match*0.5 + time_boost`
46. Memory tool capacity: 2,200 chars (memory) + 1,375 chars (user) = 3,575 chars total
47. Auto-learn cron: every 4h, limit 10 sessions, max 6 entries/round
48. SOUL.md: 107 lines, 5 behavior creeds, injected every session

---

## Engineering Delivery 工程交付

49. **25 scripts** totaling ~5,000+ lines of Python
50. **6 active cron jobs**: auto-learn, NM sync, auto-forget, dashboard refresh, daily audit, plus optional
51. **7-layer architecture**: files in ~/.hermes/memories/ (MEMORY, USER, CREATIVE, SOUL) + skills/ + SOUL.md
52. **4 external integrations**: Hindsight (retrieval), MemOS (dedup), Supermemory (timeline), Memory OS (7-layer alignment)
53. **10-round self-inspection**: found and fixed 34 bugs in v5.6.0, including 2 crash-level bugs (mm python3/python3.12 dispatch, per-session dedup collision)
54. **Self-test suite**: 6-dimension health check via `mm self-test [--quick]`

---

## Credits

- Built for [Hermes Agent](https://github.com/NousResearch/hermes-agent) by Nous Research
- Inspired by WorkBuddy · Hindsight · MemOS · Supermemory · Memory OS
