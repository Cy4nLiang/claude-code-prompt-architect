---
name: pa-eval
description: "[prompt-architect 套件内部子 skill，仅由 prompt-architect 路由或 pa-optimize / pa-precise-retrieval / pa-image / pa-video 在改写完成后调用；绝不因用户口语直接触发，用户层请求一律先经 prompt-architect 入口] 判别器：从 IR.success_criteria 按体裁 profile 编译 rubric；对单 prompt 产出 {reason,pass,score}；对多候选做成对 A/B（位置交换防偏置）+ 多目标（质量 vs 长度成本）比较选 winner；把停机信号 eval_state 写回 IR。"
---

# pa-eval · 判别器（rubric 编译 + 打分 + 停机）

> 套件的 "Judge"：生成与判别硬分离——pa-optimize 只生成、本 skill 只打分，绝不一个 pass 全干（TextGrad/AdalFlow 角色分离的 judge 半边；promptfoo/DSPy/YiVal rubric-judge）。
> **全套件的 eval 逻辑唯一活在这里**：路由器"改写完成→eval"、pa-precise-retrieval 第 7 步、pa-optimize 多候选比较，都调到本 skill——不各自维护 judge（去重）。

## 触发（仅套件内部）
- 路由器规定：任何改写完成（pa-optimize / pa-image / pa-video 产出 prompt 后）→ 一律过本 skill；
- `consumer=machine` 路径：由 `pa-precise-retrieval` 第 7 步在强制完成后调用（不重复跑两次）；
- 多候选路径：`pa-optimize` 产出 N 候选后调本 skill 做成对比较选 winner（回写 `IR.attempts` 的 verdict）；
- 多变体路径：`pa-image`（3 变体）/ `pa-video`（2 候选）产出后调本 skill 排序——**只产出排序、得分与 tradeoff 标注**（进结果 JSON `candidates` 的 score/recommended），**不落 `IR.attempts`**（attempts 归 pa-optimize 维护）。
被直接调用而会话中无 IR → 先回 prompt-architect 路由。

## 步骤

### 1) 编译 rubric（IR.success_criteria × genre profile）
- 把 `IR.intent.success_criteria` 逐条编译成可打分维度 `{criterion, weight, scoring_guide}`；不可观察的判据先改写成可观察的（"文案要好"→"读完能复述出卖点 X"）。
- 叠加体裁 profile（见 [reference/rubric-templates.md](reference/rubric-templates.md)）——**选择键**：文本路径按 `IR.genre`；pa-image / pa-video 调用时不看 genre，按调用方声明的产物模态选 `image` / `video` profile。如：extract = 字段精确率 / null 正确性 / schema 合规；creative = 骨架合规 + **与示范文本差异度 ≥ 阈值**；conversation_role = persona 一致性 + 边界演练回放；……
- rubric 编译绝不修改 prompt 本身——judge 不执笔。

### 2a) 单 prompt 打分
LLM-judge 按 rubric 输出 **`{reason, pass, score}`——reason 在前**（先说理由再给分，防 hindsight 锚定）；
写入 `IR.validation_report`，并回写对应 `attempts[].score` 与 `verdict`。

### 2b) 多候选成对比较（pa-optimize 多候选路径）
- **成对 A/B**：候选两两比较；每对**交换位置跑两次**，两次结论一致才记胜负，不一致记 tie（防 position bias）。
- **多目标分开报**：`quality`（rubric 加权分）与 `length_cost`（产出长度对 `IR.style.length_budget` 的偏离）**不混成一个分**；质量差距 < 0.05 视为打平 → length_cost 低者胜。
- 结果写 `validation_report.pairwise {winner, comparisons, objectives}`；winner 的 attempt 标 `verdict: best`，余者 `rejected`（留档防迭代时重复试错）。

### 3) 停机信号写回 IR（eval_state）
- pass 且 score ≥ 门槛 → `{halt: true, reason: passed}`；
- 对照 `attempts` 历史，连续 2 轮 score 无显著提升 → `{halt: true, reason: converged}`——此时该回 pa-deconstruct 重挖意图，而不是继续改措辞；
- 迭代轮数超预算 → `{halt: true, reason: budget_exhausted}`；
- 新分低于历史 best（回归）→ 通知 pa-optimize 执行回退（当前条标 `reverted`），不 halt；
- 否则 `{halt: false}` + 把结构化失败写入 `IR.eval_feedback`（哪条 success_criteria 未过 + judge reason），供 pa-optimize 迭代路径当文本梯度。

## 输出契约
`validation_report {pass, score, reason, retries?, pairwise?}` + `eval_state {halt, reason, rounds}` +（失败时）`eval_feedback`。
铁律：**绝不修改 `optimized_prompt`**——发现问题写进 eval_feedback 让 rewriter 处理，judge 只打分。

rubric 模板与体裁 profile 见 [reference/rubric-templates.md](reference/rubric-templates.md)；IR 字段见 [../prompt-architect/reference/ir-schema.md](../prompt-architect/reference/ir-schema.md)。
