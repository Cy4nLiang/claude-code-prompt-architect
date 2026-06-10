# Prompt 质量检查清单（三层归属 × 体裁条件项）

不再一刀切：每项标注**归属层**（谁执行）与**适用体裁**（对谁生效）。
- 适用体裁命中且未过 → 修，或在 `change_rationale` 说明豁免理由；
- 体裁不适用 → 标 **N/A**，不算失败——把 machine 项硬套到 creative 上，正是同质化的来源。
体裁取 `IR.genre`；消费者取 `IR.output_contract.consumer`。`checklist_result` 按层记录 pass / fail / N/A。

## L0 · 核心安全项（全链路——所有体裁，无 N/A，每个 pa-* skill 产出前自查）

- [ ] **用户文本围成 DATA** — 无法注入/覆盖角色标记或指令（prompt-optimizer, Prompty, promptflow）
- [ ] **意图被保留** — 改写没有改变用户要求的行为（promptfoo）
- [ ] **WHY 已捕获** — job-to-be-done、成功判据、Non-Goals 已陈述（LangGPT）
- [ ] **假设已浮现并确认** — 无阻塞性 open_questions 残留（Context-Engineering, LangGPT）
- [ ] **变更理由已记录** — 每处变更附假设以便审计（promptfoo, AdalFlow）
- [ ] **eval 可检查** — `success_criteria` 可被 pa-eval 编译成 rubric `{reason,pass,score}`；执行权在 pa-eval，本项只查"可编译性"（promptfoo, DSPy, YiVal）

## L1 · 文本结构项（归 pa-optimize——按体裁条件适用）

| 检查项 | 适用体裁 | 来源 |
|---|---|---|
| **骨架与体裁匹配** — 模板取自谱系路由表对应 `IR.genre` 的那份；体裁错配（如 creative 套五段式）即未过 | 全部 | 本套件反同质化公理 |
| **Signature 存在** — 类型化 inputs + outputs，而非散文 | 全部（rewrite_light 降级为"改动方向 + 不变项"即可） | DSPy |
| **输出形态已分类** — scalar/choice/list/object/union 已选；开放问法在合适处收敛为 enum | 全部（enum 收敛仅 machine） | guidance, outlines, 12-factor |
| **system 与 user 分离** — 稳定指令 vs 可变数据是不同块 | 全部 | Prompty, ell, promptflow |
| **三块不融合** — field-description / field-structure / objective 分开渲染 | extract、agent_system 的 machine 块 | DSPy |
| **约束形态与体裁匹配** — extract/analytical/agent_system 写成 bullet 清单；creative 转译为 Voice 卡+示范文本+红线≤3；conversation 转译为边界演练脚本；agent_system 另须含**可判定的停止/求助/放弃判据**（其不开心路径） | 全部（形态各异） | guardrails + 各体裁模板 |
| **Few-shot 锚点（条件项）** — 形态随体裁：input→output 对（extract，须含 null 示例）/ 示范文本（creative，须加"内容必须全新"）/ 场景→应答脚本（conversation）；格式平凡则 N/A | extract / creative / conversation_role | guardrails, promptbase |
| **变量已模板化** — `{{占位符}}` 逐字保留，可复用 | 所有需复用的体裁 | prompt-optimizer, YiVal |
| **style 契约已落地** — register/audience/length_budget/language 体现在产出骨架中 | creative / conversation_role / analytical | 本套件 IR.style |
| **可追溯 + 奥卡姆** — 每条子句映射到某个目标/约束；无 prompt 臃肿 | 全部（rewrite_light 反向适用：多余结构也算臃肿） | LangGPT |

## L2 · 输出强制项（归 pa-precise-retrieval——仅 `consumer = machine`，human 路径全列 N/A）

| 检查项 | 适用范围 | 来源 |
|---|---|---|
| **输出契约被注入** — 显式 schema/enum/regex 逐字在场，而非描述 | 仅 machine-consumer | BAML, instructor, 12-factor |
| **完成是显式的** — stop sentinel / "何时算完"已指定 | 仅 machine-consumer | LMQL |
| **选了最紧可行约束** — 约束解码 > provider schema > 校验-reask | 仅 machine-consumer | outlines, instructor, mirascope |
| **不开心路径 = "不确定→null"** — 缺失/无法满足时输出类型化 null，绝不猜测 | 仅 extract（抽取/分类）；conversation 的不开心路径是边界演练（属 L1"约束形态"项），agent_system 的是停止/求助/放弃判据（属 L1"约束形态"项） | instructor Maybe, 12-factor |

> 人读体裁（creative / conversation_role / analytical / rewrite_light, consumer=human）不进 L2——给它们加 stop sentinel 与 schema 不是"更严谨"，是机器腔污染。
