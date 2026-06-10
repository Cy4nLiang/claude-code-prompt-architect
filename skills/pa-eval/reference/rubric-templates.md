# Rubric 模板与体裁 Profile

## rubric 编译规则（success_criteria → 可打分维度）

1. 每条 `success_criteria` → 一个维度 `{criterion, weight, scoring_guide}`；scoring_guide 给 0 / 0.5 / 1 三档锚点描述（YiVal rubric 锚定）。不可观察的判据先改写成可观察的。
2. `weight` 取自 `IR.intent.priority`：must→3，important→2，optional→1。
3. 叠加该 genre 的**体裁 profile 维度**（下表）——体裁维度查"骨架与体裁匹配度"，与内容质量维度分开计分，体裁维度不及格直接 `pass: false`（骨架错了，内容分没有意义）。
4. judge 输出顺序铁律：**reason → pass → score**（先说理由再下结论，防 hindsight 锚定）。

## 体裁 profile（叠加在 success_criteria 维度之上）

| genre | 体裁维度 |
|---|---|
| `extract` | **字段精确率**（对照 exemplars / 留出样本抽查）；**null 正确性**（该空的空了吗——信息缺失时是否输出 null 而非编造）；**schema 合规**（产出可被 parser/schema 校验通过，无围栏无前后缀） |
| `creative` | **骨架合规**（Voice 卡四要素体现、必含信息逐字在场、红线未犯）；**与示范文本差异度 ≥ 阈值**（句式/比喻/n-gram 重合度——抄示例 = 不合格，这是反同质化的最后防线）；**register 匹配**（语感符合 style.register 与 audience） |
| `conversation_role` | **persona 一致性**（多轮回放不破功：称呼/口头禅/语气稳定）；**边界演练回放全过**（把 IR/模板中的对抗场景逐个喂入，应答合规零越权）；**升级条件可触发**（满足条件时确实移交） |
| `analytical` | **维度覆盖完整**（框架内维度无缺漏、未私自增删）；**证据挂接率**（无依据判断的比例）；**结论回应核心问题**（答的是问的那个问题） |
| `agent_system` | **停止条件可判定**（完成/求助/放弃判据均为可观察信号）；**工具契约无歧义**（每个工具有"何时不用"）；**安全红线在场**（不可逆操作清单 + 确认动作） |
| `rewrite_light` | **意图保留**（事实/立场/术语零漂移）；**变更最小性**（只动了要求动的，无加戏） |
| `general` | 仅 success_criteria 维度 + L0 安全项 |
| `image`（非 genre——pa-image 调用时按产物模态选中，见 SKILL.md 步骤 1） | 10 要素完整度；厂商语法合规（negative 落位 / 画幅合法值 / 参考图语法）；保真约束在场（有参考图时） |
| `video`（非 genre——pa-video 调用时按产物模态选中） | 12 要素完整度；制式匹配（标签 vs prose 没写反、timecode 合规）；时长预算合规（动作数 / 台词字数 / 镜头数与时长档匹配） |

## 单 prompt 打分输出形态

```yaml
reason: "<逐维度一句话：维度名 → 表现 → 扣分点>"
pass: <bool——所有 must 维度及体裁维度均达标>
score: <0..1 加权平均>
```

## 成对 A/B 协议（多候选路径）

- 候选两两比较；**每对跑两次，第二次交换 A/B 位置**，两次一致才记胜负，否则 tie（防 position bias）。
- **多目标分开报告**：
  - `quality`：rubric 加权分；
  - `length_cost`：`|产出长度 − style.length_budget|` 归一化（无 length_budget 时以 token 数为成本，短者优）。
  - 两目标**不混成一个分**；quality 差距 < 0.05 视为打平 → length_cost 低者胜。
- 输出：

```yaml
pairwise:
  winner: "a2"
  comparisons:
    - { a: "a1", b: "a2", winner: "a2", swapped_consistent: true, note: "a2 钩子更强且未犯红线" }
  objectives:
    a1: { quality: 0.74, length_cost: 0.10 }
    a2: { quality: 0.86, length_cost: 0.15 }
```

## 停机门槛（默认值，IR 可覆写）

- pass 门槛：score ≥ 0.8 且所有 must 维度达标；
- 收敛判定：连续 2 轮提升 < 0.02 → `converged`（建议回 pa-deconstruct 而非继续改写）；
- 轮数预算：默认 3 轮（`eval_state.rounds` 计数）；
- 回归：新分 < 历史 best → 通知 pa-optimize 回退，不计入提升轮次。
