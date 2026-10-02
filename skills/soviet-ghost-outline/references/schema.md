# outline.json 结构

五件套的载体。**模型只管填这份 JSON**，Markdown 和 report.html 由 `render` 渲染出来，资产清单由脚本汇总——五件套是四件模型写 + 一件算出来的。

```json
{
  "source": "书名",
  "lang": "zh",
  "params": { "episodes": 60, "minutesPerEpisode": 2, "genre": "历史错位黑色幽默", "adaptMode": "抽核", "preferences": [] },
  "adaptation": { "core": "…", "keep": [], "cut": [], "merge": [], "risks": [] },
  "characters": [ { "id": "C01", "name": "…", "role": "…", "arc": "…", "from": ["原著…", "合并：…"] } ],
  "scenes": [ { "id": "S01", "name": "…", "primary": true, "reusePlan": "…" } ],
  "payoffs": [ { "id": "P001", "episode": 1, "level": "major", "type": ["historical_dislocation"], "ontology": "historical_dislocation_A", "setup": "…", "payoff": "…", "theme": "…", "tone": "冷峻克制", "status": "proposed", "userConfirmed": false } ],
  "historicalResearch": [ { "id": "HR-001", "claim": "…", "status": "RESEARCH_REQUIRED", "storyImpact": "…" } ],
  "episodes": [ { "ep": 1, "synopsis": "…", "hook": "…", "suspense": "…", "sceneIds": ["S01"], "characterIds": ["C01"], "payoffs": ["P001"], "propIds": ["P01"], "crowdPlan": "…", "warnings": [] } ]
}
```

## params

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `episodes` | 是 | 总集数，正整数。分集数量必须与它一致 |
| `minutesPerEpisode` | 是 | 单集时长（分钟） |
| `genre` | 是 | 题材与项目定位，必须提供；不能据此擅自增加 payoff taxonomy 类型 |
| `adaptMode` | 是 | `忠实` / `抽核` / `借壳`，只能这三个 |
| `preferences` | 否 | 用户点名要保的角色、戏 |
| `thresholds` | 否 | 逐项覆盖质量门阈值：`maxLeads`(5) / `maxSupport`(10) / `maxFunctional`(10) / `maxPayoffGap`(3) / `maxPrimaryScenes`（缺省随集数动态：4 + ⌈集数/10⌉，夹在 5–15）。短篇建议收紧角色档 |

## adaptation 改编说明

`core` 一句话核心，必填。`keep` / `cut` / `merge` 每条 `{what, why}`，`keep` 至少一条；**`adaptMode` 不是忠实时 `cut` 不能为空**。`keep` 可带 `evidence`——**原文逐字片段**，禁止凭书名脑补的对策就在这：关键取舍要能指回原文。`risks` 每条 `{what, plan}`。

两个可选的**决策补注**，报告的「关键决策」区块会展示：

- `cutNote` — 砍线的结论句（「这意味着：全剧终点是……原著后 30 章基本不用」这种），说清砍完之后故事的终点变成了什么
- `mergeNote` — 合人的补注，通常写主角组入选理由（谁有完整转变弧）

给了就不能是空字符串，`validate` 会拦。major payoff 落点列表**不用另写**——报告从 `payoffs` 里自动算。

## characters 人物表

每个字段都以校验器为准，一个不多一个不少：

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `id` | 是 | `C01` 格式，全局唯一，分集靠它引用 |
| `name` | 是 | 姓名；功能性角色用称呼标签（「急诊医生」） |
| `role` | 是 | 定位一句话：女主 / 搅局配角 / 渡口的报时人…… |
| `tier` | 是 | 三档之一，见下表 |
| `arc` | lead / support 必填 | 人物弧；functional 可省——医生就是来缝针的 |
| `from` | 是 | **← 改动记录**，非空数组：原著对应谁、合并了谁、纯原创写 `["原创"]` |

- `tier` 只能三档——一刀切的角色上限混淆了「观众要记住谁」和「制作要维护多少张脸」，分档把它拆开：

  | tier | 是谁 | 上限 | 规则 |
  | --- | --- | --- | --- |
  | `lead` | 主角组（男女主 + 主反派） | 1–5 人 | `arc` 必填 |
  | `support` | 有名字的重要配角（亲属、闺蜜、副反派） | ≤ 10 | `arc` 必填 |
  | `functional` | 功能性角色（医生、秘书、店员） | ≤ 10 | **占脸不占名**：`name` 用称呼标签（「急诊医生」）；`arc` 可省——医生就是来缝针的 |

  无名背景人不进表、不追踪、不限量。

  **这份表是下游 novel-characters 的角色清单**：谁进谁不进、谁是主角组在这里定死，角色设定照着做，不用再判断一遍轻重（`tier` 对应过去就是 `importance`：`lead` → protagonist、`support` → supporting、`functional` → minor）。反过来，手上已经有 cast.json 的话也能映射进来：protagonist/major → `lead`，supporting → `support`，minor → `functional`

## scenes

- `id` 格式 `S01`，全局唯一；`name` 必填；`primary` 必填布尔（主场景上限随集数动态，见 `thresholds`）
- **在全剧只出现一次的场景必须带 `reusePlan`**（规避方案：复用哪个现有环境资产、换什么时段天气改出来）

## props 叙事道具

- **可选字段**。没写照常通过全部质量门（`prop-cap` 与 `refs` 两道会明说跳过）；写了就按下面查
- `id` 格式 `P01`，全局唯一；`name` 必填
- **`function` 必填——这一层唯一要拍板的东西：这件物件在戏里承载什么。** 填不出来说明它不是叙事道具，是场景陈设，那归 `novel-art` 的场景锚点管，不进这张表
- `beatIds` 可选：保留原字段名以兼容资产汇总，值引用 `payoffs[].id`。写了就必须指向真实 payoff id
- 上限 `maxProps` 默认 8 件（进 `params.thresholds` 可覆盖）。跟主角数量一个量级——**只收有特写、跨集出现、承载剧情的**
- 分集用 `episodes[].propIds` 引用。**没有任何一集引用的道具会被 `refs` 门点名**——跟失业角色、空转场景同一个判据

边界：尺度、锚点、状态变体、白底提示词都是 `novel-art` 的活，不在这里定。这张表回答「哪几件物件承载剧情、各自承载什么」，美术层回答「它长什么样、怎么保证六集都长一样」。

## payoffs 叙事回报表

- `id` 格式 `P001`；`level` 只能 `major` / `minor`；`type` 必须是数组，且只能使用 `references/payoff-taxonomy.md` 中的十类之一
- `ontology` 必须指向一种已定义因果结构；`setup` 与 `payoff` 都必填，不能把 hook 当作 payoff；`theme` 与 `tone` 记录主题作用和执行语调
- `status` 为 `proposed` / `confirmed` / `rejected` / `revised`；正式进入 `--stage beats` 前，每项都必须为 `status: "confirmed"` 且 `userConfirmed: true`
- `episode` 是叙事回报的落点；分集用 `episodes[].payoffs` 引用对应 `Pxxx`。没有 payoff 的分集必须写 `payoffRationale`
- 每集都必须有 `payoffs` 数组；为空时 `payoffRationale` 必填，不能用 hook 或悬念冒充 payoff
- 硬规则：叙事回报间隔 ≤ `maxPayoffGap`（默认 3），开头结尾无真空；**至少一个 major，且最早的 major 不能落在最后一集**
- 若需要新增顶级 `type`，先提出理由并暂停等待用户确认；未知类型不能进入 beats/full

## historicalResearch

每项状态为 `CANON` / `PROVISIONAL` / `RESEARCH_REQUIRED`。真实历史事实尚未核实时不得写成 canon；存在未解决的 `RESEARCH_REQUIRED` 时 `--stage full` 必须失败。详细暂停流程见 `references/historical-research.md`。

## episodes 分集梗概

- `ep` 从 1 连续编号，总数等于 `params.episodes`
- `synopsis` / `hook` / `suspense` 三栏**都必填**——【钩子】【悬念】空了视为未完成
- **叙述体**：三栏里出现 `「」『』“”` 引号对白就是在写剧本，越界，validate 会拦
- `sceneIds` / `characterIds` 必填且必须指向已登记的 id；每个角色至少出现一集、每个场景至少用一次
- `propIds` 可选（写了 `props` 才有意义），必须指向已登记的道具 id；每件道具至少出现一集
- `characterIds` ≥ 3 的集必须写 `crowdPlan`（同框拆解方案）。**校验按人数判，是代理指标**——如果这一集三人实际不同框（分处不同场次），把这个事实写成方案即可：「三人分处两场，无同框，分场拍」，照样通过
- `warnings`：梗概里扫到生成难点关键词（雨戏/肢体接触/人群/手部特写）就必须列进来，宁可多报

## 校验

```bash
node scripts/soviet-ghost-outline.mjs validate outline.json --stage skeleton|beats|full
node scripts/soviet-ghost-outline.mjs checkup outline.json   # 质量门 ✓/✗
```

stage 就是流程门：骨架草案过 `skeleton`，payoff 用户确认并完成研究 checkpoint 后、**写分集之前必须过 `beats`**；交付前过 `full`。
