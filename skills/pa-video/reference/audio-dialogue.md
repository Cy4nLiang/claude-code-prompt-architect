# 音频与对白写法指南

> 数据截至 2026-06，各家迭代快，**最终以官方最新文档为准**。
> 铁律：在 audio-native 模型（Veo / Sora / Kling / Seedance / Wan / HappyHorse）上，**不写音频 ≠ 没有音频，= 随机音频**——模型会自行幻觉出观众笑声、错误配乐或杂音。音频字段不允许留空：至少显式声明"无声"或"纯环境音"。

## 一、音频分层模型（写 prompt 时逐层过一遍）

| 层 | 写什么 | 英文措辞示例 |
|---|---|---|
| **ambience 环境底噪** | 持续、低存在感的"地点声"，奠定空间真实感；每镜必有 | `room tone, soft city hum, distant traffic`（房间底噪/城市嗡鸣/远处车流） |
| **SFX / foley 音效** | 与画面动作同步的具体声音（脚步/开门/碰撞）；写具体物件，禁写笼统的 "sound effects" | `footsteps on gravel, door slam, sizzling pan`（碎石脚步/摔门/煎锅滋滋声） |
| **music 配乐** | 风格 + 情绪走向 + 强弱变化；Sora 上禁写（见第二节） | `soft jazz piano, tension-building electronic score`（轻爵士钢琴/渐强电子配乐） |
| **dialogue 对白** | 逐字台词 + 说话人 + 情绪 tone（详见第三节） | `The man says: "..." his voice calm but firm`（台词+语气标注） |
| **silence 留白** | 刻意无声也是声音设计，必须显式声明，否则模型自己填 | `complete silence, no music, no dialogue, minimal ambient`（完全无声） |

结构化（JSON 式）prompt 的惯用块：`audio: {music, ambience, sound_effects}` + `dialogue: {spoken_lines: [{speaker, line, delivery}], subtitles: false}`；无对白时显式写 `"dialogue": none`，无配乐写 `"music": "no music"`。

## 二、无声与防幻觉

**典型事故**：场景里有人讲笑话但没写音频 → 模型幻觉出"现场观众笑声"。修法是**显式声明环境基调 + 排除句式**：

- ❌ `Character tells a joke.`
- ✅ `Character tells a joke. Audio: quiet office ambiance, no audience sounds.`

| 规则 | 措辞 |
|---|---|
| 环境音永远写具体声源（2-4 个），不写虚词 | ❌ `Audio: ambient sounds` → ✅ `Audio: roaring engines, gravel impacts, occasional driver shouts` |
| 排除句式按需追加 | `no audience sounds`（无观众声）/ `no laugh track`（无罐头笑声）/ `no background music`（无配乐）/ `no crowd noise`（无人群嘈杂） |
| 嘈杂场景反着用：把"应该吵"写出来 | `Audio: distant bands, noisy crowd, busy festival field ambience`——防模型配错环境 |
| **Sora 特例**：`Background Sound:` 只写场景内（diegetic）声音 | 严禁写 `epic orchestral score` 类后期配乐描述——Sora 会自动配乐，强加 score 会冲掉自然氛围 |
| 字幕也是常见幻觉副作用 | 对白用冒号语法 + `no subtitles, no on-screen text`（见第三节） |

## 三、对白规则

1. **台词逐字给出**。"两人讨论电影"这类即兴指令只适合 demo；可控产出必须写出每个字。
2. **说话人用外观特征点名**（多人场景）：`The woman in the red dress asks: "..." The man in glasses replies: "..."`——模型靠外观绑定声源。
3. **情绪 tone 必标**：`says with conviction` / `(laughing)` / `voice trembling`。注意 delivery 影响语速：低语、哽咽、犹豫显著拖慢，喊叫加快——留时长余量。
4. **冒号语法防字幕**（Veo 社区验证）⚠：写 `X says: "..."`（says 后带冒号）；无冒号易触发烧录字幕。再叠加 `no subtitles, no text overlays` 双保险。
5. **长度预算：8 秒 ≈ 12-15 个英文词**（一两句金句）⚠。太长 → 语速被压成念经；太短（单词 "Yes"）→ 沉默或胡言。Sora 口径：每 4 秒 clip 至多 2-3 行短句，每行 1-2 秒能说完。
6. **对白长度反推镜头时长**：先写台词、按自然语速估时（含第 3 条 delivery 修正），再定 clip 秒数——不要反过来先定秒数再塞词。
7. **lip-sync 与字幕约束句式**：中文短剧惯用「口型与字幕逐字完全一致，字幕即台词本身。每句台词控制在短句，清晰可读」；英文等价 `lip movements match the dialogue exactly, word for word`。
8. **发音修正**：专有名词用音标化拼写（Fofur → `foh-fur`）；中文生僻字用同音字替换 ⚠（仅缓解，不保证）。
9. **音色参考漂移**（Seedance 系）⚠：引用 `@音频1` 之外再用文字补音色描述（如 `低厚温润的中年男声`），且台词语气与参考音频风格保持一致。
10. **Kling 限制** ⚠：15s clip 的 lip-sync 前段（约前 2/3）更稳——对白放前段，结尾留给动作。

## 四、厂商音频语法映射

| 厂商 | 原生音频 | 对白写法 | 音频/环境音写法 | 备注 |
|---|---|---|---|---|
| **Veo 3 / 3.1** | ✅ 业界最强 | 正文冒号语法 `X says: "..."`；或独立 `Dialogue:` 行，多人时逐行 `角色 (tone): "line"` | 独立 `Audio:` 行，列多层声音（环境+音效+BGM 可并存） | 3.1 音效对齐更精、支持多人定向；防字幕靠冒号+排除句 |
| **Sora 2** | ✅ | 嵌在 Actions 列表里：`Character (tone): "line"`，每 4s ≤2-3 短行 | `Background Sound:` 只写 diegetic 场景内声音 | 严禁写后期配乐描述；模型自动配乐 |
| **Kling 3.0** | ✅ | 写进 `Audio & Style` 层：对话内容+语气；支持多人定向发声（A 说 X、B 说 Y） | 同层写环境音与配乐基调 | 中文最强，可直接中文写；lip-sync 前段更稳 ⚠ |
| **Seedance 2.0 / 即梦** | ✅⚠ 信源有分歧 | 台词写进时间码分镜块 + 「口型与字幕逐字完全一致」约束句 | `@音频1` 绑 BGM/音色参考；环境音文字描述（`带轻微环境音`） | lanshu 标"音频需单独处理"，多数 repo 实证原生 lip-sync/配乐——以官方为准 ⚠ |
| **Wan 2.x（通义万相）** | ✅ | `Sound:` 段给 `Voice: "..."` 自动对口型；Motion 段写 `lips synced to dialogue`、Entity 写 `facing the camera` | `Sound:` 段 voice / sfx / music 三元素可并存 | 数字人播报/口播场景最顺 |
| **HappyHorse（千问）** | ✅ | 台词放 `AUDIO:` 区块且**必须英文双引号**包裹，才触发 lip-sync | `AUDIO:` 区块置于 prompt 末尾，写具体音效与环境音 | 多语言 lip-sync；禁写笼统 `sound effects` |
| **Runway / Hailuo / Hunyuan / 开源系** | ❌ | 不写对白 | 不写音频；结构化 prompt 中显式置空并注明"需后期配音/配乐" | 配乐、口播、卡点全部后期完成 |
| **Pika** | ⚠ 标称部分支持但无写法实证 | 同上按无原生处理 | 同上 | 用前查官方文档 |

## 五、分场景环境音速查

| 场景 | 英文 ambience 措辞 |
|---|---|
| 办公室 | `keyboard typing, air-conditioning hum, muffled phone calls, paper rustling`（键盘/空调/电话/纸张） |
| 居家室内 | `ambient room tone, clock ticking, natural indoor acoustics`（房间底噪/钟表/室内声学） |
| 城市街道 | `distant traffic hum, occasional car horns, footsteps on pavement, muffled conversations`（车流/喇叭/脚步/人语） |
| 森林自然 | `gentle wind through trees, birdsong, rustling leaves, distant water flowing`（风过树梢/鸟鸣/叶响/远处流水） |
| 海边 | `waves crashing, seagulls calling, steady coastal wind`（浪涌/海鸥/海风） |
| 厨房烹饪 | `sizzling pan, knife chopping vegetables, boiling water, utensils clinking`（煎炒/切菜/沸水/餐具碰撞） |
| 夜晚户外 | `distant crickets, low wind, faint city buzz`（虫鸣/低风/远处城市余音） |
| 人群活动 | `noisy crowd, distant live music, festival field ambience`（嘈杂人群/远处乐声/露天活动场） |

用法：取 2-4 个声源组成 `Audio:` 行——越具体，越不容易幻觉。

## 六、音画对齐两条纪律

1. **音频时长 = 视频时长，严格对齐**。逐镜头流水线里，每个 clip 的音频（对白+环境音）必须恰好铺满该 clip 时长：对白先行、秒数跟随（第三节第 6 条）；跨镜头的环境底噪要写成连续不中断，否则剪辑点会"跳声"。
2. **音乐卡点只在能喂节奏参考素材的厂商才稳**。Seedance 的 `@音频1`（上传 BGM + prompt 写"卡点变装/背景音效与 @音频1 同步"）是被反复验证的卡点路径；其他厂商纯文字描述"卡点剪辑"属于碰运气 ⚠——没有参考素材入口的厂商，一律生成后在剪辑软件里对拍。
