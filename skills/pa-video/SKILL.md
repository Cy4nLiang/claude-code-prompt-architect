---
name: pa-video
description: "[prompt-architect 套件内部子 skill，由 prompt-architect 在模态=视频时路由调用（用户的视频 prompt 请求请先触发 prompt-architect）] 视频 prompt 专用编译器：消费 IR，定四主轴（文生vs参考驱动/单vs多镜头/有声vs无声/厂商制式），按 12 要素解剖并按厂商公式（Seedance/Veo/Sora/Kling/Hailuo/Wan/Runway/Pika/CogVideoX）渲染 timecode 分镜、lip-sync 对白、@素材语法与 negative 落位。覆盖“把图片/照片做成会动的视频、让照片里的人开口说话、数字人口播”。不适用：剪辑已有视频、实拍分镜脚本、纯文案（用 pa-optimize）。"
---

# pa-video · 视频生成 prompt 优化

> `pa-optimize` 的视频专用版：把 IR 编译成视频生成 prompt，并**按目标厂商适配**。
> 图像 prompt 的契约是"保真 + 构图 + 画幅 + 排除"；视频在此之上加**时长预算 + 分镜时序 + 运镜 + 音频对白 + 跨镜头一致性**。
> 且视频厂商连 **prompt 骨架**都不可通约（Kling 五标签 / Sora Shot List / Veo 八元素 / Seedance prose）——选错厂商或制式，再好的描述也白费。

## 适用场景（不只是广告）
适用于**任何视频需求**：商业广告 / 带货视频 / UGC 口播 / 真人短剧 / 动态海报 / 产品 360° 展示 / 穿搭变装 / ASMR / 科普 / MV / 漫改动画等。"主体"只是一个填空项——本 skill 与具体品牌无关。常见 use-case 的起手式：

| 你想做 | 起手模式 |
|---|---|
| 商业广告 / 品牌宣传 | 多镜头时间轴版模板 + 全局风格锁定 |
| 带货 / 产品展示 | 参考驱动（产品图锚定）+ 单镜头或两镜头 |
| UGC 口播 / 短剧 | 对白优先：台词定时长，lip-sync + 字幕约束 |
| 穿搭变装 / 卡点视频 | 多图引用 + 节奏参考素材（音乐卡点） |
| 动态海报 / 氛围片 | 单镜头版模板 + 风格与光线要素加重 |
| 漫改 / 静图动起来 | i2v：只写运动增量 + Preserve 风格约束 |

## 在网页版 / App 里怎么用（不会写代码时）
1. **要复刻某个具体的人/商品/画风** → 在工具里直接**上传参考素材**，并在 prompt 里写明每个素材的职责（"图 1 是人物形象，保持脸和服装完全一致；视频 1 只参考它的运镜"）；
2. **时长和画幅** → 在界面里**选档位**（5s/10s/15s，9:16/16:9），不要在 prompt 里编数字；
3. 下文的 `@图1 / timecode / seed / negative 落位` 等是给进阶用户的；网页版用户记住一句话：**先选对模型，再按本 skill 的要素把话说全**。

## 为什么单独一个 skill（与图像 prompt 的关键区别）
- **时长是硬预算**：动作数（5 秒 ≈ 3 个动作）、镜头数、台词字数（8 秒 ≈ 12-15 词）全部由时长反推——图像只管空间密度，视频先做"时长 → 信息预算"分配。
- **运镜/景别/转场是新的控制语言**：静态构图升维为"运镜动词 + 速度 + 景别"三件套，且一个镜头只指定 1 种运镜。
- **音画是必填项**：audio-native 厂商（Veo/Sora/Seedance/Wan）上"不写音频 ≠ 没有音频，= 随机音频"（会幻觉出观众笑声之类）——至少要显式声明"无声/纯环境音"。
- **保真升维为跨镜头一致性**：单图"保真"变成跨镜头的脸/服装/道具/风格不漂移，解法分三档（咒语句式 / 文本逐字复述 / 素材契约）。
- **参考素材从图扩展为多模态职责矩阵**：图 + 视频 + 音频都可作参考，每个素材必须显式分配唯一职责（首帧/人物/场景/运镜/节奏/音色…），否则模型自行猜测。

## 视频 prompt 解剖（12 要素，逐项填，缺则标注或用默认）
1. **主体与外观 Subject & Appearance**：是谁/是什么；要跨镜头出现的角色写成**可逐字复用的锚点句**（外形属性写满），多主体用"将图片 N 定义为主体 N"锚定
2. **动作与运动 Action & Motion**：动词 + 程度量化（缓慢/快速）；物理可行、优先低缓连续小动作；**动作数受时长预算硬约束**；抽象情绪改写为身体细节（"悲伤"→"肩膀微颤、低头攥拳"）
3. **场景与环境 Scene & Environment**：地点/时段/环境元素；⚠ i2v 模式下**反转为禁区**（图里已有的不要重复描述）
4. **运镜与景别 Camera & Shot**：运镜动词 + 速度 + 景别三件套；一镜一运镜；词表见 [reference/camera-language.md](reference/camera-language.md)
5. **时序与分镜 Timeline & Sequencing**【视频新增】：单镜头=一段 prose；多镜头=镜头 N×4 维度（运镜/动作表情/空间/音频）；timecode 精确秒数仅对支持的厂商写；见 [reference/shot-sequence.md](reference/shot-sequence.md)
6. **光线与氛围 Lighting & Atmosphere**：光源/方向/氛围（词表在 camera-language.md）
7. **风格与调色 Style & Grading**：媒介/电影感/调色；可用"导演 + 摄影指导 + 声音设计"三件套引用；Sora 系风格放 prompt 最前；多镜头时加全局风格锁定句
8. **音频设计 Audio**【视频新增】：music / ambience / SFX 分块写；**不允许留空**——无声也要显式声明；见 [reference/audio-dialogue.md](reference/audio-dialogue.md)
9. **对白与口型 Dialogue & Lip-sync**【视频新增】：台词逐字 + 说话人 + 情绪标注；台词长度反推镜头时长；lip-sync/字幕约束句式；渲染语法按厂商分裂（冒号语法 / Dialogue 块 / @音频）
10. **参考素材与职责 Reference & Roles**【视频新增】：每个上传素材（图/视频/音频）显式分配**唯一职责**；标准配置 = 角色锚定 + 场景定调 + 运镜参考 + 节奏氛围；i2v 写法 = 运动增量 + Preserve 约束
11. **一致性与保真 + 排除 Consistency, Fidelity & Negative**：跨镜头锁脸/服装/道具/风格（三档解法）；排除项**只挑 3-5 条最致命**，按厂商机制落位（独立行/固定字段/Avoid 句式/内联约束句）；自查矛盾约束（如 8mm 胶片感 × 4K 锐利互斥）
12. **技术参数 Technical**：**时长是第一参数**（它决定全 prompt 的信息预算），其后才是画幅/分辨率/平台档位（用厂商合法值）

## 四条主轴（先定这四条，再写词）
1. **文生视频 t2v vs 参考驱动**（i2v / 首尾帧 / 多模态 @ 引用 / v2v 延长编辑）：决定描述方向**整体反转**——t2v 要全量视觉描述；i2v 铁律是"**只描述运动，不重复图中已有视觉信息**"，保留意图用 Preserve 措辞。要复刻具体人/商品/画风 → 必须参考驱动；i2v 也是 t2v 失败后的标准降级路径。
2. **单镜头 vs 多镜头时间轴**：决定 prompt 骨架形态（单段 prose vs 镜头列表）。≤5s 单镜头单动作；8-15s 才上分镜，并按时长档位选叙事结构（4/8/10/15s）。
3. **有声 vs 无声，及对白有无**：audio-native 厂商必须显式写音频块（防幻觉）；有对白时整个结构被反向约束——台词长度决定镜头时长。
4. **目标厂商格式制式**：标签式（Kling/Sora/Veo/Wan）vs prose（Seedance/Hailuo/Pika/Runway/开源系），含 negative 机制与长度预算——查 [reference/vendor-matrix.md](reference/vendor-matrix.md)。**不要 prose 化标签厂商，也不要标签化/JSON 化短 prose 厂商**。

## 步骤
1. **接 IR**：从 `pa-deconstruct` 拿主体/用途/时长/平台/保真要求（没有就先补一轮解构）。
2. **判四条主轴**：t2v 还是参考驱动？单镜头还是多镜头？有声有对白吗？目标厂商定了吗？
3. **盘点参考素材并分配职责**【视频新增】：列出用户的每个素材（图/视频/音频），各分配唯一职责（首帧/尾帧/人物锚定/场景/运镜参考/节奏/音色/BGM），没有职责的素材不要传。
4. **选厂商**：按需求查 [reference/vendor-matrix.md](reference/vendor-matrix.md)（要原生对白、要多模态参考复刻、要物理真实感、中文生态、要 v2v 编辑还是开源可控），用户没指定就给 2 个推荐 + 理由。
5. **时长预算与分镜规划**【视频新增】：时长 → 叙事结构与镜头数；单镜时长按内容类型启发式；台词字数按秒数预算；查 [reference/shot-sequence.md](reference/shot-sequence.md)。
6. **按该厂商约定编译**：套下方模板，按矩阵处理骨架制式（标签/prose）、timecode 支持度、音频对白语法、negative 落位、参考素材引用语法。
7. **一致性 + 排除**：跨镜头锁定句式或素材契约；排除项 3-5 条按厂商落位；跑一遍矛盾约束 lint。
8. **自检闭环**（precise-retrieval 的视频版）：出片后按 5 维核对——视觉质量 / 时序一致性（脸/服装/道具漂移）/ 动态程度与物理合理（手部/惯性/穿帮）/ 文本对齐 / 音画同步与口型。失败按降级路径修：重申一致性锁 → 提高参考强度或转 i2v → 拆短镜头减动作 → 换更强一致性厂商。
9. **双候选规则（反同质化）**：**风格敏感体裁**（商业广告 / 品牌宣传 / 氛围片 / MV——风格本身是卖点、且方向存在真分歧的需求）→ 默认开 **2 候选**：`电影感 cinematic`（考究运镜、胶片质感、调色叙事）vs `UGC 实拍感`（手持、自然光、生活化口吻）——同一 IR 两套完整 prompt，各标 tradeoff（如 cinematic：质感高级/投放转化未必赢 UGC），交 pa-eval（video profile）排序后都交付。**口播 / 产品演示 / i2v 保真 / 短平快带货 → 单候选**：制式已被内容类型锁死，开候选是浪费。
10. **可复用**：把会变的做成 `{{占位符}}`（主体/台词/场景/时长档），产出可批量的模板。

## 通用视频 prompt 模板（厂商无关骨架，再按矩阵适配）

**A. 单镜头版（≤8s 默认）**
```text
[主体] a <subject with key appearance anchors>.
[动作] <one main action, paced: slowly/rapidly>, physically plausible.
[场景] <location, time of day, 2-3 environment elements>.   ← i2v 时删掉这块
[运镜] <ONE camera move + speed + shot size>, e.g. slow push-in, medium close-up.
[光线/风格] <lighting>, <style/grading>.
[音频] Music: <...>. Ambience: <...>. SFX: <...>.   ← 无声则写 "ambient room tone only, no music"
[排除] <3-5 条最致命，按厂商落位：独立 negative 或写进正文>.
[技术] <时长/画幅用厂商合法档位>.
```

**B. 多镜头时间轴版（10-15s）**
```text
[全局] Style: <全局风格锁定>. Duration: <总时长>. Characters: <可逐字复用的角色锚点句>.
       Consistency: same face, same outfit, same lighting across all shots.
[镜头1] <景别+运镜（仅 1 种）> | <动作与表情> | <空间> | <音频/台词（说话人: "逐字台词", 情绪）>
[镜头2] …（每镜头同样 4 维度；转场写衔接方式：hard cut / match cut / 元素连续性）
[尾部] Technical: <画幅/档位> + <排除项>.
```
> timecode 写法（`[00:00-00:05] Shot 1`）仅在 Seedance/Veo 等支持的厂商使用精确秒数；其余厂商用"镜头 N"序号，**不强写秒数**。

三铁律：**一镜一运镜**；**5 秒内不塞多动作**；**音频字段不留空**。

## 输出契约
- `optimized_video_prompt`：按目标厂商制式渲染好的 prompt（含时长/画幅合法值、negative 落位、timecode 写法、@参考素材引用语法）。
- `vendor_choice`：选了哪家 + 一句理由 + 备选（按四条主轴查矩阵得出）。
- `shot_plan`【视频新增】：分镜计划——镜头数、每镜时长、景别 + 运镜（每镜仅 1 种）、转场；单镜头需求标 `single-shot` 并省略分镜表。
- `audio_spec`【视频新增】：BGM 风格 / 环境音 / SFX / 台词（说话人/逐字内容/情绪/lip-sync 要求）；厂商不支持原生音频时显式置空并注明"该厂商无音频，需后期配"。
- `asset_plan`【视频新增，仅参考驱动时】：素材职责分配表——每个素材的唯一职责 + 该厂商的引用语法（@图1 / [Image1] / 首尾帧参数）。
- `fidelity_checklist`：出片后自检 3-6 项，覆盖视频特有失败模式——跨镜头一致性、物理合理性、音画同步与口型、无字幕水印 logo、时长与动作量匹配。
- `rationale`：主轴判断与取舍说明；若需求超出纯文生能力（必须复刻真人脸/精确商品）→ 点名"用参考素材驱动（Seedance @引用 / Kling 多模态 / Veo 多图一致性）更稳"，不假装文生视频能做到。

厂商差异与逐家改写见 [reference/vendor-matrix.md](reference/vendor-matrix.md)；运镜词表见 [reference/camera-language.md](reference/camera-language.md)；分镜与一致性见 [reference/shot-sequence.md](reference/shot-sequence.md)；音频对白见 [reference/audio-dialogue.md](reference/audio-dialogue.md)。IR 字段见 [../prompt-architect/reference/ir-schema.md](../prompt-architect/reference/ir-schema.md)。

## 产出 HTML 结果页
完成后，把 `kind:"video"` + `intent / optimized_video_prompt(放入 prompts) / vendor_choice(放入 vendor) / shot_plan / audio_spec / asset_plan / fidelity_checklist(放入 checklist) / rationale` 写成结果 JSON 渲染成 HTML 供查看与检查——JSON 契约与脚本定位规则见 [../prompt-architect/reference/render-protocol.md](../prompt-architect/reference/render-protocol.md)（脚本以 SKILL.md 实际所在目录定位，勿硬编码安装路径）。
双候选路径改用 `candidates` 键（label=电影感/UGC 实拍 + strategy/tradeoff/content）走多方案 grid 对比渲染。
