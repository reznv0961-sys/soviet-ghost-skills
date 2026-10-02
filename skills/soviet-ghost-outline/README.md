# soviet-ghost-outline

《苏联亡灵局》专用叙事结构技能。它从 `novel-outline` 复制并演化，**不修改原技能**；保留结构化大纲、角色分档、场景控制、分批写集、质量门、报告与资产统计，把“爽点”改为必须逐项确认的叙事回报（payoff）。

核心语调：冷峻、克制、官僚荒诞、历史错位、政治寓言。重要叙事回报必须先给用户候选方案并暂停；依赖真实历史的设定必须先核实，未解决时不能通过正式 `full` 校验。

## 工作流程

1. 收集集数、时长、题材和偏好；读取原稿与用户确认的 canon。
2. 长文本按章节分卷，短文本直接进入骨架。
3. 先产出改编说明、人物、场景和叙事回报候选，不写分集。
4. 展示每项 payoff 的类型、level、ontology、setup、payoff、theme 和风险，暂停等待用户逐项确认。
5. 对依赖真实历史的内容建立历史研究暂停点；未确认前暂停。
6. 确认 payoff 后运行 `validate --stage beats`，再由用户拍板骨架。
7. 按每批最多 10 集生成分集梗概，hook 与 payoff 分开记录。
8. 运行 full 校验并修复违规，再渲染报告与资产清单。

详细规则见 [SKILL.md](SKILL.md) 与 `references/`。

## CLI

需要 Node.js 18+，无 npm 依赖：

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

`params.thresholds.maxPayoffGap` 默认 3 集。既有报告布局、离线渲染、体检与资产统计保留；调性关键词扫描只发警告，不自动判失败。

## 苏联笑话参考

笑话参考内容只保存在工作区根目录的 [`soviet-jokes.md`](../../../soviet-jokes.md)。本 skill 和其他参考文件不复制笑话文本或逐条分析；创作时只借鉴抽象机制，并产出原创情节。

## 核心文件

- `SKILL.md`：Agent 工作流与强制暂停点
- `scripts/soviet-ghost-outline.mjs`：分卷、校验、体检、渲染、资产汇总
- `scripts/selftest.mjs`：确定性自测，不调用模型
- `references/schema.md`：大纲数据结构
- `references/payoff-taxonomy.md`、`payoff-ontology.md`：固定类型与因果结构
- `references/tone-bible.md`、`historical-research.md`、`canon-rules.md`、`anti-patterns.md`：项目约束
- `examples/ep01-outline.json`：EP01 验收样例

原始通用版仍位于 `skills/novel-outline/`，与本 skill 独立。
