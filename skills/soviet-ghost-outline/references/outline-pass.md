# 骨架设计 · 改编、人物、场景与叙事回报

根据全部分卷摘要（或短篇原文）和用户参数，起草大纲骨架：`adaptation`、`characters`、`scenes`、`payoffs`、`historicalResearch`。在骨架和 payoff 通过确认门之前，不得写分集梗概。

## 执行顺序

1. **改编取舍。**明确主冲突；每条保留、删减和合并都写明理由。依赖原文的决定应附逐字依据。
2. **人物。**合并功能重复的角色，在 `characters[].from` 记录来源，再按现有 `lead` / `support` / `functional` 分档。功能性角色可以没有人物弧。
3. **场景。**遵守随集数变化的主场景上限。只出现一次的场景必须填写 `reusePlan`。
4. **叙事回报候选。**定义观众将获得什么认知或情绪回报，而非制造什么奇观或胜利。填写固定 taxonomy 类型、`level`、ontology、setup、payoff、theme、tone、落点集数与风险。确认它不是单纯的 hook 或 cliffhanger。
5. **道具。**沿用原有叙事道具规则；`beatIds` 填写对应 payoff 的 ID。待 payoff 与场景需求稳定后，再把道具引用写进分集。

所有判断必须依据用户提供的文本或用户确认的 canon。不得凭书名虚构原文内容，也不得凭模型记忆填补历史空白。

## 必须暂停：payoff 候选确认

逐项呈现候选，供用户审阅：

```text
### 第 01 集 · P001
类型：
层级：
Ontology：
Setup：
Payoff：
主题作用：
风险与克制处理建议：
状态：proposed
[STOP — PAYOFF CONFIRMATION REQUIRED]
```

未获用户批准前，不得新增或删除 payoff，不得移动落点、改变 level 或 type，也不得改变核心含义。等待用户明确逐项批准或修改。只有得到批准后，才能将 `status` 设为 `confirmed`、将 `userConfirmed` 设为 `true`。若用户要求修改，状态应回到 `revised` / 未确认，待再次批准。

只能使用 `payoff-taxonomy.md` 中的十种类型。若没有合适类型，解释现有 taxonomy 的不足并暂停，等待用户确认后再添加。

## 历史研究暂停点

若候选依赖尚未纳入用户确认 canon 的真实历史事实，停止并遵循 `historical-research.md`。将问题记为 `RESEARCH_REQUIRED`；用户解决之前，不得继续依赖该事实发展剧情。

## 骨架拍板

payoff 获得明确确认、必要的历史问题得到解决后，运行：

```bash
node scripts/soviet-ghost-outline.mjs validate <outline.json> --stage beats
```

随后把改编取舍、合并人物和 payoff 落点交给用户拍板。用户未确认骨架前不得开始写分集。
