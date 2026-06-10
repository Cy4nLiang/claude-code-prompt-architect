---
name: pa-precise-retrieval
description: "[prompt-architect 套件内部子 skill，仅由 prompt-architect 路由调用] 输出强制与 eval 门禁：当 IR 的输出契约为机器可解析形态（枚举/列表/对象/判别联合）且输出将被程序消费时，按模型能力编译最紧强制策略（约束解码 > provider 结构化输出 > 校验+reask），失败升级 reask，最后调用 pa-eval 对照 success_criteria 打分写回。"
---

# pa-precise-retrieval · 精准取数 / 精确抽取

> 套件的 "Generator + Guide + Runner"：把意图编译成最紧约束并按模型能力强制执行。
> 借鉴 outlines/guidance/LMQL（约束解码）、instructor/mirascope（模式多态）、BAML（SAP 容忍解析）、guardrails（最小差量 reask）、promptbase（投票）、promptfoo（judge）。

> 网页版 / 不会写代码时：约束解码、provider schema 这些需要 API 或开发环境；你在普通聊天框里能做的"精准取数"就是——**把输出格式用示例钉死 + 一句"严格按此格式、不确定就留空别编" + 自己核对一遍**，不达标就把哪里不对说清楚让它重发。下文 API 细节可作进阶参考。

## 触发
`output_contract.consumer = machine`（输出将被程序解析）且 `output_contract.shape ∈ {choice, list, object, discriminated_union}`，或 `genre=extract`。输出给人读（consumer=human）的创意/对话任务**不进**本管道。被直接调用而会话中无 IR 时，先回 prompt-architect 路由。

## 输入
IR（尤其 `output_contract`）+ 来自 `pa-optimize` 的 `optimized_prompt` + 解码控制标记（能否 mask logits / 用 provider 结构化输出？）。

## 步骤

### 1) 编译到最紧约束（outlines）
能用 enum/Literal 就别用自由文本；能用 regex/JSON-schema 就别写"请返回 JSON"。**更窄的语言 = 更小的失败面**。

### 2) 按模型能力选强制策略（策略多态 — instructor/outlines/mirascope）
| 能力 | 策略 |
|---|---|
| 调用方控制解码 | **约束解码**：把契约编译成 grammar/FSM/logit-mask，违规不可表示（guidance token-mask / LMQL `where` / outlines FSM Index） |
| 有 provider 结构化输出 | **provider schema**：严格 `json_schema`（`extra=forbid` / `additionalProperties:false`）或强制 tool-call（instructor TOOLS / prompty `response_format`） |
| 仅散文模型 | **校验-reask**：挂容忍解析器 + 校验器 |

### 3) 显式声明完成（LMQL finality）
停止短语 / sentinel / "这就是你做完的标志"。

### 4) 容忍解析（BAML SAP）
剥 markdown 围栏、忽略 CoT 前缀、单值↔数组互转；多个合法解释时按"偏差罚分"排序选最低偏差者，而非硬失败。

### 5) 校验-reask 循环（instructor / guardrails）
失败时把**精确结构化错误**（"字段 X 失败因为 Y，期望 Z"）作为下一轮喂回；**最小 reask**——只重问坏掉的子字段，按 path 合并回（guardrails 最小差量）。
有限重试；**反复解析失败就升级强制**（干净格式 → JSON 模式 → 约束 schema，DSPy 逐级降级；promptbase 升温重试）。

### 6) 鲁棒性集成（可选，高风险场景）
采样 N 次，扰动表面形式（选项顺序/温度），对去扰动后的答案多数投票，把投票方差作为置信信号（promptbase choice-shuffle）。

### 7) eval 门禁 → 调用 pa-eval（去重：judge 逻辑只活在 pa-eval）
强制完成后把 IR + 产出交 **pa-eval**：由它编译 rubric（extract profile：字段精确率 / null 正确性 / schema 合规）、打分 `{reason, pass, score}` 写 `validation_report`、写停机信号 `eval_state`；
失败时 pa-eval 写 `eval_feedback`，本 skill 把它连同强制层上下文（解析错误/重试历史）一起回传 `pa-optimize` 迭代。**本 skill 不再自带 judge**。

## 输出契约
二选一：
- **(a) 强制规格**：调用方可直接运行的 grammar/regex/JSON-schema + stop + parser + reask 策略，与 prompt 一起打包；或
- **(b) 抽取值**：若本 skill 也执行调用，则直接给出抽取到的类型化值 + `validation_report`（`{pass, score, reason, retries, confidence}`）。

硬失败时：返回**类型化失败对象**，绝不返回静默垃圾。

IR 与契约字段见 [../prompt-architect/reference/ir-schema.md](../prompt-architect/reference/ir-schema.md)。
