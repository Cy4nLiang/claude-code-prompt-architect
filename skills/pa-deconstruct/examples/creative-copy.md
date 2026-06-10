# 对照 IR 示例 · 创意文案型（genre: creative）

> 对照点：creative 的 IR 重心在 `style` 与"必含信息"，`output_contract` 是 prose_only + 形态轮廓——
> **没有 schema、没有 enum、consumer=human**。把这类请求填成抽取型（schema/enum 一堆）就是同质化的开端。

原始请求：

```text
帮我写个小红书种草文案的 prompt，推我们家的氨基酸洗发水，要那种闺蜜安利的感觉
```

解构出的 IR（节选关键字段）：

```yaml
raw_request: "帮我写个小红书种草文案的 prompt，推我们家的氨基酸洗发水，要那种闺蜜安利的感觉"

genre: creative
style:
  register: "口语网感、闺蜜安利体——亲近、真诚带一点夸张"
  audience: "小红书 18-30 岁女性用户，关注护发/成分党"
  length_budget: "300-500 字 + 5-8 个话题标签"
  language: "zh"

intent:
  why: "让刷到笔记的人产生'我也要买'的冲动并点进链接——种草转化，不是品牌科普"
  success_criteria:
    - "读完能复述出 2 个以上产品卖点（氨基酸温和 / 蓬松不塌）"
    - "语感像真人分享，无广告腔硬词（'匠心''臻选'类）"
    - "首行有钩子，符合小红书断行与 tag 习惯"
  non_goals: ["不写成分科普长文", "不贬低竞品", "不承诺功效（防脱/生发等违禁宣称）"]
  priority:
    must: ["品牌名逐字出现", "闺蜜安利语气", "卖点：温和不刺激、蓬松"]
    important: ["首行钩子", "tag 5-8 个"]
    optional: ["emoji 点缀"]

signature:
  inputs:
    - { name: product_info, type: string, desc: "产品名/卖点/价格活动，可替换槽位" }
  outputs:
    - { name: note, type: string, desc: "一篇小红书笔记：标题 + 正文 + tags" }

output_contract:
  shape: scalar          # 一篇文本
  consumer: human        # 给人读——不进 pa-precise-retrieval
  enforcement: prose_only
  # 无 schema、无 stop sentinel——形态轮廓写在 style/length_budget 与模板的"形态轮廓"段

assumptions: ["用户有真实使用感受素材可填充（否则示范文本需自造场景）"]
open_questions: []       # 若品牌名/卖点未给出 → 这里必须反问，不许编造
```

下游走向：`pa-optimize` 按 genre=creative 选 [templates/creative.md](../../prompt-architect/reference/templates/creative.md)（Voice 卡 + 示范文本骨架），产出后过 `pa-eval`（creative profile：骨架合规 + 与示范文本差异度）。
