---
name: pa-optimize
description: "[prompt-architect 套件内部子 skill，仅由 prompt-architect 路由调用，不要因用户口语直接触发] 把 IR 编译成结构化 prompt：按 IR.genre 选体裁骨架（模板谱系，fallback general），默认 3 候选×正交策略假设交 pa-eval 择优（extract 与 rewrite_light 单候选）；schema 注入、few-shot 即数据、变量模板化；带 eval_feedback 时执行 critic→rewriter 最小编辑。触发条件：IR 已就绪且无阻塞 open_questions，或迭代路径（已有 prior optimized_prompt + 反馈）。会话中无 IR 时先回 prompt-architect。"
---

# pa-optimize · 改写 / 脚手架

> 套件的 "Adapter + TGDOptimizer"：把 IR 契约渲染成结构化 prompt；若有历史尝试，则做一次 critic→rewriter。
> 借鉴 DSPy Adapter、LangGPT 骨架、AdalFlow 槽位语法、BAML schema 注入、guardrails 约束清单、TextGrad 角色分离、promptfoo 假设。

## 触发
- IR 有完整 `signature` + `intent` 且无阻塞 `open_questions`；
- 或迭代路径：IR 中已有 prior `optimized_prompt` + `eval_feedback`（由 `pa-eval` 在 eval 失败时回写）/ 用户反馈。

## 步骤（改写路径——默认 3 候选）

**多候选规则（反同质化：一次押三种结构假设，让 pa-eval 选，而不是一条路写到黑）**：
- 非 extract 体裁默认产 **3 个候选**：从 [reference/strategy-tips.md](reference/strategy-tips.md) 按 genre 选 3 个**相互正交**的 strategy tip，每个候选绑定一句 hypothesis（「用 X 策略改变骨架哪里，预期解决 IR 的哪条判据」），各自独立走一遍下方 1-8，记入 `IR.attempts`（a1/a2/a3，`parent_id: null`）。
- 完成后交 **pa-eval** 成对比较 + 多目标打分选 winner → `optimized_prompt`；落选者标 `rejected` 留档（防迭代时重复试错）。
- **extract / consumer=machine 维持单候选**（合理的统一）：约束空间窄，多候选只会产出同一骨架的微调——伪多样性；预算花在 pa-precise-retrieval 的强制上更值。`rewrite_light` 同样单候选——轻任务开 3 候选是过度工程。
- 迭代路径（带 eval_feedback）不开新 3 候选，仍走 critic→rewriter 最小编辑。

1. **按 `IR.genre` 选体裁模板，fallback general** — 按模板谱系路由表（[../prompt-architect/reference/prompt-template.md](../prompt-architect/reference/prompt-template.md)）的链接取对应模板文件（genre 值的下划线在文件名中写作连字符：`conversation_role`→`templates/conversation-role.md`，余者同理）：creative=Voice 卡+示范文本骨架、conversation_role=Persona+对话策略+边界演练、extract=五段式+schema、analytical=分析框架+证据规则、agent_system=工具契约+决策循环、rewrite_light=≤10 行极简；未命中用 general 兜底。**禁止把 extract 五段式硬套到人读体裁上——体裁错配即同质化**，checklist"骨架与体裁匹配"项判未过。
2. **三块分离，绝不融合**（DSPy）：field-description（语义）/ field-structure（精确格式 + 停止 sentinel）/ task-objective。〔machine 块适用：extract / agent_system 工具块；人读体裁见各模板的等价结构〕
3. **分解后再组合**（guidance / Context-Engineering）：把复杂输出拆成独立指定的子部分，再拼回。
4. **约束写成 bullet 清单**（guardrails `validator.to_prompt`），别埋进散文。〔extract/analytical/agent_system；creative 转译成 Voice 卡+示范文本，conversation 转译成边界演练脚本〕
5. **few-shot 即数据**（guardrails / promptbase / DSPy demos）：挂已验证正确的示例——**形态随体裁**：extract 是 input→output 对（含 null 示例 +"不确定→null"规则，仅本体裁）；creative 是示范文本（加"内容必须全新"）；conversation 是场景→应答脚本。
6. **注入输出契约**（BAML / instructor）：把 `output_contract.schema` 逐字作为主指令注入；优先紧凑 TS-like / JSON-schema，而非"请返回 JSON"。〔仅 consumer=machine；人读体裁写"形态轮廓/回复规约"（取自 `IR.style`），不写 schema〕
7. **模板化变量**（prompt-optimizer / YiVal）：保留 `{{占位符}}` 为可复用槽位，逐字保留，绝不展开。
8. **可追溯 + 奥卡姆剪枝**（LangGPT）：删掉任何无法映射到某个目标/约束的子句，防 prompt 臃肿。

## 步骤（迭代路径，带反馈时）

执行**两段 critic→rewriter**（TextGrad / AdalFlow 角色硬分离）：
1. **critic**：只产出一段"文本梯度"——当前 prompt 哪里失败、与期望差什么，**不改写**。
2. **rewriter**：只应用**一种**最小编辑法（新增 / 加示例 / 改写替换 / 删除），不大改。
3. **读写 `IR.attempts`**（版本管理，YiVal OPRO + ell 内容寻址思路）：进入迭代前先读 attempts 了解已试过什么；每轮 rewriter 产出**追加**一条 `{id, prompt, hypothesis, score, verdict, parent_id}`（`parent_id` 指向被编辑的版本）；eval 分数回写 `score`；**re-eval 回归就回退**——当前条标 `reverted`，恢复 `verdict: best` 的版本（TextGrad/AdalFlow validation-gate）。
4. 重复反馈时强制更大改动（看 attempts 里 rejected 的 hypothesis，避免在同一处打转）。

## 输出契约
- `optimized_prompt`：role-tagged 消息（system + user），含输出格式/schema 块、约束清单、≤K 个 few-shot 对、保留的 `{{占位符}}`、停止 sentinel；无开头废话、无多余代码围栏（prompt-optimizer）。
- `change_rationale`（promptfoo 假设）：列出改了什么、为什么，每条可追溯到 IR 的某个目标。
- **意图保留**是硬约束：改写**不得改变**用户要求的行为（promptfoo `OPTIMIZE_PROMPT` 铁律）。

## 发布门禁
产出前对照 [../prompt-architect/reference/quality-checklist.md](../prompt-architect/reference/quality-checklist.md) 逐项过；
未过项要么修，要么在 `change_rationale` 注明豁免理由。

## 何时交棒
改写完成后**一律先过 `pa-eval`**（路由器规定：打分 + 停机信号写回；多候选时由它成对比较选 winner）。其中：
- `output_contract.consumer = machine`（输出将被程序解析：值/枚举/列表/对象喂代码、API、流水线）或 `genre=extract` → 交棒 `pa-precise-retrieval` 做强制，pa-eval 由其第 7 步调用（不重复跑）；
- 输出给人读（consumer=human）→ 本 skill 产出 → pa-eval 打分 → 终点，不进强制管道。

## 闭环（套件不是一次性的）
指标驱动循环（借鉴 DSPy MIPRO/GEPA、TextGrad/AdalFlow 文本梯度、promptfoo eval-loop）：
1. `pa-deconstruct` → IR（契约 + 体裁 + 开放问题）
2. 本 skill → 候选 prompt（+ 变更假设，记入 `IR.attempts`）
3. `pa-eval` → rubric 打分 `{reason,pass,score}` + 停机信号 `eval_state`（machine 路径经 `pa-precise-retrieval` 强制后由其第 7 步调用）
4. **失败**：pa-eval 写 `eval_feedback` 当文本梯度喂回 → 本 skill 走迭代路径做**最小单点编辑**；best/rejected 记忆在 `IR.attempts`，回归就回退
5. **意图错位**（LangGPT "正骨 vs 修复"）：回 `pa-deconstruct` 重新澄清上游，而非修补措辞
6. 停机由 `eval_state` 判定：passed / converged（分数停滞→建议回解构）/ budget_exhausted，用留出验证防过拟合

**角色始终分离**：critic 只诊断、rewriter 只编辑、judge 只打分（judge 已收敛进 pa-eval）——绝不一个 pass 全干（借鉴 TextGrad/AdalFlow）。

## 产出 HTML 结果页
完成后，把 `intent / signature / optimized_prompt / checklist_result / change_rationale` 写成结果 JSON 渲染成 HTML 供用户查看与检查——JSON 契约与脚本定位规则见 [../prompt-architect/reference/render-protocol.md](../prompt-architect/reference/render-protocol.md)（脚本以 SKILL.md 实际所在目录定位，勿硬编码安装路径）。
**多候选路径用 `candidates` 键**走多方案 grid 对比渲染：每个候选标注 `strategy / hypothesis / tradeoff`（一句话代价说明）与 pa-eval 得分，winner 标 `recommended`——让用户看到"押了哪三种结构、各牺牲了什么"，而不是只看到一个黑箱终稿。
