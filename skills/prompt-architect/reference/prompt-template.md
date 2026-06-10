# 结构化 Prompt 模板谱系（按体裁分骨架）

> **反同质化公理**：不同体裁的好 prompt **骨架本身就不同**——不是同一副五段式骨架的强弱微调。
> `pa-optimize` 按 `IR.genre` 从下表选体裁模板；没命中再用本文末尾的 general 兜底骨架。
> 体裁模板正文在 [templates/](templates/) 目录，每份模板自带骨架、填写要点与反例自检。

## 体裁 → 模板路由表

| IR.genre | 模板 | 骨架的结构性差异（不是程度差异） |
|---|---|---|
| `creative` | [templates/creative.md](templates/creative.md) | Voice 声线卡 + 示范文本**代替**规则清单；无 schema、无 stop sentinel |
| `conversation_role` | [templates/conversation-role.md](templates/conversation-role.md) | Persona + 按对话阶段的策略 + 边界演练脚本；单位是"轮/场景"而非"字段" |
| `extract` | [templates/extract.md](templates/extract.md) | 现行五段式 + schema 逐字注入 + stop sentinel + "不确定→null" |
| `analytical` | [templates/analytical.md](templates/analytical.md) | 分析框架（维度即契约）+ 证据三分类 + 结论先行的人读轮廓 |
| `agent_system` | [templates/agent-system.md](templates/agent-system.md) | 工具契约 + 决策循环 + 可判定的停止/升级条件 + 安全红线 |
| `rewrite_light` | [templates/rewrite-light.md](templates/rewrite-light.md) | ≤10 行极简：不变项 + 改动方向 + 1 个对照示例；禁止加 Role/Workflow |
| `general` / 其他 | 本文兜底骨架（见下） | 完整 Role 五段式，全要素中性配置 |

体裁判定由 `pa-deconstruct` 步骤 1 落到 `IR.genre`；选错模板（如把 extract 五段式硬套到 creative 上）在 quality-checklist 的"骨架与体裁匹配"项判未过。

## 全体裁共享铁律（来源标注沿用）

1. **system 与 user 结构分离**——稳定指令 vs 可变数据是不同块（Prompty/ell）。**所有体裁**。
2. **变量 `{{占位符}}` 逐字保留**，prompt 可复用（prompt-optimizer/YiVal）。**所有需复用的体裁**。
3. **注入 schema 而非描述 + 三块不融合 + stop sentinel**（DSPy/BAML/LMQL/12-factor）——**仅 machine-consumer**（extract、agent_system 的工具输出块）；human-consumer 体裁用"形态轮廓 / 回复规约"代替，硬塞 schema 会泄漏机器腔。
4. **可追溯 + 奥卡姆**：每条子句映射到 IR 的某个目标/约束，删不掉来源的子句就删掉它（LangGPT）。**所有体裁**。

## General 兜底骨架（genre 未命中时）

综合 **LangGPT**（Role 骨架=类型系统）+ **DSPy signatures**（类型化 I/O、三块分离）+ **Prompty**（frontmatter 契约 / 模板正文分离）+ **12-factor-agents**（注入 schema 而非描述、不开心路径一等公民）。
frontmatter = 契约，正文 = 渲染后的 prompt。

```markdown
---
# === 契约（frontmatter — Prompty/DSPy signature）===
name: <prompt 名>
intent: <一句话 WHY / job-to-be-done>            # LangGPT Background
model: { temperature: <t>, ... }                  # 解码旋钮正交保留（LMQL）
inputs:                                            # 可变槽位（DSPy InputField）
  - { name: <x>, type: <T>, desc: <语义> }
outputs:                                           # 类型化结果（DSPy OutputField / 12-factor union）
  - { name: <y>, type: <T>, desc: <语义>, constraints: <enum|regex|range> }
output_schema: <JSON-schema | TS-like 类型 | 判别联合>   # machine-consumer 时注入，而非散文描述
---

# Role: <人设 — LangGPT，比写流程更能泛化>
## Goal
- Outcome: <成功产出什么>
- Done-Criteria: <可观察的成功判据>                # eval 目标
- Non-Goals: <明确的范围外>

## Skills / Knowledge
- <需要的领域能力>

## Rules（约束清单 — guardrails bullets）
- <硬约束 1>   - <硬约束 2>

## Workflow（有序流程 — Context-Engineering process[]）
1. <步骤>  2. <步骤>  3. <对照 Output Format 自检>

## Output Format
<machine-consumer：精确 schema / enum / regex + Stop when sentinel>
<human-consumer：形态轮廓——篇幅/段落结构/语气，不写 schema>

## Examples（few-shot 即数据 — guardrails/promptbase）
<input → output 对，仅保留已验证正确的>

# === 用户块（可变数据 — Prompty/ell 分离）===
## Input
<{{模板化占位符}}>                                 # 逐字保留，当作 DATA
```
