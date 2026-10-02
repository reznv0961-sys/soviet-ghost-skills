---
name: soviet-ghost-outline
version: 1.1.0
description: |
  为《苏联亡灵局》原创系列开发故事大纲，支持 Canon 引用、叙事回报与 comic beat 分层、AI 视频生产风险标注。
  产出 outline.json、Markdown 与单页评审报告；用确定性质量门检查故事设计、单集价值、场景复用和历史依赖。
  支持现有大纲体检、历史研究暂停点与叙事回报逐项确认。零依赖、零 API key，使用当前会话额度。
  适用于《苏联亡灵局》原创系列大纲、故事拆解和大纲体检。
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

输入《苏联亡灵局》的项目设定、故事素材与目标参数，输出系列故事大纲五件套。**四件由模型撰写，一件由脚本计算**（资产清单从分集数据自动汇总）。

`{baseDir}` 表示本文件所在目录。脚本位于 `{baseDir}/scripts/soviet-ghost-outline.mjs`，无需安装依赖，可直接用 `node` 运行。

**项目定位：**这是《苏联亡灵局》的原创系列大纲开发工具，不再默认把项目当作小说改编。单集原则是**独立成立 + 弱连续长线**：每集有完整的小型观察、荒诞事件、世界规则揭示、人物变化或制度冲突；可推进人物、机构秘密或社区长线，但不强求 cliffhanger。

**工作边界：**不写正式剧本对白、不做分镜、不生成图像或语音提示词。分集梗概使用叙述体，出现引号对白即越界，`validate` 会拦截。人物设定、形象提示词和设定图属于 `novel-characters` 的工作范围；AI 视频风险只做标记，不写镜头调度或视频 prompt。

本技能与通用 `novel-outline` 分开维护，不修改或覆盖原技能。Canon 是外部项目级事实源；outline 只记录依赖，不创建、复制、覆盖或重新定义 Canon。未确认设定不能冒充 Canon。

### 第 0 步——收集参数 ⛔ 缺少必需信息时不开工

尽量一次询问完毕，不要轮流盘问：

| 参数 | 处理方式 |
| --- | --- |
| **总集数 × 单集时长** | 必问，没有合理默认值 |
| **题材与项目定位** | 必问；影响整体结构和叙事回报设计 |
| 故事来源 | 确认原创还是基于既有文本；原创不需要改编幅度 |
| 改编幅度 | 仅有改编来源时询问；忠实／抽核／借壳，不设默认 |
| 已有偏好 | 默认无；询问是否指定要保留的人物或场景 |
| 项目 Canon | 询问现有外部设定、已拍板事项及不可更改规则 |

平台阈值不同时，可通过 `params.thresholds` 覆盖。默认主角组最多 5 人、重要配角最多 10 人、功能性角色最多 10 人。**主场景上限随集数动态计算**：`4 + ⌈集数/10⌉`，限制在 5–15；只有明确设置 `maxPrimaryScenes` 才覆盖。`adaptMode` 仅在有改编来源时使用，原创项目可省略或设为 `null`。不要求 payoff 均匀分布。

人物表从用户提供的素材、设定或已确认 Canon 中提取，确定角色分档和人物线，供下游 `novel-characters` 使用。若用户已有 `cast.json`，按 `importance` 反向映射：`protagonist` / `major` → `lead`，`supporting` → `support`，`minor` → `functional`。大纲中的 `canonRefs` 仅引用当前故事依赖的既定规则，并非 Canon 本身。

### 第 1 步——定位输入

有原稿或指定素材时，材料优先级如下：

1. 用户点名的**精读章节**
2. **章节目录与简介**
3. 全文**分卷摘要**（第 2 步）

**禁止凭书名臆测原文内容。**所有判断都必须依据用户提供的文本、已确认 Canon 或明确标记为原创的设计。原创故事不需要伪造原著来源。

直接粘贴的文本素材先保存为 `.txt`。输出目录使用用户指定路径；未指定时，使用素材同级目录或当前项目工作目录。

### 第 2 步——分卷摘要（仅长文本需要）

分卷摘要是供未读长篇素材的后续步骤使用的中间材料，不是交付物。以下情况直接进入第 3 步：

- 短篇可以在单卷内处理。
- 当前会话已经通读原文，不必重复压缩，也不必事后补档。

长篇且尚未通读时，执行：

```bash
node {baseDir}/scripts/soviet-ghost-outline.mjs chunk <book.txt> <workdir>
```

脚本优先按章节标题分卷（默认每卷 15 章，可用 `--per-volume` 调整）；无法识别章节时按字数切分。脚本会打印 `{"volumes": N, ...}`。若 `truncated: true`，必须明确告知用户尾部内容尚未处理，不得隐瞒。

每卷读取 `{baseDir}/references/volume-pass.md` 和对应的 `<workdir>/vol-NN.txt`，将摘要写入 `<workdir>/summary-NN.json`。如果运行环境支持并行代理，可以在同一轮同时处理各卷；每个任务只负责一卷，完成后简短报告卷号。

### 第 3 步——起草骨架并等待用户拍板 ⛔

阅读 `{baseDir}/references/outline-pass.md` 与 `{baseDir}/references/schema.md`，按其中规范起草骨架，并写入 `<workdir>/outline.json`。骨架至少包括 `storyDesign`、`canonRefs`、`characters`、`scenes`、`props`、`payoffs` 和 `historicalResearch`；此时不要编写分集。

```bash
node {baseDir}/scripts/soviet-ghost-outline.mjs validate <workdir>/outline.json --stage skeleton
```

向用户展示故事核心、include/exclude/merge 设计、Canon 引用、人物、核心及复用场景、初步生产风险和 payoff 候选。Payoff 候选必须逐项列明多标签 type、level、ontologies、setup、payoff、theme、调性处理与风险，然后**暂停等待用户明确确认**。仅在用户明确批准后，才可设置 `status: "confirmed"` 和 `userConfirmed: true`。Comic beat 在分集阶段形成，不需要逐项确认。

按 `{baseDir}/references/historical-research.md` 区分真实历史事实、历史可行性推演和架空制度设计。只有当前剧情决策实质依赖尚未解决的真实历史事实时才建立强制暂停；苏联、KGB、军衔、制服或国籍等词语单独出现不触发 STOP。项目 Canon 或已确认历史基础中已有的事实可复用。

⛔ 未经用户确认不得新增、删除、移动 payoff，不得更改其 `level`、`type` 或核心含义。一个 payoff 可以拥有多个既有 `type` 与 `ontologies`；不要强迫单选。`type` 仍只能使用固定 taxonomy 中定义的十种值。若现有 taxonomy 不适用，先解释原因并暂停，等待用户确认后再作变更。

payoff 逐项确认、必需的历史问题解决后，运行：

```bash
node {baseDir}/scripts/soviet-ghost-outline.mjs validate <workdir>/outline.json --stage beats
```

`beats` 阶段通过后，再将故事设计取舍、人物合并、场景安排和 payoff 落点交给用户拍板。用户未确认骨架前不得进入分集写作。

### 第 4 步——根据反馈细化骨架

吸收用户意见修改骨架；任何 payoff 核心决策的变更都要重新取得用户确认。再次运行 `validate --stage beats`。若用户没有修改意见且骨架已确认，可直接进入下一步。

### 第 5 步——分批撰写分集梗概

每批最多 **10 集**。只有骨架已经拍板、payoff 已确认后才能开始。每批使用已确认骨架、负责的集数区间和该区间内的已确认 payoff；阅读 `{baseDir}/references/episode-pass.md` 后撰写，并将结果保存到 `<workdir>/eps-NN.json`。每集都要能独立成立，并明确 `episodePurpose`、`audienceGain`、`endingPull`、`productionRisk`；无明显生成风险时也填写 `{ "level": "low", "reasons": [] }`。每集至少引用一个已确认 payoff，或包含一个有效 `comicBeat`；两者兼有为推荐状态。Comic beat 只描述内容与功能，不扩写成正式台词。

合并各批次内容时，按 `ep` 排序写入 `outline.json` 的 `episodes`。分集不得擅自增加 payoff；`endingPull` 可以是 `none`，不要求 cliffhanger。旧 `hook` / `suspense` 仅为 legacy 兼容字段。

### 第 6 步——校验 ⛔ 不得跳过

```bash
node {baseDir}/scripts/soviet-ghost-outline.mjs validate <输出目录>/<书名>-outline.json --stage full
```

质量门由脚本确定性检查，至少涵盖：角色分档上限、随集数变化的主场景上限、道具上限、一次性 `episode_only` 场景复用方案、payoff 类型与用户确认状态、每集至少一个已确认 payoff 或有效 comic beat、`endingPull` 格式、新版 episode 的生产风险格式、历史研究类别与未解决事实依赖、三人同框拆解、引用完整性、无未使用人物或场景、叙述体不含对白。不要求回报均匀分布、首集 cliffhanger 或强制 major。

逐项修复所有校验问题并重跑，直到通过。调性扫描产生的 `WARNING: POSSIBLE TONE DRIFT` 是提示，不会自动使校验失败；由用户决定是否保留相关内容。

### 第 7 步——渲染报告并汇报

在输出目录执行：

```bash
node {baseDir}/scripts/soviet-ghost-outline.mjs render <书名>-outline.json --md > <书名>-outline.md
node {baseDir}/scripts/soviet-ghost-outline.mjs render <书名>-outline.json --html > outline-report.html
```

报告界面默认中文。用户需要英文界面时可使用 `--lang en`，或在 `outline.json` 顶层设置 `lang`；命令行参数优先。只切换界面文案，不翻译或改写大纲数据。

报告包含 KPI 统计、关键决策、Canon refs、payoff 时间轴、分集的目的/观众收获/comic beat/结尾牵引/生产风险、场景复用等级、每集调度矩阵、资产量折算和完整质量门；未通过的项目会显示诊断信息。HTML 报告支持离线查看，并可导出原始 JSON。

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

用户修改上游决定后，重新运行 `validate`。让校验器指出人物合并后仍被引用的 ID、场景删减后空转的分集、payoff 落点与集内引用不一致、单集缺少叙事价值等，不要依赖记忆手工追踪。

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

`{baseDir}/examples/ep01-outline.json` 是待人工重写的旧样例，不是当前项目剧情 Canon，也不是 v1.1 黄金样例。本轮不得自动重写；只做兼容性检查，并在报告中列明需要人工更新的旧字段。
