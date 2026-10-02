# 故事骨架 · 项目 Canon、人物、场景与叙事回报

根据用户提供的故事素材、已确认的项目 Canon 与目标参数，先起草故事设计骨架。骨架至少包含 `storyDesign`、`canonRefs`、`characters`、`scenes`、`props`、`payoffs`、`historicalResearch`。此阶段不写分集。

## 执行顺序

1. **故事设计。**明确 `storyDesign.core`，并分别列出 `include`、`exclude`、`merge`、`risks`，条目格式为 `{what, why}`。不要求 exclude 非空。仅有明确文本来源时才填写可选 `sourceAdaptation`。
2. **Canon 依赖。**只记录当前故事依赖的项目级事实到 `canonRefs`（`id`、简述、来源）。Canon 是外部事实源；不可在 outline 中复制完整世界观、重新定义或擅自升级未确认信息。
3. **人物。**合并功能重复的角色，再按 `lead` / `support` / `functional` 分档。人物来源可来自素材、用户设定、Canon 或原创；不得为了保留旧改编字段而虚构原著对应角色。
4. **场景与资产。**主场景上限随集数计算。为新场景标明 `reuseClass`：`core`、`recurring` 或 `episode_only`；一次性 `episode_only` 场景要有 `reusePlan`。预判 AI 视频生成风险，只填写 `productionRisk`，不设计镜头。
5. **payoff 候选。**明确认知层级、setup、因果发展、payoff/result、theme、tone、落点与风险。允许 `type` 和 `ontologies` 多选；仍只能用既有十种 type 与 A/B ontology。
6. **历史研究问题。**对照 `historical-research.md`，只登记会影响故事决策的事实与可行性问题。纯架空制度设计不建立强制暂停。

## 必须暂停：payoff 候选确认

逐项展示 payoff 候选，等待用户明确确认：

```text
### 第 01 集 · P001
Type（可多选）：
Level：
Ontologies（可多选）：
Setup：
因果发展：
Payoff / result：
主题作用：
调性处理与风险：
状态：proposed
[STOP — PAYOFF CONFIRMATION REQUIRED]
```

只有获得明确批准后，才能设置 `status: "confirmed"`、`userConfirmed: true`。未经批准不得新增、删除、移动 payoff，也不得更改其 level、type 或核心含义。Comic beat 在分集写作阶段形成，不需要逐项确认，也不得包装成新的 payoff。

## 骨架评审内容

向用户展示：

- 故事核心及 include / exclude / merge 设计；
- 本故事依赖的 Canon refs；
- 角色分档、人物关系与角色来源；
- core / recurring / episode_only 场景及复用规划；
- payoff 候选与确认状态；
- 历史研究问题及其故事影响；
- 初步 AI 视频生产风险。

骨架通过 `validate --stage skeleton` 后展示给用户拍板。Payoff 逐项确认及必要历史决策完成后运行：

```bash
node scripts/soviet-ghost-outline.mjs validate <outline.json> --stage beats
```

未获用户确认的 payoff 不得进入正式分集。用户未确认骨架前不得开始分集写作。
