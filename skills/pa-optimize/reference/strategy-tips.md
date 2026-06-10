# Strategy Tips 库（多候选改写的策略基因）

`pa-optimize` 改写路径按 `IR.genre` 选 **3 个相互正交**的 tip，各生成一个候选。
每个 tip = 一种**结构性押注**（改变骨架重心，不是换措辞）；hypothesis 必须落到「预期解决 IR 里的哪条 success_criteria / eval_feedback」。

**选取规则**：
1. 必选该 genre 的"本命 tip"（下表✦标注）；
2. 第二个选结构对立面（与本命 tip 改变骨架的方向相反）；
3. 第三个选风险探索项（可能大胜也可能垫底——反正有 pa-eval 兜底）。
两个候选骨架趋同（同段落结构、同约束形态）→ 换掉一个再生成；候选间差异是本机制的全部价值。

## Tip 库

| tip | 一句话定义 | 骨架影响 | 本命体裁 | 何时别用 |
|---|---|---|---|---|
| `persona-driven` | 把约束转译成强人设声线 | Voice/Persona 段成为主体，规则减半 | ✦creative ✦conversation_role | 输出被程序解析时 |
| `exemplar-led` | 示范文本/示例承载规则：精选示例 + 一句总纲 | Rules 几乎消失，示范/Examples 块为主体 | ✦creative（示范文本）/ extract（input→output 对） | 无已验证示例可挂时 |
| `minimal` | 极简指令：砍到只剩意图 + 不变项 + 一个示例 | 全骨架 ≤10 行 | ✦rewrite_light / 轻 creative | 多重约束、多受众任务 |
| `workflow-heavy` | 强化有序步骤 + 每步自检 | Workflow 块为主体 | ✦analytical / agent_system | 单步任务（纯噪音） |
| `outline-locked` | 锁输出轮廓：结论先行 + 固定章节 + 不得增删 | 输出结构块为主体 | ✦analytical | 创意发散场景 |
| `negative-first` | 红线与反例前置：先写"不要什么"再写"要什么" | 红线/反例成段置顶 | 高合规风险场景；屡次越界的迭代轮 | 初稿（会过度防御、压创造力） |
| `dialogue-scripted` | 对话脚本承载策略：边界演练场景扩到 6-8 个 | 场景→应答脚本为主体，策略段萎缩 | ✦conversation_role | 一次性产出任务 |
| `contract-anchored` | 契约即 prompt：schema + 字段注释承载全部语义 | Output Format 极大化，其余段萎缩 | ✦extract / agent_system 工具块 | human-consumer 体裁 |
| `context-rich` | 重背景铺陈：业务上下文、读者画像、反面案例写满 | 背景段为主体，指令短 | creative 长文 / analytical | 上下文不实（编出来的背景是毒药） |

## hypothesis 一句话模板

> 「用 `<tip>`（把骨架重心从 <X> 移到 <Y>），预期改善 <IR.success_criteria 第 n 条 / eval_feedback 中的失败点>，代价是 <tradeoff 一句话>。」

tradeoff 必填——它直接进 HTML 结果页的候选卡片，让用户知道每个方案牺牲了什么（如 persona-driven：风格张力强 / 信息密度低；minimal：零噪音 / 边界场景无防护）。

## 体裁 × 推荐三元组（默认起手，可按 IR 微调）

| genre | 默认三候选 |
|---|---|
| creative | persona-driven ✕ exemplar-led ✕ minimal |
| conversation_role | persona-driven ✕ dialogue-scripted ✕ negative-first |
| analytical | workflow-heavy ✕ outline-locked ✕ context-rich |
| agent_system | workflow-heavy ✕ contract-anchored ✕ negative-first |
| rewrite_light | 单候选 minimal（轻任务开 3 候选是浪费，pa-eval 也省一轮） |
| extract | **单候选**（contract-anchored 唯一解——见 SKILL.md 多候选规则） |
