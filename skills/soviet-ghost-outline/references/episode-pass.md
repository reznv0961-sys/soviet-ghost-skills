# 分集梗概 · 独立成立，弱连续推进

只有用户确认骨架且所有拟引用 payoff 都已确认后才能写分集。每批最多 10 集。每集原则上独立成立，可以推进人物关系、机构秘密或社区长线，不要求 cliffhanger。

## 每集必须回答

1. `synopsis`：这一集发生什么？使用克制叙述体，不写正式对白。
2. `episodePurpose`：这一集为什么存在？使用开放字符串数组。可参考 `world_reveal`、`character_development`、`institutional_satire`、`historical_observation`、`relationship_progress`、`long_arc_progress`、`atmosphere`、`setup`、`transition`。
3. `audienceGain`：观众看完多理解了什么（`newUnderstanding`）？本集推进了什么（`storyProgress`）？
4. `payoffs`：是否兑现已确认的结构性叙事回报？不得自行新增或更改 payoff。
5. `comicBeats`：至少提供一个有效荒诞/滑稽执行单元，或引用至少一个已确认 payoff。可两者兼有。Comic beat 不要求 setup-payoff 因果链，也不需要逐项用户确认。
6. 长线：是否推进弱连续的人物、制度、世界、历史或关系线；没有推进时不必硬加悬念。
7. `endingPull`：结尾用问题、世界线索、人物反应、反讽画面、长线牵引，或 `none`。完整的荒诞镜头和克制反应可以自然收束，不要人为制造重大悬念。
8. `productionRisk`：每个新版 episode 必填，标注 AI 视频生成风险等级与原因；没有明显风险时填写 `{ "level": "low", "reasons": [] }`。旧 episode 可缺省以兼容存量数据。

## Comic beat 写法

使用 `{kind, beat, function}`，kind 是轻量标签，可以自定义合理值，不建立 ontology 或大型 taxonomy。描述一个简短动作、行政细节、视觉荒诞或反应及其叙事功能；不要扩写成正式对白。

例如：“女主以严肃行政态度提醒 Karl 谨慎回答”，不要在 outline 中写完整台词。正式对白由下游 script skill 撰写。

## 生产与连续性约束

- `sceneIds`、`characterIds`、`propIds` 只能引用已确认骨架中的 ID。
- `episodePurpose` 为字符串数组；`audienceGain.newUnderstanding` 与 `audienceGain.storyProgress` 在正式 episode 中都应有内容。
- `endingPull.kind` 可使用 `question`、`world_clue`、`character_reaction`、`ironic_image`、`long_arc`、`none`。`none` 不失败。
- 每集至少引用一个 `status: confirmed` 且 `userConfirmed: true` 的 payoff，或包含一条字段齐全的 comicBeat。
- 三人以上角色若确实同框，填写 `crowdPlan`；分场出现则说明不是同框。
- 生产风险字段可放在 episode 或 scene，结构为 `{level: low|medium|high, reasons: []}`。推荐覆盖车辆运动、多人互动、复杂空间调度、道具交接、透明角色、移动门、地图文字、服装连续性、角色一致性、天气和复杂运镜等风险。
- 只写本批负责的集数范围，合并时按集数排序。
- 新写 `endingPull`，不再依赖旧 `hook` / `suspense`。不得擅自改变项目 Canon。
