# 视频生成厂商能力矩阵

> 数据截至 2026-06，各家迭代极快，**最终以官方最新文档为准**。带 ⚠ 的条目为单一信源、推演结论或信源间有分歧。
> 核心结论：**选错厂商比写错 prompt 更致命**——视频厂商连 prompt 骨架都不可通约（五层标签 vs 单段 prose vs Shot List vs 时间码分镜），先按"要不要原生音频对白 / 要不要参考素材 / 中英文 / 时长"选厂商，再按该家骨架写词。

## 一、能力矩阵

| 厂商 / 模型 | 官方公式骨架 | timecode 分镜 | 原生音频 & 对白 | i2v / 参考素材语法 | negative 机制 | 时长 / 分辨率 | 最适合 |
|---|---|---|---|---|---|---|---|
| **Seedance 2.0**（字节，即梦/Dreamina 同引擎） | 8 要素 prose：主体+动作+场景+运镜+光影+风格+音频+画质/约束尾缀；即梦端用中文 8 维度；社区主流是时间码分镜 | ✅ 长视频建议强制分段（`0-3s / 3-6s / 6-10s...`） | 原生：自动音效/配乐 + 台词 lip-sync + `@音频1` 绑 BGM ⚠（一家信源标"需后期"，与多数矛盾） | **业界最全**：`@图1/@视频1/@音频1` 引用体系，≤9 图 + ≤3 视频 + ≤3 音频（共 ≤12 文件），每个素材**显式指派单一职责**（首帧/尾帧/人物/场景/服装/产品/运镜/动作/特效/节奏/音色/BGM…）；另支持视频延长与 v2v 编辑；**禁上传真人脸** | 无独立参数 → 内联 `[NEGATIVE]` 段 / JSON `negative_prompt` 字段 / 尾句 `no text, no watermark` | 4–15s 可选；分辨率 ⚠ 720p（官方文档转写）vs 1080p（社区） | 多素材参考复刻、多镜头叙事广告、爆款短视频 |
| **Veo 3.1**（Google DeepMind） | 官方 8 元素（framing/style/lighting/角色/场景/动作/对白/音频），`Dialogue:` 与 `Audio:` **各自单独成行**；社区扩展为 7 字段含 Technical(Negative) | ✅ `[00:00-00:02]` 时间戳写法 | **业界最强**：多人定向对白、音效逐层列出；quirk①台词用冒号 `says: "..."` 防字幕烧进画面；quirk②必须显式声明环境音防"音频幻觉"（如 `no audience sounds` 防凭空观众笑声） | ⚠ 记载薄：Flow 多参考图入景、首尾帧（回落旧版模型）、同人多照片锁角色 | `Technical (Negative Prompt):` 字段或自然语言 `Avoid ...` | ⚠ 8s 单段 / 60s / 148s chained 三说并存；16:9 为主、竖屏原生支持待确认 | 原生对白 lip-sync、多人对话、带音效的成品片段 |
| **Sora 2**（OpenAI） | Shot List 分层：`Style:`（最强杠杆，放最前）→ 场景 → `Cinematography:` → `Actions:` 逐 beat 列表 → `Background Sound:`；另有电影工业参数化写法；80–150 词最优 | ❌ 官方理念是 **beats > seconds**——动作拆成可计数的节拍，不写绝对秒数 | 原生音画一体：对白嵌在 Actions 里（`Character (tone): "line"`），短句为佳；Sound 段**只写 diegetic 场景内声音**，别点配乐（模型自动配，硬写会冲掉氛围） | 参考图当"选角"锁定长相画风；Cameos 真人客串（需授权）；Remix 定向微调（只改指定细节不重抽） | 无机制 → `Avoid X, keep Y` 自然语言 | ⚠ 25s(Pro)/15s 与"20s"说法内部冲突；⚠ web/app 停服传闻（单一信源） | 电影质感一次成片、高级摄影叙事 |
| **Kling 3.0 / 可灵**（快手） | 三套递进：① 4 部分基础 Subject+Action+Context（环境元素控制在 3–5 个）+Style；② 5 层标签 `Scene:/Characters:/Action:/Camera:/Audio & Style:`；③ i2v 专用只写运动 | 无时间戳语法；时序写成"开头→中间→结尾"动作流；智能分镜功能出长片 | 原生音画同步 + **角色定向发声**（角色 A 说 X、角色 B 说 Y） | **最强 i2v 阵营**：Motion Brush 运动笔刷、首尾帧；铁律=只描述运动、不重述图中已有视觉（重述会触发重画） | **最显式**：界面有独立 negative 输入框；常用负面行 `cartoonish, smooth plastic skin（塑料皮肤）, floating limbs（漂浮肢体）, sliding feet（脚底打滑）, text morphing` | 15s 单段 / 智能分镜约 2 分钟；720p/1080p 两档（测试用低档省点数） | 中文剧情、图生视频、物理感、多角色对话 |
| **Hailuo 02 / 海螺**（MiniMax） | "Director's AI"克制公式：1–2 个主体清晰描述 + 时序动作 + 简洁场景 + 物理细节强调；**忌形容词堆砌**（堆砌必糊） | 无；时序写进自然语言（"镜头先缓缓下降，随后向右环绕"） | ⚠ 需后期（源材料无任何音频写法；新版本是否原生待查官方） | ⚠ 记载空白（subject reference、官方导演模式运镜指令语法均缺失） | 无独立框 → 正向描述替代（"画面清晰，无模糊，人物表情自然"） | 10s / 1080p ⚠（档位与画幅合法值未记载） | 物理真实感（流体/火焰/布料/重力）、微表情情绪转换 |
| **Wan / 通义万相**（阿里） | `Entity / Scene / Motion / Sound` 四段标签；Sound 段是灵魂：voice（给台词自动对口型）+ sfx + bgm 三层可并存 | ⚠ 未证实（唯一记载来自污染信源，已弃用） | 原生音视频；数字人 lip-sync 三要素：Entity 写 talking-head 正对镜头 + Motion 写 `lips synced to dialogue` + Sound 段给具体台词 | ⚠ 实际空白（可信信源未展开首尾帧/参考图） | 无记载 | 15s（2.7）⚠；2.6 开源可本地部署 | 数字人播报、对口型广告 / MV |
| **Runway Gen-4/4.5 + Aleph** | Gen-4 自由文本：subject+action+setting+camera+motion+style+constraints；Aleph 编辑用 6 动词 `add / remove / change / replace / re-light / re-style` + 三原则：说改什么、说保持什么、说改多少 | 无记载 | ❌ 无原生，需后期 | ⚠ Gen-4 的 i2v/keyframes 主流程在源材料中全空白；**Aleph v2v 编辑独家** | 无记载；约束内联写在 prompt 尾部 | ⚠ 10s（Gen-4.5）/ 5s（Aleph 单次编辑），单一信源 | v2v 编辑已有视频：改光、换风格、增删元素 |
| **Pika 2.5** | 单段短句模板 `A [subject] [action] in [setting], [style], [camera], [quality]` + **Pikaffects 特效名词直接当词用**（Melt 融化 / Explode 爆裂 / Squish 挤压 / Levitate 悬浮 / Cake-ify 变蛋糕…）；铁律：单一主体、必加 `no morphing`、显式写 `high quality`（默认输出偏低质） | ❌；Pikaframes 首尾帧补间 | ⚠ "部分支持"，无任何写法细节 | Pikaframes：给首帧 + 尾帧描述，模型补中间过渡 | 内联 `no morphing` 习语为主；据称也接受 `Negative:` 行 ⚠ | ⚠ 25s（场景延长）；基础时长档/分辨率未记载 | 特效感爆款短视频、单主体小品 |
| **其他**：HappyHorse 1.0（阿里千问 App）/ Hunyuan 1.5（腾讯开源）/ LTX·Mochi·CogVideoX（开源）/ Higgsfield | HappyHorse：30–55 词紧凑 + `AUDIO:` 块 + @tag 参考 ⚠（全部单一信源且系视频二手摘编）；Hunyuan：Normal/Master 双模式 prompt rewrite；开源系：单段英文 prose、忌多场景切换；Higgsfield：短指令 > 长描述 + `[Soul ID: name]` 角色占位符 | HappyHorse 用 `SHOT 1 (0:00-0:05)` 时间码 | HappyHorse 原生联合生成；Hunyuan / LTX / Mochi / CogVideoX 均无 | CogVideoX 有专门 I2V 权重；Higgsfield Soul ID 训练后多场景同脸 | 基本无独立机制 | 多在 5–15s 区间；LTX 可实时生成迭代 | 预算敏感 / 本地部署 / LoRA 训练；多场景角色一致性（Soul ID） |

## 二、怎么选（决策指引）

| 你的首要诉求 | 选 | 理由 |
|---|---|---|
| **原生对白 + 多人 lip-sync 成品** | Veo 3.1 | `Dialogue:`/`Audio:` 双标记 + 多人定向发声，音频体系最完整 |
| **多模态参考复刻**（角色/场景/运镜/BGM 全要锁定） | Seedance 2.0 | @ 引用 + 单素材单职责指派，参考体系全行业最全 |
| **物理真实感**（流体/布料/重力/真实动作） | Hailuo 02 > Sora 2 > Kling 3.0 | 物理仿真是 Hailuo 第一卖点，prompt 用物理词不用形容词 |
| **中文生态 / 中式美学 / 国内平台** | Kling 3.0 ≈ 即梦 | 中文理解领先，官方建议直接写中文、忌中英混用 |
| **编辑已有视频**（改光/换风格/增删元素） | Runway Aleph | v2v 编辑动词独家，别家没有这个维度 |
| **数字人播报 / 对口型广告** | Wan（或 Veo 3.1） | Sound 段给台词即自动对口型，talking-head 写法成熟 |
| **电影质感一次成片** | Sora 2 | Style 锚定 + beats 拆解 + 工业级参数化写法 |
| **预算敏感 / 本地部署 / LoRA** | Hunyuan 1.5 / Wan 2.6 / LTX / CogVideoX | 开源可商用，LTX 还能实时迭代 prompt |

> 跨厂商迁移时**先换骨架再润色**：把 Kling 五层标签直接喂给 Sora、或把 Shot List 喂给 Hailuo，效果都会劣化——每家的骨架就是它的"接口签名"。

## 三、同一条 prompt 逐家改写（示例：雨夜咖啡馆，女子放下咖啡杯抬头望向镜头微笑，i2v 带参考图）

改写要点只动骨架与措辞密度，画面锚点不变：

- **Seedance 2.0（prose + @ 引用 + 保真尾缀）**：`@Image1 as the first frame. She lowers her cup to the saucer, gaze drifts back to camera, a faint smile forms（放下杯子、目光移回、浮起微笑）. Rain intensifies on the glass. Slow push-in.` 结尾必加保真句 `Maintaining face and clothing consistency, no subtitles, no watermark`（保持人脸服装一致、无字幕水印）。
- **Veo 3.1（8 元素 + 双标记行）**：一段 prose 写镜头/光线/动作，然后单独两行：`Dialogue: (no dialogue)`、`Audio: rain on glass intensifying, espresso machine hiss, ceramic clink, no music`（雨声渐强、咖啡机嘶声、瓷器轻响、无配乐）——**不写 Audio 行就可能幻听出配乐或观众声**。
- **Sora 2（Shot List 分层）**：先 `Style: 35mm cinematic, warm-cool color grade`（35mm 电影感、暖冷撞色）锚定；动作拆 beats：`- She lowers her cup` `- Gaze drifts to camera` `- Faint smile forms`；`Background Sound:` 只写场景声（rain, café murmur），**不点配乐**。
- **Kling 3.0（五层标签；i2v 时砍成纯运动）**：t2v 用 `Scene:/Characters:/Action:/Camera:/Audio & Style:` 五行各司其职；i2v 则删光视觉重述只留动词流：`She lowers her cup, then turns toward the camera with a faint smile`——**必须带 endpoint**（动作有终点），防循环卡死。
- **Hailuo 02（克制 + 物理细节）**：删掉全部氛围形容词，只留物理：`ceramic cup contacts saucer with realistic weight（杯子带真实重量触碰托盘）, her head turns with natural neck tension（颈部肌肉自然转头）, rain streaks accelerate down the glass（雨痕沿玻璃加速流下）`。
- **Wan（四段标签）**：`Entity:` 参考图中的女子 / `Scene:` 雨夜咖啡馆、霓虹反光 / `Motion:` 放杯→转头→微笑 + slow push-in / `Sound:` rain intensifying, café ambience（要对白就在 Sound 段直接给台词，自动对口型）。

## 四、通用纪律（所有厂商都成立）

- **主体先行、永不用代词**：稳定特征（服装/发型/年龄）写在最前，全文用同一称呼指代。
- **动词具体化**：`rockets / slams / spins`（猛冲/撞上/旋转）替代 `moves / goes`；情绪转成物理动作（`her jaw tightens` 下颌绷紧，而非 "she feels sad"）。
- **光源具名**：`neon sign / candlelight / overcast sky`（霓虹灯/烛光/阴天）替代 `dramatic lighting`——真实光源才有稳定阴影。
- **动作给 endpoint**：`...then settles back into place`（随后归位）——没有终止状态的动作极易陷入循环。
- **i2v 只写运动**：图里已有的颜色/构图/陈设一律不重述，重述会触发重画（Kling / Seedance / Hunyuan / Wan 多家共通铁律）。
- **声音写具体内容并排除不要的**：写 `metallic clink of a coin hitting stone`（硬币落石的金属脆响）而非笼统 "sound effects"；原生音频家顺手排除（`no music, no audience sounds`）。

## 五、已知分歧与待查证（写该厂商 prompt 前查官方文档定版）

- **Veo 3.1 时长与画幅**：8s 单段 / 60s / 148s chained 三种说法并存；"原生仅 16:9"是旧版信息，竖屏是否已原生支持未确认。
- **Seedance 关键参数自相矛盾**：最高分辨率 720p（官方文档转写）vs 1080p（社区）；音频是否原生——多数信源记载原生 lip-sync/自动配乐，一家标"需后期"，按多数采信但需官方定夺。
- **Sora 2 现状**：web/app 停服说法仅单一信源；时长 25s/15s/20s 三说冲突；分辨率与画幅合法值在所有源材料中无记载。
- **Runway Gen-4 的 i2v 主流程**（参考图、keyframes、分辨率档、negative 机制）在源材料中全为空白，仅 Aleph 编辑写法清楚。
- **Hailuo 的 i2v 与音频**：subject reference 与官方导演模式运镜指令语法完全缺失；"无原生音频"的结论对新版本是否仍成立未验证。
- **Wan 可信信息极薄**：另一信源的 Wan 章节系照抄其 Veo 章节（残留"Google 的模型"字样），已整段剔除；时长/分辨率/首尾帧/negative 全部待查阿里官方。
- **Pika 基础参数不明**：单段时长档、分辨率、"音频部分支持"的具体含义、Pikaffects 官方完整清单均未讲清。
- **JSON 结构化 prompt 全行业无官方背书**：社区 JSON 实例（Veo 风字段解剖、Seedance 风 timeline+negative_prompt）有出片实证但无 A/B 对照；fps / seed / 运动强度等 API 级参数同样无信源——一律以官方 API 文档为准。
