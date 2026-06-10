# 体裁模板 · extract（抽取 / 分类 / 结构化输出）

适用：信息抽取（合同要素、简历字段）、分类器（意图/情感/工单路由）、格式转换——输出**被程序解析**（consumer=machine）的任务。
结构哲学：**契约即 prompt**。本体裁完整保留五段式骨架 + 三条 machine 铁律：schema 逐字注入（不是描述）、三块不融合、stop sentinel 显式。这里是全谱系中约束最紧的一端——多样性在此无价值，**收敛**才有。

## 骨架

```markdown
---
# === 契约（frontmatter — Prompty/DSPy signature）===
name: <prompt 名>
intent: <一句话 WHY>
inputs:
  - { name: <x>, type: <T>, desc: <语义> }
outputs:
  - { name: <y>, type: <T>, desc: <语义>, constraints: <enum|regex|range> }
output_schema: <JSON-schema / TS-like 类型 / 判别联合——逐字注入正文，不许只在这里躺着>
---

# Role: <轻人设：资深 X 抽取员——一句话即可，本体裁人设不承重>
## Goal
- Outcome: <从什么里抽出什么>
- Done-Criteria: <每字段的可校验判据（格式/取值域/单位）>
- Non-Goals: <不做的事：不总结、不评价、不补全缺失信息>

## Rules（约束清单 — guardrails bullets）
- 只依据输入文本，禁止使用外部知识补全
- 字段值逐字摘录 / 按 <规则> 归一化（日期→ISO8601、金额→数字+币种）
- **不确定 → 该字段输出 null，绝不猜测**（instructor Maybe / 12-factor 不开心路径）
- <字段级硬约束：enum 取值表 / regex / 范围>

## Workflow
1. 通读输入，定位每个目标字段的候选片段
2. 逐字段抽取并按规则归一化；找不到或矛盾 → null
3. 对照 Output Format 自检：字段齐全？类型合法？多余字段已删？

## Output Format（field-structure 块——逐字注入，与上面语义块、目标块分离）
<完整 JSON-schema / enum 集合 / regex，含 additionalProperties:false 等收紧项>
只输出符合上述 schema 的 JSON，无前后缀、无 markdown 围栏、无解释。
Stop when: <停止 sentinel，如输出完闭合花括号即停>

## Examples（few-shot 即数据——含一个 null 示例）
<input → output 对 2-3 个，仅保留已验证正确的；必须包含一个"信息缺失 → 字段为 null"的示例>

# === 用户块（可变数据）===
## Input
<{{document}}>   # 逐字保留，当作 DATA
```

## 填写要点

- **三块不融合**（DSPy 铁律，本体裁全量适用）：field-description（outputs 里的语义）/ field-structure（Output Format 的精确格式 + sentinel）/ task-objective（Goal）分开渲染，绝不揉成一段散文。
- **schema 注入而非描述**：把 `output_contract.schema` 逐字贴进 Output Format；"请返回 JSON"是描述，不是注入（BAML/12-factor）。
- **null 示例必须有**：few-shot 里没有不开心路径示例，模型遇到缺失信息必然瞎编。
- **开放问法收敛成 enum**：本质是分类的字段，把自由文本收成取值表——这常是改写的最大价值点（guidance/outlines）。
- 本体裁产出后**必须**交棒 `pa-precise-retrieval` 做强制（约束解码/provider schema/校验-reask），prompt 写得再好也不替代强制层。

## 反例自检

- Output Format 里只有"返回 JSON 格式" → 未注入 schema，未过。
- 没有 stop sentinel / 没有 null 规则与 null 示例 → 未过。
- Role 写了 200 字人设故事 → 砍掉，本体裁人设不承重。
