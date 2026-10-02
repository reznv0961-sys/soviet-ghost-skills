# outline.json 结构（v1.1）

outline 是一个故事的设计与执行计划，不是项目世界观 Canon。Markdown、HTML 报告和资产统计由脚本生成。

```json
{
  "source": "故事名",
  "lang": "zh",
  "params": {
    "episodes": 1,
    "minutesPerEpisode": 8,
    "genre": "制度荒诞与历史错位",
    "adaptMode": null,
    "preferences": []
  },
  "canonRefs": [
    { "id": "CANON-...", "summary": "本故事使用到的既定规则简述", "source": "项目世界观文档" }
  ],
  "storyDesign": {
    "core": "故事核心",
    "include": [{ "what": "纳入设计", "why": "理由" }],
    "exclude": [{ "what": "不纳入设计", "why": "理由" }],
    "merge": [{ "what": "合并设计", "why": "理由" }],
    "risks": [{ "what": "风险", "why": "处理方式" }]
  },
  "sourceAdaptation": {
    "mode": "抽核",
    "source": "指定原稿",
    "notes": [{ "what": "改编说明", "why": "理由", "evidence": "可选逐字依据" }]
  },
  "characters": [
    { "id": "C01", "name": "…", "role": "…", "tier": "lead", "arc": "…", "from": ["原创"] }
  ],
  "scenes": [
    {
      "id": "S01", "name": "…", "primary": true, "reuseClass": "core", "reusePlan": "…",
      "productionRisk": { "level": "low", "reasons": [] }
    }
  ],
  "props": [],
  "payoffs": [
    {
      "id": "P001", "episode": 1, "level": "minor",
      "type": ["historical_dislocation", "identity_conflict"],
      "ontologies": ["historical_dislocation_A", "identity_conflict_B"],
      "setup": "…", "payoff": "…", "theme": "…", "tone": "克制",
      "status": "proposed", "userConfirmed": false
    }
  ],
  "historicalResearch": [
    {
      "id": "HR-001", "category": "historical_fact", "claim": "…",
      "status": "RESEARCH_REQUIRED", "storyImpact": "…"
    }
  ],
  "episodes": [
    {
      "ep": 1, "synopsis": "叙述体梗概",
      "episodePurpose": ["world_reveal", "institutional_satire"],
      "audienceGain": { "newUnderstanding": "…", "storyProgress": "…" },
      "endingPull": { "kind": "ironic_image", "content": "…" },
      "sceneIds": ["S01"], "characterIds": ["C01"], "payoffs": [], "comicBeats": [
        { "kind": "bureaucratic_detail", "beat": "女主以严肃行政态度提醒对方谨慎回答", "function": "体现时代错位下的制度冷幽默" }
      ],
      "productionRisk": { "level": "low", "reasons": [] },
      "propIds": [], "warnings": []
    }
  ]
}
```

## 项目 Canon 与故事设计

- `canonRefs` 可为空；非空时每项包含 `id`、`summary`、`source`。summary 只是依赖索引，不能代替 Canon 原文。
- Canon 是外部项目级事实源。outline 只能引用，不得自行创建、升级、覆盖或重新定义 Canon；未确认设定不得伪装成 Canon。
- `storyDesign` 必填，`core` 必须有内容；`include`、`exclude`、`merge`、`risks` 均为数组，条目使用 `{what, why}`。不要求 `exclude` 非空。
- 仅当故事确实基于原稿时填写可选 `sourceAdaptation`；原创故事不需要该字段。

## params

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `episodes` | 是 | 总集数，正整数；分集数量与它一致 |
| `minutesPerEpisode` | 是 | 单集时长，正数 |
| `genre` | 是 | 题材与项目定位 |
| `adaptMode` | 否 | 仅用于兼容改编来源；`忠实` / `抽核` / `借壳`。原创省略或设为 `null` |
| `preferences` | 否 | 用户点名保留的人物、场景等 |
| `thresholds` | 否 | 覆盖角色数、道具数和主场景数阈值。`maxPayoffGap` 是 deprecated legacy 参数，不再作为核心门 |

## 人物、场景与资产

- 人物 ID 使用全局唯一 `C01` 格式。角色分档为 `lead`、`support`、`functional`；只有非功能性角色必须有 `arc`。
- 新大纲场景使用 `reuseClass`：`core`（系列核心资产）、`recurring`（预计重复出现）、`episode_only`（少量剧情使用）。旧数据可省略该字段。
- `episode_only` 场景若只出现一次，必须填写 `reusePlan`；评审报告会标示其单独制作资产的必要性。`productionRisk` 可在场景或分集记录。
- 新版正式 episode 必须填写 `productionRisk`；没有明显风险时写 `{ "level": "low", "reasons": [] }`。旧 episode 可缺省以兼容存量数据。scene 上的 `productionRisk` 可选。结构为 `{level, reasons}`；level 只能是 `low` / `medium` / `high`，reasons 为字符串数组。推荐标签包括 `vehicle_motion`、`multi_actor_interaction`、`crowd`、`complex_spatial_blocking`、`prop_handoff`、`hand_closeup`、`physical_contact`、`transparent_character`、`moving_door`、`background_parallax`、`text_or_map`、`costume_continuity`、`character_consistency`、`weather`、`complex_camera_motion`。此处只提示下游生产风险，不写分镜或视频 prompt。
- `props` 可选。`id` 使用 `P01`，`name` 与 `function` 必填，`beatIds` 可引用 payoff ID。

## payoff

- 整份大纲的 `payoffs` 可以是空数组。此时 `beats` 阶段不因 payoff 表为空失败；正式分集仍须逐集通过 payoff / comicBeat 价值门。
- payoff 是结构性叙事回报，需有 setup、因果发展和 payoff/result，并继续逐项等待用户确认。
- `type` 为非空数组，可同时使用现有十种 taxonomy 标签；不得新增第十一类。
- 新格式使用 `ontologies` 数组。一个 payoff 可以有多个 ontology；每个 ontology 必须属于其中一个 `type`。旧 `ontology` 单字符串可兼容读取，标记 deprecated。
- `level` 仍表示认知层级：`minor` 改变对场景、关系或局部规则的理解；`major` 改变对人物、制度、世界结构、历史关系或主题的整体理解。笑得很响不自动是 major，安静揭示也可能是 major。
- 正式 episode 只能引用同时满足 `status: "confirmed"` 与 `userConfirmed: true` 的 payoff。
- payoff 时间间隔、至少一个 major、major 必须早于最后一集均为 deprecated 旧规则，不再构成核心失败条件。

## episodes

- `ep` 从 1 连续编号；`synopsis`、`sceneIds`、`characterIds` 和 `payoffs` 必填。
- `episodePurpose` 是字符串数组，不锁死枚举。建议：`world_reveal`、`character_development`、`institutional_satire`、`historical_observation`、`relationship_progress`、`long_arc_progress`、`atmosphere`、`setup`、`transition`。
- 正式分集的 `audienceGain` 要填写 `newUnderstanding` 与 `storyProgress`，分别说明观众新理解和本集推进。
- `endingPull` 格式为 `{kind, content}`；kind 为 `question`、`world_clue`、`character_reaction`、`ironic_image`、`long_arc`、`none`。`none` 合法且 content 可为空或省略；不要求 cliffhanger。
- `comicBeats` 是可选数组，每项 `{kind, beat, function}`。kind 是轻量标签，不限制 taxonomy；建议 `dry_capper`、`bureaucratic_detail`、`visual_absurdity`、`historical_mismatch`、`reaction`、`procedural_gag`、`other`。不设 ontology，也不要求逐项用户确认。只写叙事功能和内容，不写正式对白。
- 每集至少引用一个已确认 payoff，或至少有一个有效 comicBeat；两者兼有为推荐状态。无 payoff 时不再要求 `payoffRationale`。
- `hook`、`suspense` 是 deprecated legacy 字段；新大纲统一使用 `endingPull`。
- 单集原则是独立成立 + 弱连续长线，不必人为制造重大悬念。

## 历史研究

`historicalResearch` 条目使用 `id`、`category`、`claim`、`status`、`storyImpact`。category 为 `historical_fact`、`historical_plausibility`、`fictional_institution_design`。历史事实或可行性推演可以在有实质剧情依赖时要求研究；纯架空制度设计不触发历史 STOP。只有尚未解决且会实质改变剧情的历史依赖阻止 full。详细规则见 `historical-research.md`。

## 校验

```bash
node scripts/soviet-ghost-outline.mjs validate outline.json --stage skeleton|beats|full
node scripts/soviet-ghost-outline.mjs checkup outline.json
```

`skeleton` 校验故事设计、人物、场景与 Canon 引用；`beats` 额外要求 payoff taxonomy 与人工确认有效；`full` 检查每集价值、生产风险、历史依赖和全部 ID 引用。旧字段 `adaptation`、`ontology`、`hook`、`suspense` 可读取兼容，但新大纲不得继续使用其旧结构。
