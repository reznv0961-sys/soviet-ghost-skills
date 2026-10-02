---
name: soviet-ghost-outline
version: 1.0.0
description: |
  将《苏联亡灵局》项目设定转化为短剧大纲五件套：改编说明、人物表、叙事回报表、分集梗概和资产清单。
  产出 outline.json、Markdown 与单页评审报告；用确定性质量门检查角色分档、场景上限、叙事回报间隔、分集钩子与悬念等要求。
  支持现有大纲体检、历史研究暂停点与叙事回报逐项确认。零依赖、零 API key，使用当前会话额度。
  适用于《苏联亡灵局》大纲设计、改编、分集拆解和大纲体检。
allowed-tools:
  - Read
  - Write
  - Bash
  - Task
  - Glob
triggers:
  - soviet-ghost-outline
  - 苏联亡灵局
  - 亡灵局大纲
  - 改编大纲
  - 短剧大纲
  - 大纲体检
metadata:
  license: Apache-2.0
  requires:
    bins:
      - node          # 18 及以上，只用标准库，无 npm 依赖
  runtimes:
    - claude-code
    - codex
---

## soviet-ghost-outline

输入《苏联亡灵局》的项目设定、原稿与目标参数，输出短剧改编大纲五件套。**四件由模型撰写，一件由脚本计算**（资产清单从分集数据自动汇总）。

`{baseDir}` 表示本文件所在目录。脚本位于 `{baseDir}/scripts/soviet-ghost-outline.mjs`，无需安装依赖，可直接用 `node` 运行。

**工作边界：**不写剧本台词、不做分镜、不生成图像或语音提示词。分集梗概使用叙述体，出现引号对白即越界，`validate` 会拦截。人物设定、形象提示词和设定图属于 `novel-characters` 的工作范围。

本技能与通用 `novel-outline` 分开维护，不修改或覆盖原技能。保留原有的参数收集、原稿定位、长文本分卷、骨架拍板、分批写集、确定性校验、报告渲染、资产统计和体检工作流；将通用爽点体系改为《苏联亡灵局》专用的叙事回报（payoff）确认流程。

### 第 0 步——收集参数 ⛔ 缺少必需信息时不开工

尽量一次询问完毕，不要轮流盘问：

| 参数 | 处理方式 |
| --- | --- |
| **总集数 × 单集时长** | 必问，没有合理默认值 |
| **题材与项目定位** | 必问；影响整体结构和叙事回报设计 |
| 改编幅度 | 默认**抽核**（忠实／抽核／借壳），告知用户 |
| 已有偏好 | 默认无；询问是否指定要保留的人物或场景 |
| 用户确认的 canon | 询问现有设定、已经拍板的内容及不可更改事项 |

平台阈值不同时，可通过 `params.thresholds` 覆盖。默认主角组最多 5 人、重要配角最多 10 人、功能性角色最多 10 人、叙事回报间隔最多 3 集。**主场景上限随集数动态计算**：`4 + ⌈集数/10⌉`，限制在 5–15；只有明确设置 `maxPrimaryScenes` 才覆盖。短篇建议收紧角色档阈值。

人物表从原文或用户设定中提取，确定角色分档、人物线和来源，供下游 `novel-characters` 使用。若用户已有 `cast.json`，可直接将其作为人物原料；按 `importance` 反向映射：`protagonist` / `major` → `lead`，`supporting` → `support`，`minor` → `functional`。

### 第 1 步——定位输入

材料优先级如下：

1. 用户点名的**精读章节**
2. **章节目录与简介**
3. 全文**分卷摘要**（第 2 步）

**禁止凭书名臆测原文内容。**所有判断都必须依据用户提供的文本或已确认 canon。`adaptation.keep` 中的关键取舍应附上原文逐字片段作为 `evidence`。

直接粘贴的正文先保存为 `.txt`。输出目录使用用户指定路径；未指定时，使用原稿同级目录。

### 第 2 步——分卷摘要（仅长文本需要）

分卷摘要是供未读原文的后续步骤使用的中间材料，不是交付物。以下情况直接进入第 3 步：

- 短篇可以在单卷内处理。
- 当前会话已经通读原文，不必重复压缩，也不必事后补档。

长篇且尚未通读时，执行：

```bash
node {baseDir}/scripts/soviet-ghost-outline.mjs chunk <book.txt> <workdir>
```

脚本优先按章节标题分卷（默认每卷 15 章，可用 `--per-volume` 调整）；无法识别章节时按字数切分。脚本会打印 `{"volumes": N, ...}`。若 `truncated: true`，必须明确告知用户尾部内容尚未处理，不得隐瞒。

每卷读取 `{baseDir}/references/volume-pass.md` 和对应的 `<workdir>/vol-NN.txt`，将摘要写入 `<workdir>/summary-NN.json`。如果运行环境支持并行代理，可以在同一轮同时处理各卷；每个任务只负责一卷，完成后简短报告卷号。

### 第 3 步——起草骨架并等待用户拍板 ⛔

阅读 `{baseDir}/references/outline-pass.md` 与 `{baseDir}/references/schema.md`，按其中规范起草骨架，并写入 `<workdir>/outline.json`。骨架包括 `adaptation`、`characters`、`scenes`、`payoffs` 和 `historicalResearch`；此时不要编写分集。

```bash
node {baseDir}/scripts/soviet-ghost-outline.mjs validate <workdir>/outline.json --stage skeleton
```

向用户展示改编取舍、人物合并、场景规划和 payoff 候选。Payoff 候选必须逐项列明类型、level、ontology、setup、payoff、theme、调性处理与风险，然后**暂停等待用户明确确认**。仅在用户明确批准后，才可设置 `status: "confirmed"` 和 `userConfirmed: true`。

若候选依赖真实历史事实，按 `{baseDir}/references/historical-research.md` 建立暂停点。说明待确认的事实及其对故事的影响，等待用户选择研究、提供设定、明确架空或删除相关内容。不得用模型记忆补齐，也不得将未核实事实写成 canon。

⛔ 未经用户确认不得新增、删除、移动 payoff，不得更改其 `level`、`type` 或核心含义。`type` 只能使用固定 taxonomy 中定义的十种值。若现有 taxonomy 不适用，先解释原因并暂停，等待用户确认后再作变更。

payoff 逐项确认、必需的历史问题解决后，运行：

```bash
node {baseDir}/scripts/soviet-ghost-outline.mjs validate <workdir>/outline.json --stage beats
```

`beats` 阶段通过后，再将改编取舍、人物合并、场景安排和 payoff 落点交给用户拍板。用户未确认骨架前不得进入分集写作。

### 第 4 步——根据反馈细化骨架

吸收用户意见修改骨架；任何 payoff 核心决策的变更都要重新取得用户确认。再次运行 `validate --stage beats`。若用户没有修改意见且骨架已确认，可直接进入下一步。

### 第 5 步——分批撰写分集梗概

每批最多 **10 集**。只有骨架已经拍板、payoff 已确认后才能开始。每批使用已确认骨架、负责的集数区间和该区间内的已确认 payoff；阅读 `{baseDir}/references/episode-pass.md` 后撰写，并将结果保存到 `<workdir>/eps-NN.json`。环境支持并行时，可同时撰写彼此独立的集数区间。

合并各批次内容时，按 `ep` 排序写入 `outline.json` 的 `episodes`。分集不得擅自增加 payoff；hook、suspense 与 payoff 必须分开记录。

### 第 6 步——校验 ⛔ 不得跳过

```bash
node {baseDir}/scripts/soviet-ghost-outline.mjs validate <输出目录>/<书名>-outline.json --stage full
```

质量门由脚本确定性检查，至少涵盖：角色分档上限、随集数变化的主场景上限、叙事道具上限、一次性场景复用方案、payoff 类型与用户确认状态、payoff 间隔及开头结尾空档、至少一个 major payoff 且不只落在最后一集、每集钩子与悬念、三人同框拆解、生成难点预警、引用完整性、无未使用人物或场景、叙述体不含对白，以及历史研究事项已解决。

逐项修复所有校验问题并重跑，直到通过。调性扫描产生的 `WARNING: POSSIBLE TONE DRIFT` 是提示，不会自动使校验失败；由用户决定是否保留相关内容。

### 第 7 步——渲染报告并汇报

在输出目录执行：

```bash
node {baseDir}/scripts/soviet-ghost-outline.mjs render <书名>-outline.json --md > <书名>-outline.md
node {baseDir}/scripts/soviet-ghost-outline.mjs render <书名>-outline.json --html > outline-report.html
```

报告界面默认中文。用户需要英文界面时可使用 `--lang en`，或在 `outline.json` 顶层设置 `lang`；命令行参数优先。只切换界面文案，不翻译或改写大纲数据。

报告包含 KPI 统计、关键决策、payoff 时间轴、分集概览、场景概览、每集调度矩阵、资产量折算和完整质量门；未通过的项目会显示诊断信息。HTML 报告支持离线查看，并可导出原始 JSON。

汇报时说明总集数、角色与场景数量、payoff 分布、报告路径，以及是否存在截断或未通过项。最终输出：

```text
<输出目录>/
├── <书名>-outline.json
├── <书名>-outline.md
└── outline-report.html
```

### 体检模式

用户提供现有大纲并只要求诊断时，将其整理为 `outline.json`。必要字段缺失时询问用户或明确标注缺失，再执行：

```bash
node {baseDir}/scripts/soviet-ghost-outline.mjs checkup <outline.json>
node {baseDir}/scripts/soviet-ghost-outline.mjs render <outline.json> --html > outline-report.html
```

体检报告用于展示所有质量门的通过与未通过项；未通过不会阻止报告渲染。

### 上游变更与联动校验

用户修改上游决定后，重新运行 `validate`。让校验器指出人物合并后仍被引用的 ID、场景删减后空转的分集、payoff 移动后产生的间隔问题等，不要依赖记忆手工追踪。

### 边界与兼容性

- 单次最多处理 60 卷；超过上限时明确报告 `truncated`，不得静默截断。
- 阈值是参数而不是固定命令；平台需求不同可用 `params.thresholds` 覆盖，无须改脚本。
- 报告支持中文与英文界面；默认为中文。切换界面语言不翻译大纲数据。
- 五件套中的资产清单始终由脚本计算，避免手工填写漏项。
- 所有苏联笑话参考内容只允许存放在工作区根目录的 [`soviet-jokes.md`](../../../soviet-jokes.md)。此技能及其他参考文件不得复制笑话文本或逐条分析；创作时仅提炼抽象机制并生成原创情节。
- 项目专属规则仅维护在本目录；通用 `skills/novel-outline` 保持独立，不得为本技能修改。

### 自测

```bash
node {baseDir}/scripts/selftest.mjs
```

自测不调用模型；修改脚本后应运行并确认全部断言通过。

### 自带样例

`{baseDir}/examples/ep01-outline.json` 是 EP01 验收样例。它用于回归校验，不应被当成额外的用户 canon。
