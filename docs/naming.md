# 命名

## 结论

**notae** — 拉丁语 `nota`（标记、注解）的复数。

中世纪抄书人写在文本旁的解释性标记就叫 notae，罗马速记系统叫
notae Tironianae。词义本身就是"保存在原文旁边的他人解释"，
跟本产品做的事是同一件——这是选它的主要理由。

- 品牌名 = 二进制名 = `notae`，5 个字母，不需要 alias。
- `explain` 保留给核心子命令：`notae explain <ref>`。
  用户每天敲的那个词就是产品在做的事，产品名负责辨识度，两件事不互相牺牲。
- 中文念"诺泰"，无发音障碍。
- 避开了「笔记应用」品类陷阱（Notion / Obsidian / Logseq / flomo），
  同时保住 note 的词源血脉。

## 占用情况

查询时间 2026-09-29。crates.io 对所有查询返回 403（限流，非占用信号），该列已弃用。

| 名字 | npm | PyPI | GitHub 组织 | 最大同名仓库 |
|---|---|---|---|---|
| **notae** | 空 | 空 | 占 | `KDE/notae` (3★) |
| understand-note | 空 | 空 | 空 | `jurihock/oh-my-notes` 类 ≤3★ |
| oh-my-note | 空 | 空 | 空 | 23 个同名仓库，最高 3★ |
| explain-cli | 空 | 占 | 空 | — |
| relume | 空 | 空 | 占 | 25★（Figma kit） |
| glosa | 空 | 占 | 占 | `carpentries/glosario` (133★) |

`notae` 的 GitHub 组织名被占，但最大撞名仓库只有 3 星，实际无竞争。
npm 和 PyPI 都空着，发包不受限。

## 硬排除

以下候选有商业占用或不可修复的语义撞车：

| 名字 | 原因 |
|---|---|
| `explain` | **SQL 关键字**，Postgres/MySQL/SQLite 都用它看查询计划，终端语境下歧义不可修复。npm/PyPI/组织三占。32,050 个同名仓库，头部 `explainshell` 14k★。且它已是本项目核心子命令名，用作产品名会锁死 CLI 语法 |
| `understand` | `Understand-Anything` 84.6k★，语义空间已满 |
| `restate` | `restatedev/restate` 4.5k★，活跃的 durable execution 平台 |
| `defog` | `defog-ai` 是同行业 AI 公司，`sqlcoder` 4k★ |
| `retell` | RetellAI，融过资的语音 AI 公司 |
| `unpack` | 开发语境下意为解压，与本产品的存档功能语义撞车 |
| `unfog` | `soywod/unfog` 205★ CLI 待办工具，同赛道 |
| `gloss` | 语义最准（philology 里 gloss 就是难懂段落旁的解释），但 npm/PyPI/组织全占，且 `charmbracelet/lipgloss` 使其在 CLI 圈显得跟风 |
| `limpid` / `scholia` | 语义对，但中文圈发音困难。scholia 另有 `WDscholia` 257★ |
| `nota`（单数） | `notable/notable` 23k★ 笔记应用，正面撞车 |

## 选型标准

按权重排序，供后续子品牌（如文档站、包名）沿用：

1. **指向"理解"而非"存储"。** 价值在重讲，不在存笔记。叫 note 会把卖点埋进
   全世界最饱和的品类。
2. **独特而非描述性。** 立住的工具不描述自己：rg、fd、bat、jq、git、curl、zed。
   描述性复合名（`ai-note-taker`、`understand-note`）是 1 star 仓库的命名签名。
3. **无商业占用。** 避免两年后被迫改名。
4. **中文圈可发音。** 目标用户含中文开发者。
5. **短到能直接当二进制。** 否则必须 alias，品牌在日常使用中隐身。
   备用方案是品牌名与二进制名分离（ripgrep→`rg` 先例），notae 不需要。
