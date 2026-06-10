# 运镜与镜头语言词表（Camera Language）

> 数据截至 2026-06。术语为通用电影语言，主流视频模型均能直接理解标准术语；各家语法差异（@引用、机位短语、timecode）查同目录 vendor-matrix.md。
> 运镜是视频版的"构图"：图像 prompt 写画面怎么摆，视频 prompt 还要写镜头怎么动。
> 词表条目格式 = 名称（中英）/ 作用 / 何时用 / 示例措辞。**措辞示例用英文**（多数模型英文 prompt 更稳），括注中文释义。

## 〇、硬规则（先记这 4 条再查表）

| # | 规则 | 说明 |
|---|---|---|
| 1 | **一个镜头只指定 1 种运镜** | 同段同时要求推拉摇移会明显增加画面不稳定；"static camera" 和 "orbit" 这类互斥指令绝不能出现在同一段 |
| 2 | **运镜三件套：运镜动词 + 速度词 + 景别** ⚠ | 只写 "push in" 模型会自定节奏和取景；写 "slow push-in from medium shot to close-up" 才可控（综合推演规则） |
| 3 | **同类景别不连用超过 3 次** ⚠ | 连续多个特写会让观众丢失空间感，穿插中景/全景重建定位（单一信源：penshot 分镜规则） |
| 4 | **景别衔接有语义** ⚠ | wide→medium = 空间定位；medium→close-up = 情感聚焦；close-up→close-up = 视线匹配。转场：场景变化用淡入淡出、情绪高潮直切、时间跳跃用叠化（单一信源：penshot） |

## 一、景别 7 档（EWS → ECU）

| 英文 | 中文 | 作用 | 示例措辞 |
|---|---|---|---|
| Extreme Wide Shot (EWS) | 大远景 | 地理交代、孤立感、宏大尺度 | "extreme wide shot of a lone figure crossing the vast desert"（强调渺小与孤立） |
| Wide Shot (WS) / Establishing | 远景 / 建立镜头 | 环境交代、开场定位 | "wide shot establishing the dim library interior"（先让观众知道在哪） |
| Full Shot (FS) | 全景（全身） | 全身肢体动作、人与环境的关系 | "full shot showing the dancer's entire body in motion" |
| Medium Shot (MS) | 中景 | 腰部以上；对话与叙事的主力景别 | "medium shot, waist up, of the chef explaining the recipe" |
| Medium Close-Up (MCU) | 中近景 | 头肩；意图、手部动作、工具细节 | "medium close-up, head and shoulders, as she hesitates" |
| Close-Up (CU) | 特写 | 面部充满画面；情绪与关键细节 | "close-up of her eyes widening in realization" |
| Extreme Close-Up (ECU) | 大特写 | 眼睛、纹理、微表情、关键道具 | "extreme close-up of the key turning in the lock" |

另有三种"关系机位"常与景别并列使用：over-the-shoulder（过肩，对话空间）、POV（主观视角，代入感）、two shot（双人同框，关系展示）。

## 二、基础运镜 7 种

| 中文 | 英文 | 何时用 | 示例措辞 |
|---|---|---|---|
| 推镜 | Push in / Dolly in | 聚焦强调、建立悬念、拉近心理距离 | "slow push-in toward the artifact as it begins to glow" |
| 拉镜 | Pull back / Dolly out | 揭示环境、收尾抽离、反转揭晓 | "camera slowly pulls back to reveal the empty street" |
| 摇镜 | Pan（横摇）/ Tilt（纵摇） | 原地转动扫过空间或揭示高度 | "slow pan left across the skyline" / "tilt up from the worn boots to the determined face" |
| 移镜 | Truck / Lateral move | 横向平移，靠前后景视差展示空间层次 | "camera trucks right past the market stalls, subject stays frame-left" |
| 跟镜 | Tracking / Follow shot | 跟随主体行进，保持空间关系 | "smooth tracking shot following the runner from the side" |
| 升降 | Crane up / Crane down | 垂直升降切换尺度：贴近 ↔ 上帝视角 | "camera cranes up, revealing the entire city skyline at night" |
| 环绕 | Orbit / Arc | 围绕主体多角度展示：产品、角色亮相 | "camera orbits slowly around the levitating crystal" |

## 三、高级运镜 10 种

| 中文 | 英文 | 何时用 | 示例措辞 |
|---|---|---|---|
| 希区柯克变焦 | Dolly zoom (Hitchcock zoom) | 推轨+反向变焦的空间挤压眩晕；恐惧/顿悟瞬间 | "dolly zoom on his face at the moment of fear, background warping" |
| 甩镜 | Whip pan | 极快横摇带运动模糊；高能转场、场景无缝衔接 | "whip pan from subject A to subject B, motion blur" |
| 焦点转移 | Rack focus | 前后景之间转移合焦点，引导注意力、双主体叙事 | "rack focus from the foreground flower to her face in the background" |
| 一镜到底 | One-take / Oner | 无剪辑连续长镜头；沉浸式跟随、空间连贯 | "one continuous take following the runner up the stairs onto the rooftop, no cuts" |
| 手持 | Handheld | 轻微抖动制造纪实感、紧迫感、临场混乱 | "handheld camera with subtle shake, documentary style" |
| 稳定器 | Steadicam / Gimbal | 顺滑流动的跟拍；商业广告、优雅长镜 | "gimbal-smooth follow shot, liquid camera movement" |
| 第一人称 | FPV / First-person POV | 角色主观视角；vlog、沉浸体验 | "first-person POV shot from the character's eyes" |
| 航拍 | Aerial / Drone shot | 高空俯瞰；开场建立、自然风光、宏观地理 | "aerial drone shot flying low over the coastline" |
| 荷兰角 | Dutch angle | 地平线倾斜制造不安；心理失衡、危险暗示 | "Dutch angle, horizon tilted, sense of unease" |
| 固定镜头 | Static / Locked-off | 零运动旁观，让动作穿过画面；对话、产品悬浮展示 | "static camera, locked-off shot, the subject moves through the frame" |

## 四、节奏与速度词

| 英文 | 中文 | 修饰效果 |
|---|---|---|
| slowly / slow | 缓慢 | 悬念、抒情、庄重；运镜速度的默认安全选项（slow push-in 建立悬念） |
| gently / softly | 轻柔 | 亲密、温情；适合小幅动作（leaves gently swaying） |
| gradually | 逐渐 | 强调渐变过程（lighting gradually warming as the mood shifts） |
| smoothly / steadily | 平稳流畅 | 专业感、广告质感（smooth tracking shot） |
| rapidly / quickly | 快速 | 能量、紧迫；⚠ 快速大幅动作易变形，慎用于人物肢体 |
| subtly / slightly | 微微 | 克制的微表情、微动作（slightly tilts her head） |

- 速度词同时服务运镜与动作：动作描述要"肢体细化 + 程度量化"（slowly raises her hand / 缓慢抬手），优先低缓连续的小动作，规避狂奔、大跳、剧烈翻滚等高爆发动作。
- **用程度词，不用数字**：角速度、英尺/秒、毫米级抖动等数值模型不会字面执行，只会污染 prompt——写 "slow" 而非 "2 feet per second"。

## 五、灯光速查（14 种）

| 英文 | 中文 | 氛围效果 |
|---|---|---|
| golden hour | 黄金时刻 | 温暖、怀旧、浪漫、旅行感 |
| blue hour | 蓝调时刻 | 冷峻、科幻、孤独 |
| overcast / soft diffused light | 阴天漫射光 | 真实、平静、均匀无硬影 |
| hard key light | 硬光 | 戏剧性、锐利阴影、危险 |
| soft window light | 柔和窗光 | 亲密、平静、生活感 |
| low-key lighting / chiaroscuro | 低调光 / 明暗对照 | 悬疑、黑色电影、心理纵深（大面积阴影） |
| high-key lighting | 高调光 | 明亮、乐观、安全、美妆广告 |
| silhouette / backlight | 剪影 / 背光 | 神秘、力量感、轮廓叙事 |
| rim light | 轮廓光 | 主体与背景分离、勾边强调 |
| neon spill | 霓虹溢光 | 赛博朋克、夜生活、人工色彩 |
| volumetric light / god rays | 体积光 / 丁达尔光束 | 史诗感、神圣感、空气中可见光柱 |
| practical lights | 实景光源 | 画面内可见的灯具发光，真实自然 |
| warm tungsten | 钨丝暖光 | 复古室内、家的感觉 |
| cool fluorescent | 荧光冷白 | 办公/医院/地铁，人工、不安、临床感 |

## 六、调色与风格速查

| 调色（grading） | 中文 / 效果 | | 风格（style） | 英文关键词 |
|---|---|---|---|---|
| teal and orange | 青橙调：好莱坞动作片标配 | | 电影感 | cinematic, film grain, anamorphic, 35mm |
| warm grading | 暖调：温情、怀旧 | | 纪录片 | documentary, handheld, naturalistic, vérité |
| cool grading | 冷调：疏离、科幻、悲伤 | | 黑色电影 | film noir, high contrast B&W, deep shadows |
| desaturated / muted palette | 低饱和：克制、文艺、阴郁 | | 赛博朋克 | cyberpunk, neon-lit, holographic |
| high saturation | 高饱和：活力、广告、热带 | | 复古胶片 | vintage, 8mm, VHS, retro grain |
| black and white / monochrome | 黑白：永恒感、心理化 | | 动漫 | anime aesthetic, cel-shaded, Ghibli-inspired |

## 七、机位与厂商特殊语法注记

- **Veo 机位短语** ⚠：社区经验称在机位描述后追加 "(thats where the camera is)" 能显著提升 Veo 的机位遵循，如 "handheld camera at chest height (thats where the camera is) tracking the subject"。单一信源（Veo-3-Prompting-Guide 社区总结），仅对 Veo 有效，其他模型不要照搬。
- **Seedance（即梦）运镜引用**：复杂运镜与其用文字描述，不如直接引用参考视频："reference @Video1's camera movement"——比任何词表都更精确。@引用语法与各家 negative/timecode/音频差异详见 vendor-matrix.md。
