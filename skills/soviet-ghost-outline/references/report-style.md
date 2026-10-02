# 大纲报告的设计约定

报告由 `scripts/soviet-ghost-outline.mjs render --html` 生成，样式全部内联在 `renderHtml()` 里。**改样式只改那一处。**

## 它是什么

单页离线评审报告，用于审阅故事设计、分集价值、Canon 依赖、场景资产与生产风险。维持冷灰印张与铁锈红印记的视觉语言，图形全部内联 SVG / CSS，不加载外部资源。

## 区块顺序

```text
页眉        故事名、参数、质量门徽章、导出 JSON
KPI 带      集数、payoff、角色分档、主场景、生产风险、故事来源
摘要        payoff / comicBeat / 两者兼有的集数、高生产风险集数、场景复用等级统计
病灶横幅    有未过质量门时逐条显示诊断
01 叙事回报节奏  图 / 表 tab；间隔只作分布信息，不判失败
02 分集概览  显示 synopsis、episodePurpose、audienceGain、payoff、comicBeat、
             endingPull 与 productionRisk
03 场景概览  出现集、reuseClass、承载回报、场景级生产风险
04 关键决策  故事设计取舍、人物位、可选 major payoff 位置
05 每集调度矩阵  角色、场景、道具出现集
06 资产量折算
07 人物表
08 故事设计  核心、Canon refs、纳入 / 排除 / 合并 / 风险与历史研究
09 质量门    完整 ✓ / ✗ 清单
```

**屏幕上收，纸上全展开**：打印时显示所有时间轴 / 表格面板及全部分集卡。离线可打开，内嵌 JSON 可原样导出。

## 区块约定

- **单集概览：**`endingPull.kind = none` 作为合法展示，不显示空悬念占位。旧 `hook` / `suspense` 可在兼容模式作为结尾牵引展示。
- **Payoff：**时间轴按 `level` 标示，不以笑声强弱解释 major / minor。type 与 ontology 多标签按原数据呈现。
- **Comic beat：**单集卡显示 `kind`、简短内容与功能，不把它混进结构性 payoff 时间轴。
- **场景复用：**场景卡显示 `core`、`recurring` 或 `episode_only`。一次性 `episode_only` 场景缺少复用方案时由质量门诊断。
- **Canon 与历史研究：**Canon refs 只显示故事依赖摘要及来源；历史研究表显示 category、claim、status 和 storyImpact。
- **生产风险：**分集 / 场景展示 level 与 reasons，报告只用于后续 AI 视频生产预警，不生成镜头指令。
- **资产量：**角色分档、场景复用统计、道具与生产风险从大纲数据汇总，模型不手填统计数字。

## 质量门

页眉徽章显示通过或失败；未通过时 KPI 带下方弹出诊断横幅，文末列出完整质量门。每集必须至少引用一个已确认 payoff，或包含一个字段完整的 comicBeat；两者兼有可单独统计。节奏轴上的回报间隔只作分布信息，不设硬阈值；不要求首集 hook，也不强制 major payoff。

## 界面文案与安全

界面文字集中在 `I18N`，默认中文，并保留英文界面。不要将模型数据硬编码进文案。大纲数据进入 HTML 前一律 `esc()` 转义；内嵌 JSON 中的 `<` 转义以防 `</script>` 截断。不加载外部脚本或样式，支持打印与减少动效偏好。
