# soviet-ghost-outline

本文件保留旧版 README 路径；当前工作流以 [README.md](README.md) 和 [SKILL.md](SKILL.md) 为准。

本技能服务于《苏联亡灵局》原创系列大纲。单集独立成立并可弱推进长线；payoff 与 comicBeat 分开，每集至少具备其中一种。项目 Canon 由外部事实源维护，outline 只记录 `canonRefs`。新版 episode 必须填写 `productionRisk`，最低可写 `{ "level": "low", "reasons": [] }`。

`beats` 阶段允许 `payoffs: []`；正式分集若没有 payoff，必须由有效 comicBeat 提供单集价值。旧格式仍兼容读取，但新内容应使用 `storyDesign`、`ontologies`、`endingPull` 等 v1.1 字段。
