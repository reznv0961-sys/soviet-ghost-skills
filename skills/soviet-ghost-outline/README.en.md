# soviet-ghost-outline（中文说明）

《苏联亡灵局》专用叙事结构 skill。它从 `novel-outline` 独立复制而来，**不修改原 skill**；保留结构化大纲、角色分档、场景控制、分批写集、质量门、报告与资产统计，将通用“爽点”改为必须由用户确认的叙事回报（payoff）。

整体调性冷峻、克制、官僚荒诞、历史错位，并带有政治寓言意味。重要 payoff 必须先提出候选方案，再逐项等待用户确认；尚未解决的历史事实会阻止 `full` 阶段校验通过。

## 工作流程

1. 收集集数、时长、题材、偏好、原稿和用户确认的 canon。
2. 长文本按章节分卷；短文本直接进入骨架设计。
3. 先起草改编说明、人物、场景和 payoff 候选，不写分集。
4. 展示每项 payoff 的类型、level、ontology、setup、payoff、theme 和风险，暂停等待用户明确确认。
5. 对依赖真实历史的内容建立历史研究暂停点，确认前暂停。
6. 确认 payoff 后运行 `validate --stage beats`，再请用户拍板骨架。
7. 每批最多写 10 集，并将 hook 与 payoff 分开记录。
8. 运行 full 校验、修复违规，再生成报告和资产清单。

详细规则见 [SKILL.md](SKILL.md) 与 `references/`。

## 命令行

需要 Node.js 18+，不依赖 npm 套件：

```bash
node scripts/soviet-ghost-outline.mjs chunk book.txt workdir
node scripts/soviet-ghost-outline.mjs validate outline.json --stage skeleton
node scripts/soviet-ghost-outline.mjs validate outline.json --stage beats
node scripts/soviet-ghost-outline.mjs validate outline.json --stage full
node scripts/soviet-ghost-outline.mjs checkup outline.json
node scripts/soviet-ghost-outline.mjs render outline.json --html
node scripts/soviet-ghost-outline.mjs render outline.json --html --lang en
node scripts/soviet-ghost-outline.mjs assets outline.json
node scripts/selftest.mjs
```

`params.thresholds.maxPayoffGap` 默认是 3 集。保留原有报告布局、离线渲染、体检与资产汇总。调性漂移扫描只发出警告，不自动判定失败。

## 苏联笑话参考

所有苏联笑话参考内容只保存在工作区根目录的 [`soviet-jokes.md`](../../../soviet-jokes.md)。本 skill 与其他参考文件不复制笑话原文或逐条分析；只提炼抽象机制，并据此创作原创情节。

## 主要文件

- `SKILL.md`：Agent 工作流程与强制暂停点
- `scripts/soviet-ghost-outline.mjs`：分卷、校验、体检、渲染和资产汇总
- `scripts/selftest.mjs`：确定性自测，不调用模型
- `references/schema.md`：大纲数据结构
- `references/payoff-taxonomy.md`、`payoff-ontology.md`：固定类型与因果结构
- `references/tone-bible.md`、`historical-research.md`、`canon-rules.md`、`anti-patterns.md`：项目规范
- `examples/ep01-outline.json`：EP01 验收样例

通用原版仍独立保存在 `skills/novel-outline/`。
