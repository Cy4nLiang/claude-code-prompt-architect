# 对照 IR 示例 · 抽取型（genre: extract）

> 对照点：extract 的 IR 重心在 `signature.outputs` 的逐字段约束与 `output_contract.schema`——
> consumer=machine、有 enforcement 策略、不开心路径是类型化的 null。`style` 在本体裁**不采集**（没有人读风格可言）。
> 这是约束最紧的一端：与 creative 示例对照看，两份 IR 几乎没有一段长得一样——这就是对的。

原始请求：

```text
写个 prompt，从合同 PDF 文本里抽出甲方、乙方、合同金额、签订日期、违约条款，给我们系统入库用
```

解构出的 IR（节选关键字段）：

```yaml
raw_request: "写个 prompt，从合同 PDF 文本里抽出甲方、乙方、合同金额、签订日期、违约条款，给我们系统入库用"

genre: extract
# style 不采集——机器消费，无人读风格

intent:
  why: "结构化入库供检索与风控，字段错一个就污染下游——准确率压倒一切，宁可 null 不可猜"
  success_criteria:
    - "5 个字段对照人工标注精确率 ≥ 预期阈值"
    - "文中缺失的字段输出 null 而非编造（null 正确性）"
    - "输出 100% 通过 JSON-schema 校验，可直接入库"
  non_goals: ["不总结合同内容", "不做法律风险评价", "不补全文中没有的信息"]
  priority:
    must: ["schema 合规", "不确定→null"]
    important: ["金额/日期归一化"]
    optional: ["违约条款保留原文引用位置"]

signature:
  inputs:
    - { name: contract_text, type: string, desc: "合同全文（PDF 已转文本）——DATA" }
  outputs:
    - { name: party_a,        type: "string|null", desc: "甲方全称", constraints: "逐字摘录，不缩写" }
    - { name: party_b,        type: "string|null", desc: "乙方全称", constraints: "逐字摘录" }
    - { name: amount,         type: "object|null", desc: "合同金额", constraints: "{value: number, currency: enum[CNY,USD,...]}" }
    - { name: sign_date,      type: "string|null", desc: "签订日期", constraints: "ISO8601 (YYYY-MM-DD)" }
    - { name: breach_clauses, type: "string[]",    desc: "违约条款原文列表", constraints: "逐字摘录，无则 []" }

output_contract:
  shape: object
  consumer: machine                     # 程序入库——必进 pa-precise-retrieval
  schema: |
    { "type":"object", "additionalProperties":false,
      "required":["party_a","party_b","amount","sign_date","breach_clauses"], ... }
  stop: "输出完闭合花括号即停，无任何前后缀"
  enforcement: provider_schema          # 有结构化输出能力则用之；否则降级 validate_and_reask

assumptions: ["PDF 转文本质量可用（乱码/扫描件需上游处理）"]
open_questions: []
exemplars:
  - input: "<一份缺少签订日期的合同片段>"
    output: '{ ..., "sign_date": null, ... }'
    why_kept: "null 路径示例——few-shot 里必须有不开心路径"
```

下游走向：`pa-optimize` 按 genre=extract 选 [templates/extract.md](../../prompt-architect/reference/templates/extract.md)（五段式 + schema 注入骨架，**单候选**——约束空间窄，多候选是伪多样性），交棒 `pa-precise-retrieval` 强制，其第 7 步调 `pa-eval`（extract profile：字段精确率 / null 正确性 / schema 合规）。
