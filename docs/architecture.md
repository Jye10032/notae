# 架构设计

> 状态：草案，讨论中
> 命名已定：**notae**。取拉丁语 `nota` 的复数——中世纪抄书人写在文本旁的
> 解释性标记就叫 notae。选型过程与占用数据见 [naming.md](./naming.md)。

## 1. 这是什么

一个本地优先的 CLI 工具：把你看不懂的材料交给它，它按你的思维习惯重新组织语言讲一遍，
并把解读、当时的困惑点、原文存档三者绑在一起沉淀下来。

**不是笔记应用。** 笔记是副产品。核心价值是"按个人学习习惯重讲一遍"这个动作，
以及三个月后回看时能一键跳回当时看不懂的原文、并看到当时的困惑点。

**不是 RAG 问答。** 不做"把你的库全读一遍然后回答问题"。是你指着哪儿，它讲哪儿。

### 形态决策

- **本地优先**：所有内容存在用户自己的磁盘上，纯 markdown + frontmatter。
- **可移植**：删掉本工具，笔记仍然是笔记，Obsidian 能直接打开。
  这是"不怕上传到别人平台后无法复用"这个顾虑的正面回答。
- **CLI**：有真实文件系统和真实工具链（ripgrep、sqlite、git）可用，
  很多事情用代码做比用 LLM 做又快又免费。

## 2. 核心设计原则

按重要性排序。后面所有具体设计都是这五条的推论。

### 2.1 Pull 而不是 Push

不做"导入后先全读一遍"，只做"你指着哪儿我讲哪儿"。

理由：用户不理解的东西通常很短——一段话、一个公式、一个函数、论文里的某一节。
真正烧 token 的不是解读，是围绕它的基础设施。Pull 模式同时解决三件事：
token 省了、首次体验快了（几秒看到一段有用的解释，而不是等五分钟 ingest）、
亮点也更集中。

### 2.2 存档免费，理解付费

`add` 命令一个 LLM 都不调。抓取、正文提取、切块、算 hash、生成骨架，全部用代码做。

理由：存档是零成本操作，用户才敢往里扔东西。"导入要花钱"会直接杀死使用频率，
而使用频率是这个产品唯一的生死线。

### 2.3 能用代码就不用模型，能用小模型就不用大模型

- 代码做：切块、TOC 提取、hash、去重、链接图维护、index 生成、frontmatter 读写、token 记账。
- Haiku 做：分类打标签、triage 哪几节是难点、概念页的机械合并。
- Opus 做：**只有真正的重讲**。

理由：亮点全在重讲那一步，那里不能省；其余地方省下来的钱都补贴到那一步。

### 2.4 所有权边界

工具区域和用户区域互不越界。这是防止"追加时混乱"的唯一有效手段——
不是把目录分得更细，而是划清谁能写哪里。详见 §3.2。

### 2.5 manifest 是缓存，不是权威

`.notae/manifest.db` 必须能从文件完全重建（`notae reindex`）。删掉它不丢任何数据。

理由：这是 §1 里"可移植"承诺的技术保证。同时它让
"哪些笔记过期了""这个 chunk 解读过吗""这个月花了多少 token"
这类查询用 SQL 零 token 完成，而不是靠扫文件或者问模型。

## 3. 目录规范

### 3.1 结构

```
vault/
  .notae/                   # 工具私有，用户不碰
    manifest.db             # sqlite：sources / chunks / notes / links / hashes / token 账本
    log.jsonl               # append-only 事件流，可 grep
    cache/                  # 骨架、embedding（可选）
    config.toml             # 模型选择、预算上限、抓取策略

  profile/                  # 用户拥有
    learning-profile.md     # 学习习惯画像，人机共写
    conventions.md          # schema，等价于 CLAUDE.md

  sources/                  # 写入后不可变
    2026/09/<slug>/
      raw.md                # 正文，纯文本
      meta.yaml             # 来源、抓取时间、hash、chunk 数
      skeleton.yaml         # 代码生成的结构骨架（标题树 + 每块首句 + token 数）
      assets/               # 图片等本地附件

  notes/                    # 工具生成
    <slug>/
      explain.md            # 解读正文
      confusion.md          # 当时的困惑点 + 引用的 chunk id

  concepts/                 # 跨源的概念页（v2，MVP 不做）
  inbox/                    # 用户手动扔文件的地方，随便建子目录
  index.md                  # 生成物，一行一页
```

### 3.2 所有权边界

| 目录 | 谁写 | 可变性 | 说明 |
|---|---|---|---|
| `.notae/` | 工具 | 可变 | 用户不应手改；删掉可用 `reindex` 重建 |
| `profile/` | 用户 + 工具 | 可变 | 工具只在 `profile learn` 时追加，不删用户写的内容 |
| `sources/` | 工具（仅 add 时） | **写入后不可变** | 之后永不修改，这是引用稳定性的前提 |
| `notes/` | 工具 | 可变 | 重新 explain 会覆盖，历史交给 git |
| `concepts/` | 工具 | 可变 | |
| `inbox/` | 用户 | 可变 | 工具只读，不改名、不移动 |
| `index.md` | 工具 | 生成物 | 手改会被覆盖 |

两条硬规则：

1. **工具只写 `.notae/`、`notes/`、`concepts/`、`index.md`。**
   用户建的任何文件夹，工具不改名、不移动、不删除。
   需要纳管时通过 `notae adopt <path>` 显式操作。
2. **`sources/` 写入即冻结。** 重新抓取同一 URL 生成新的 slug（带日期后缀），
   不覆盖旧的。这样 chunk id 永久有效，三个月前的笔记引用不会失效。

这两条撑到 500 个源不会乱。

## 4. 文件格式约定

### 4.1 `sources/<slug>/meta.yaml`

```yaml
id: 2026/09/attention-is-all-you-need
title: Attention Is All You Need
origin: https://arxiv.org/abs/1706.03762
kind: paper            # paper | article | code | book | paste
fetched: 2026-09-29T14:10:00+08:00
content_hash: sha256:9f2a...
chunks: 42
tokens_est: 15200
```

### 4.2 `sources/<slug>/skeleton.yaml`

**代码生成，不调模型。** 这是省 token 的关键载体：
骨架进 context（一篇 20 页论文大概 300 token），模型再按需取具体块。

```yaml
chunks:
  - id: c01
    path: "1 Introduction"
    lead: "Recurrent neural networks, long short-term memory..."
    tokens: 312
    hash: sha256:1a2b...
  - id: c07
    path: "3.2.1 Scaled Dot-Product Attention"
    lead: "We call our particular attention Scaled Dot-Product..."
    tokens: 285
    hash: sha256:4c5d...
```

chunk id 按顺序稳定分配（`c01`..`cNN`），带 hash 以便检测原文漂移。

### 4.3 `notes/<slug>/explain.md`

```yaml
---
source: 2026/09/attention-is-all-you-need
chunks: [c07, c08]          # 本次解读实际引用的原文块
profile_version: 3          # 生成时用的画像版本，画像更新后可据此标记过期
model: claude-opus-5
created: 2026-09-29T14:22:00+08:00
tokens: { in: 2431, out: 1180, cached: 890 }
---
```

### 4.4 `notes/<slug>/confusion.md`

这个文件是产品差异化的落点，成本几乎为零——它是 explain 的副产品。

```yaml
---
asked: "为什么 QK^T 要除以 sqrt(d_k)"
chunks: [c07]
resolved: true
revisit_after: 2026-10-29    # 间隔重复的钩子
---
```

`notae recall` 读这两个 frontmatter，用代码把原文段落拼回来并排展示。
零额外 token。

## 5. Token 策略

### 5.1 成本分布

| 环节 | 成本特征 | 对策 |
|---|---|---|
| ingest 存档 | 与源大小成正比 | 全部用代码做，**零 token** |
| explain 解读 | 与"看不懂的那一小段"成正比 | 唯一花钱的地方，不省 |
| 追问迭代 | 低，只要 context 里只留相关块 | 画像走 prompt cache |
| query 检索 | 天然随库增长 | 本地检索选候选，只给模型 3–5 页 |
| lint 维护 | naive 实现近似二次增长 | 只查 dirty set + 一跳邻居 |

### 5.2 具体手段

1. **给模型工具，而不是给它正文。**
   骨架进 context，暴露 `read_chunk(source, chunk_id)` 让模型自己决定读哪几块。
   长文省 85% 以上，且解读质量不降——它本来也只需要那几段。
2. **本地检索前置。** 用 ripgrep / sqlite FTS 在本地筛出候选页，
   而不是把 `index.md` 整个塞进 context。这是 CLI 形态相对纯 markdown 约定的主要优势。
3. **画像做成 prompt cache 前缀。** `learning-profile.md` + `conventions.md` 跨调用稳定，
   放在缓存前缀里按折扣计费。注意默认 TTL 只有 5 分钟（有 1 小时选项），
   所以它在"一次连续深挖"里省得多，跨天帮助有限——
   交互设计上应鼓励单次深挖而非零散提问。
4. **hash 级幂等。** 解读过的 chunk 不再付第二次钱，换个问法追问只付新增部分。
5. **模型分层。** 见 §2.3。
6. **增量 lint。** dirty set 记在 manifest 里，全库扫描永不默认执行。
7. **token 账本。** 每次调用写入 manifest，`notae cost` 可查。
   预算上限写在 `config.toml`，超了先问再跑。

### 5.3 估算示例

一篇 20 页论文约 15k token。以下是估算，**未实测**：

- 朴素做法：每次提问把全文喂给 Opus → 15k 输入 × N 次。
- 本设计：存档 0 token；骨架用代码生成；用户问第 3 节 →
  1.5k 原文块 + 1k 画像（走缓存）→ 约 2.5k。
- 首次约省 85%，后续因画像缓存 + chunk 复用可到 95%。

这组数字需要在真实材料上验证，是 MVP 阶段要收的第一个指标。

## 6. 命令面

### MVP：三个命令

```bash
notae add <url|file|->                # 免费存档 + 生成骨架
notae explain <ref> --confused "…"    # 核心，贵模型，唯一花钱的地方
notae recall <note>                   # 笔记 ↔ 原文双向跳转，零 token
```

`--confused` 是可选的。不给就让模型自己 triage 难点（Haiku 一次）；
给了就直接定位，更省也更准。

### v2 及以后

```bash
notae init                            # 初始化 vault
notae ask "…"                         # 跨源提问，本地检索 + 引用
notae profile learn                   # 从反馈更新画像
notae profile bootstrap               # 冷启动：从用户自己的讲解里提取画像
notae adopt <path>                    # 纳管用户手建的目录
notae reindex                         # 从文件重建 manifest
notae lint [--incremental]            # 查矛盾、过期、孤立页
notae cost [--month]                  # token 账本
```

## 7. 画像的冷启动

单独拎出来，因为这是最容易做塌的地方。

如果"学习习惯"只是一段前缀（"多用类比、少用公式"），用户感觉不到差别，
产品就没有护城河。

**方案**：onboarding 时让用户用自己的话讲 3 个他已经懂得很透的东西，
然后一次 Opus 调用从中提取他的表达模式——先讲结论还是先铺背景、
类比取自什么领域、能容忍多少抽象、习惯什么颗粒度的例子。

成本约几千 token，但产出的画像是真正个人化的，
而且第一次 `explain` 用户就能感觉到"这像我自己说话"。

持续更新渠道：参考费曼法那套"你先讲、AI 后判"的交互，
每次用户复述都是一次画像信号。

## 8. 待定问题

- [x] ~~命名~~ → **notae**，见 [naming.md](./naming.md)。
      `explain` 保留给核心子命令，不占产品名。
- [ ] 画像的 schema：自由文本 markdown 还是结构化字段？
      自由文本更灵活但难做版本比对，结构化反之。
- [ ] `profile_version` 变更后，旧笔记标记为过期还是静默保留？
- [ ] 切块策略：按标题层级、按 token 窗口、还是混合？代码类材料要不要按 AST？
- [ ] 抓取用什么：readability 类库够不够，还是需要无头浏览器兜底？
- [ ] 间隔重复（`revisit_after`）要不要进 MVP？倾向不进，先验证核心闭环。

## 9. 下一步

先落地 `add` + `explain` + `recall` 三个命令加本目录规范，
不做 `concepts/`、lint、embedding。用真实材料跑一周，
重点验证两件事：§5.3 的 token 估算是否成立，以及画像那层到底立不立得住。

真正的设计约束会在那一周暴露出来，其余设计都等它。




