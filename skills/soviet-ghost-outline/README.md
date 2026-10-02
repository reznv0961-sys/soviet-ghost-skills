# soviet-ghost-outline

《苏联亡灵局》原创系列故事大纲技能。与通用 `novel-outline` 分开维护，不默认把项目当作小说改编。单集原则为**独立成立 + 弱连续长线**：不强求 cliffhanger，每集至少有一个已确认 payoff 或有效 comicBeat。

故事 outline 只通过 `canonRefs` 引用外部项目级 Canon，不创建或改写世界观事实。结构性 payoff 继续逐项等待用户确认；comicBeat 是轻量的荒诞 / 滑稽执行单元，不要求逐项确认，也不需要完整因果结构。历史研究只有在未解决事实会实质改变剧情时才强制暂停；纯架空制度设计不触发 STOP。AI 视频生产风险只做标记，不进入分镜或视频 prompt。

## 工作流程

1. 收集集数、时长、题材、故事来源、偏好与项目 Canon；原创故事无需 `adaptMode`。
2. 长文本素材按章节分卷；短文本直接进入故事骨架。
3. 起草 `storyDesign`、`canonRefs`、人物、场景、道具、payoff 候选与历史研究问题，不写分集。
4. 展示 payoff 的多标签 type、level、`ontologies`、setup、payoff、theme 和风险，暂停等待用户逐项确认。
5. 确认必要的历史事实依赖及骨架后运行 `validate --stage beats`。
6. 每批最多 10 集写作；每集包含 `episodePurpose`、`audienceGain`、`endingPull`、`productionRisk`，并有 payoff 或 comicBeat。没有明显生产风险时仍填写 `{ "level": "low", "reasons": [] }`。
7. 运行 full 校验并修复问题，再渲染报告与资产清单。

详细流程见 [SKILL.md](SKILL.md)，JSON 字段见 [schema.md](references/schema.md)。

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

`beats` 阶段允许整个 `payoffs` 表为空；正式分集必须逐集满足 payoff 或 comicBeat 质量门。调性关键词扫描只发 warning，不自动判失败。

## 苏联笑话参考

笑话参考内容只保存在工作区根目录的 [`soviet-jokes.md`](../../../soviet-jokes.md)。本 skill 和其他参考文件不复制笑话文本或逐条分析；创作时只借鉴抽象机制，并产出原创情节。

## 核心文件

- `SKILL.md`：工作流程与必须确认的暂停点
- `scripts/soviet-ghost-outline.mjs`：分卷、校验、体检、渲染、资产汇总
- `scripts/selftest.mjs`：确定性自测，不调用模型
- `references/schema.md`：v1.1 大纲结构与旧字段兼容说明
- `references/payoff-taxonomy.md`、`payoff-ontology.md`：固定类型与因果结构
- `references/episode-pass.md`、`outline-pass.md`、`historical-research.md`：写作与研究流程
- `references/tone-bible.md`、`canon-rules.md`、`anti-patterns.md`：项目约束
- `examples/ep01-outline.json`：待人工重写的旧样例，不是 Canon 或 v1.1 黄金样例

原始通用版仍位于 `skills/novel-outline/`，与本 skill 独立。
