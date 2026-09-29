# notae

把看不懂的材料交给它，它按你的思维习惯重新组织语言讲一遍，
并把**解读**、**当时的困惑点**、**原文存档**三者绑在一起沉淀下来。

> `notae` 是拉丁语 `nota` 的复数——中世纪抄书人写在文本旁的解释性标记。

```bash
notae add https://arxiv.org/abs/1706.03762
notae explain attention --confused "为什么 QK^T 要除以 sqrt(d_k)"
notae recall attention          # 三个月后回看：解读 + 当时的困惑 + 原文并排
```

## 这不是什么

- **不是笔记应用。** 笔记是副产品，核心价值是「按个人学习习惯重讲一遍」这个动作。
- **不是 RAG 问答。** 不做"把你的库全读一遍然后回答问题"，是你指着哪儿它讲哪儿。

## 设计取向

- **本地优先。** 内容存在你自己的磁盘上，纯 Markdown + frontmatter，Obsidian 能直接打开。
  删掉本工具，笔记仍然是笔记。
- **存档免费，理解付费。** `add` 不调用任何 LLM——抓取、切块、生成骨架全部用代码完成。
- **能用代码就不用模型，能用小模型就不用大模型。** 省下来的预算全部补贴到「重讲」那一步。

## 状态

设计阶段。MVP 范围是 `add` + `explain` + `recall` 三个命令。

- [架构设计](docs/architecture.md) — 目录规范、文件格式、token 策略
- [命名](docs/naming.md) — 选型标准与占用数据

## License

MIT
