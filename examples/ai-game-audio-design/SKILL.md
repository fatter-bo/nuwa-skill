---
name: ai-game-audio-design
description: |
  AI游戏音频设计专家。用AI工具完成游戏音频全流程——BGM/音效/人声/引擎适配。
  面向独立开发者，零音乐基础也能产出商用级游戏音频。
  内置棋牌类/合成类/通用UI三大音效提示词模板库，覆盖Suno BGM生成、ElevenLabs音效与人声生成、
  FFmpeg后处理、Unity/Cocos Creator引擎集成的端到端Pipeline。
  当用户说「音效」「BGM」「游戏音乐」「背景音乐」「音频设计」「Suno」「游戏声音」「出牌音效」时使用。
  也适用于「帮我生成一首游戏BGM」「做一套棋牌音效」「游戏里需要什么音频」「怎么用AI做游戏配乐」等场景。
---

# AI游戏音频设计专家

> 零音乐基础，用AI工具30分钟产出一首商用级游戏BGM，1小时交付50个专业音效。

## 使用说明

**这个Skill不扮演角色，是一个系统化工具。** 用户提出游戏音频相关需求时，按照下面的工作流自动执行。

---

## 执行工作流（Agentic Protocol）

**用户说"帮我做游戏音频"时，按以下流程执行：**

```
Phase 0: 需求澄清
  → 游戏类型（棋牌/合成/消除/RPG/其他）
  → 需要哪些音频（BGM/音效/人声/环境音）
  → 目标平台（微信小游戏/H5/Unity/Cocos Creator/原生App）
  → 风格偏好（轻快/古风/爵士/电子/8-bit等）
  → 预算约束（决定工具选型：Suno Pro $10 vs 免费方案）

Phase 1: 音频资产清单
  → 根据游戏类型列出所有需要的音频资产
  → 标注每个资产的类型/场景/情绪/时长/优先级
  → 确认清单与用户对齐

Phase 2: BGM生成
  → 编写Suno提示词（使用"BGM提示词模板"）
  → 每个场景生成5-8首候选
  → 用户筛选确认
  → 循环测试（连续播放3遍检查循环点）

Phase 3: 音效生成
  → 从"音效提示词模板库"选取对应模板
  → ElevenLabs批量生成（每个音效4个变体）
  → 人工筛选最佳变体

Phase 4: 人声生成（如需要）
  → 整理台词清单
  → ElevenLabs选择/克隆声线
  → API批量生成 + 人工筛选

Phase 5: 统一后处理
  → FFmpeg批量处理：去静音 + 响度标准化 + 格式转换
  → BGM: -16 LUFS / OGG
  → 音效: -14 LUFS / OGG
  → 人声: -16 LUFS / OGG

Phase 6: 引擎集成
  → 按命名规范组织文件目录
  → 配置引擎音频参数（压缩策略/加载方式）
  → 混音平衡调试（BGM/音效/人声音量层级）

Phase 7: QA验收
  → 执行"音频QA检查表"（10项）
  → 真机测试
  → 迭代修正
```

---

## 核心方法论框架

### 框架1: 游戏音频六层模型

游戏音频的功能层级划分，每层有不同的混音优先级和技术处理方式：

| 层级 | 类型 | 功能 | 混音优先级 | AI工具 |
|------|------|------|-----------|--------|
| L1 | BGM（背景音乐） | 营造氛围、驱动情绪 | 基底层，音量最低 | Suno |
| L2 | 环境音（Ambience） | 构建空间感和沉浸感 | 次底层 | ElevenLabs SFX |
| L3 | UI音效 | 确认操作反馈 | 中层 | ElevenLabs SFX |
| L4 | 操作反馈音效 | 即时行为确认（出牌/合成） | 高优先级 | ElevenLabs SFX |
| L5 | 演出音效 | 强化关键时刻（胜利/大招） | 最高优先级 | ElevenLabs SFX |
| L6 | 人声（Voice） | 叙事和指引 | 动态（播放时ducking BGM） | ElevenLabs TTS |

**音量分配基准**（基于总输出-16 LUFS）：
- BGM: -18 to -24 dB
- 环境音: -20 to -26 dB
- UI音效: -12 to -16 dB
- 操作反馈: -8 to -12 dB
- 演出音效: -6 to -10 dB
- 人声: -10 to -14 dB（播放时BGM ducking -6 to -12dB）

来源：[Game Developer - Sound Design Primer](https://www.gamedeveloper.com/)、[GDC Vault - Game Loudness Standards](https://gdcvault.com/play/1017781/Game-Loudness-Industry)、[Designing Sound - Loudness In Game Audio](https://designingsound.org/2013/02/28/loudness-in-game-audio/)

---

### 框架2: AI工具选型决策树

根据需求类型快速选择最佳AI工具：

```
需要什么类型的音频？
├── BGM（背景音乐）
│   ├── 需要人声/歌词？ → Suno（最强歌词能力）
│   ├── 纯器乐BGM → Suno（纯器乐模式）
│   ├── 零版权风险 → MusicGen（MIT开源）
│   └── 预算为零 → MusicGen本地部署
│
├── 音效（SFX）
│   ├── 物理音效（牌声/筹码/碰撞） → ElevenLabs Sound Effects
│   ├── 抽象/UI/魔法音效 → ElevenLabs Sound Effects
│   ├── 环境音/氛围音 → ElevenLabs SFX 或 AudioGen（开源）
│   └── 预算为零 → Freesound.org + AudioGen
│
├── 人声（Voice）
│   ├── 高质量角色配音 → ElevenLabs v3 + 声音克隆
│   ├── 系统提示/短句播报 → ElevenLabs Flash v2.5
│   ├── 本地部署/完全控制 → Coqui XTTS v2.5 / ChatTTS
│   └── 人群/环境人声 → Bark
│
└── 不确定？ → ElevenLabs生态（SFX + TTS一站式）
```

**工具能力速查表**：

| 工具 | 类型 | 商用授权 | 定价 | 核心优势 |
|------|------|---------|------|---------|
| Suno v4.5+ | BGM生成 | Pro/Premier可商用 | $10-30/月 | 全能型BGM，支持循环模式 |
| ElevenLabs SFX | 音效生成 | 付费免版税 | $5起/月 | 48kHz专业级，每次4变体 |
| ElevenLabs TTS | 人声生成 | 付费免版税 | 含在订阅中 | 70+语言，声音克隆 |
| MusicGen (Meta) | BGM生成 | MIT许可证 | 免费 | 完全开源，零版权风险 |
| AudioGen (Meta) | 音效生成 | MIT许可证 | 免费 | 开源，环境音擅长 |
| Coqui XTTS v2.5 | 人声生成 | MPL-2.0 | 免费 | 开源，零样本克隆 |
| ChatTTS | 人声生成 | AGPL-3.0 | 免费 | 中文语音质量极高 |

来源：[Suno Hub](https://suno.com/)、[ElevenLabs](https://elevenlabs.io/)、[Meta AudioCraft](https://github.com/facebookresearch/audiocraft)、[AIMagicx - Suno vs Udio vs ElevenLabs 2026](https://www.aimagicx.com/blog/suno-vs-udio-vs-elevenlabs-music-comparison-2026)

---

### 框架3: BGM提示词工程模板

Suno提示词结构：`[Genre/Style] + [Mood] + [Instruments] + [BPM] + [Additional]`

**关键原则**：描述而非命令；指定子风格和年代；用Metatag标注结构（[Intro]/[Verse]/[Chorus]等）；明确BPM和Key。

| 游戏场景 | Suno提示词模板 |
|---------|---------------|
| 休闲游戏主界面 | `Style: cheerful casual game soundtrack, light and playful` `Instruments: piano, xylophone, ukulele, light synth pads, gentle bells` `Tempo: 110 BPM, Key of C major` `Mood: happy, carefree, warm` `Additional: seamless loop, no fade in/out, no vocals` |
| 棋牌游戏大厅 | `Style: smooth jazz lounge, sophisticated card game atmosphere` `Instruments: brushed drums, upright bass, muted trumpet, piano` `Tempo: 85 BPM` `Mood: relaxed, classy, understated` `Additional: seamless loop, minimal melodic movement, background music` |
| 合成游戏关卡 | `Style: whimsical puzzle game soundtrack, magical and curious` `Instruments: marimba, pizzicato strings, celesta, soft synth` `Tempo: 100 BPM` `Mood: curious, delightful, slightly mysterious` `Additional: seamless loop, evolving subtle variations` |
| 对局紧张时刻 | `Style: tense strategic game music, building suspense` `Instruments: low strings, subtle percussion, dark synth pad` `Tempo: 90 BPM` `Mood: focused, tense, anticipation` `Additional: seamless loop, dynamic intensity build` |
| 胜利/结算 | `Style: triumphant celebration fanfare, game victory` `Instruments: brass, orchestral strings, snare roll, cymbal crash` `Tempo: 130 BPM` `Mood: victorious, exciting, celebratory` `Additional: 10-15 seconds, dramatic ending, no loop needed` |

**风格一致性技巧**：多首BGM共用固定的Style+Instruments基底，只改Mood/Tempo/Scene参数。统一Key（C/F/G大调之间切换）。全部BGM通过相同后处理链（EQ/压缩/响度标准化）。

来源：[Suno Hub - Best Ways To Create Music With AI](https://suno.com/)、[MusicSmith.ai - Suno AI Prompt Guide](https://musicsmith.ai/)、[Song AI Farm - Suno Prompt Engineering Guide 2026](https://songaifarm.com/)

---

### 框架4: 音效提示词模板库

ElevenLabs Sound Effects提示词规则：描述声源+环境+声学特征；指定材质/尺寸/距离/时间特征。

#### 棋牌类音效

| 音效 | 提示词 | 时长 |
|------|--------|------|
| 普通出牌 | "single playing card being placed firmly on a felt table, slight paper snap, close mic" | 200-400ms |
| 快速出牌 | "quick flick of a playing card landing on table, sharp and snappy, close-up" | 100-200ms |
| 洗牌 | "deck of playing cards being riffle shuffled, crisp paper sounds, casino table" | 1-2s |
| 发牌 | "cards being dealt one by one rapidly, sliding across felt surface, slight whoosh" | 0.5-1s |
| 筹码放置 | "single ceramic poker chip placed on felt table, solid clack sound" | 200-300ms |
| 筹码堆碰撞 | "stack of poker chips being pushed together, multiple ceramic clicks and clatters" | 0.5-1s |
| 大量筹码推注 | "large pile of poker chips sliding across felt table, cascade of ceramic clicks, casino ambience" | 1-2s |
| 炸弹牌型 | "dramatic explosive impact with reverb, powerful bass thump, screen-shaking energy, game special move" | 1-2s |
| 王炸牌型 | "epic explosion with thunder crack, massive reverberant boom, energy burst with sparkle trail" | 2-3s |
| 计时器滴答 | "clock ticking steadily, clean metronome tick, isolated, no reverb" | 1s循环 |
| 计时器警告 | "urgent fast ticking clock, increasing tempo, warning beep, tension building" | 2s |
| 胜利 | "triumphant short fanfare, brass and strings, victorious and celebratory, game win" | 2-3s |
| 失败 | "gentle descending tones, soft disappointment sound, not harsh, game over casual" | 1-2s |

#### 合成类（Merge Game）音效

| 音效 | 提示词 | 时长 |
|------|--------|------|
| 拖拽物品 | "small object being picked up and held, soft pop with slight whoosh, game item drag" | 100-200ms |
| 合成成功 | "magical merge sound, two items combining with sparkling chime, ascending tones, satisfying pop" | 300-500ms |
| 高级合成 | "powerful magical transformation, crystal resonance with golden shimmer, upgrade achievement" | 500ms-1s |
| 升级 | "level up chime, ascending musical scale with sparkle, achievement unlocked feel" | 1-1.5s |
| 收集物品 | "soft coin collect sound, gentle metallic ding, casual game pickup" | 100-200ms |
| 解锁新内容 | "treasure chest opening with magical reveal, wonder and discovery, bright tones" | 1-2s |
| 星星飘散 | "twinkling stars scattering, magical sparkle dust falling, high-pitched fairy dust" | 1-1.5s |
| 消除爆破 | "satisfying pop burst with particle scatter, bubble pop chain reaction, casual game clear" | 300-500ms |
| 连击 | "rapid ascending combo hits, each higher than last, building excitement, game combo" | 1-2s |

#### 通用UI音效

| 音效 | 提示词 | 时长 |
|------|--------|------|
| 按钮点击 | "clean UI button click, soft digital tap, modern interface sound, subtle and satisfying" | 50-100ms |
| 弹窗弹出 | "popup window appearing, soft whoosh with a gentle boing, playful UI sound" | 200-400ms |
| 弹窗关闭 | "popup closing, reverse whoosh, quick and clean dismissal sound" | 100-200ms |
| 页面切换 | "smooth page swipe transition, soft whoosh left to right, clean UI navigation" | 200-300ms |
| 奖励领取 | "reward claim sound, coin shower with triumphant ding, satisfying collection jingle" | 0.5-1s |
| 错误提示 | "gentle error notification, soft low-pitched buzz, not harsh, friendly warning" | 200-300ms |
| 金币掉落 | "gold coins dropping and bouncing on surface, metallic jingling cascade, game reward" | 1-2s |
| 通知消息 | "gentle notification chime, pleasant two-tone alert, mobile game notification" | 200-400ms |
| 倒计时结束 | "countdown complete buzzer, definitive end signal, game time up" | 500ms-1s |

来源：[ElevenLabs Sound Effects](https://elevenlabs.io/)、[Scenario - ElevenLabs SFX v2评测](https://www.scenario.com/)

---

### 框架5: 音频格式与引擎适配规范

#### 格式选型

| 用途 | 格式 | 压缩 | 采样率 | 声道 | 说明 |
|------|------|------|--------|------|------|
| BGM | OGG Vorbis | Quality 100% | 44.1kHz | Stereo | 文件小、循环完美 |
| 短音效（<1s） | WAV/ADPCM | 无/轻压缩 | 44.1kHz | Mono | 低延迟 |
| 长音效（>2s） | OGG Vorbis | Quality 70% | 44.1kHz | Mono | 平衡大小和质量 |
| 人声 | OGG Vorbis | Quality 70% | 44.1kHz | Mono | 语音不需要立体声 |

**绝对不用MP3做循环BGM**（编码器首尾静音填充导致循环缝隙）。

#### Unity引擎配置

| 音频类型 | Load Type | Compression Format | 说明 |
|---------|-----------|-------------------|------|
| 短音效（<200KB） | Decompress On Load | ADPCM | 低延迟，常驻内存 |
| 中等音效 | Compressed In Memory | Vorbis 70% | 平衡 |
| BGM/长音频 | Streaming | Vorbis 100% | 不占内存 |

音效勾选`Force To Mono`（省50%内存）。BGM保持Stereo。

#### Cocos Creator注意事项

- Web平台：音频必须等待用户首次交互后才能播放（Chrome策略）
- 微信小游戏：并发音频数建议不超过10个
- 使用`playOneShot()`播放短音效，`play()`播放BGM

#### 响度标准

| 平台 | 目标响度 | 峰值上限 |
|------|---------|---------|
| 移动端游戏 | -16 LUFS | -1 dBTP |
| 主机游戏 | -24 LUFS | -1 dBTP |
| PC游戏 | -23 LUFS | -1 dBTP |

来源：[Unity Manual - Audio Compression](https://docs.unity3d.com/Manual/AudioFiles-compression.html)、[Cocos Creator 3.8 Manual - AudioSource](https://docs.cocos.com/creator/3.8/manual/en/audio-system/audiosource.html)、[GANG Loudness Recommendations](https://gdcvault.com/play/1017781/Game-Loudness-Industry)

---

### 框架6: 后处理标准化Pipeline

所有AI生成音频都必须经过统一后处理，确保专业一致性：

```
AI生成原始音频
    ↓
Step 1: 裁剪静音（FFmpeg silenceremove）
    ↓
Step 2: 响度标准化（FFmpeg loudnorm → -14/-16 LUFS）
    ↓
Step 3: 淡入淡出（音效: 5ms fade-in；BGM: 按需）
    ↓
Step 4: 格式转换（WAV → OGG Vorbis）
    ↓
Step 5: 命名规范化（{类型}_{场景}_{描述}_{变体号}.ogg）
    ↓
Step 6: QA验证（循环测试 / 响度一致性 / 爆音检测）
```

**一站式后处理命令**：

```bash
# BGM: 标准化到-16 LUFS + 转OGG
ffmpeg -i input.wav -af "loudnorm=I=-16:LRA=11:TP=-1" -c:a libvorbis -q:a 7 output.ogg

# 音效: 去静音 + 标准化到-14 LUFS + 淡入淡出 + 转OGG
ffmpeg -i input.wav \
  -af "silenceremove=start_periods=1:start_threshold=-50dB:stop_periods=1:stop_threshold=-50dB,loudnorm=I=-14:LRA=11:TP=-1,afade=t=in:d=0.005,afade=t=out:st=-0.01:d=0.01" \
  -c:a libvorbis -q:a 6 output.ogg

# 人声: 去静音 + 标准化到-16 LUFS + 转OGG
ffmpeg -i input.wav \
  -af "silenceremove=start_periods=1:start_threshold=-45dB:stop_periods=1:stop_threshold=-45dB,loudnorm=I=-16:LRA=11:TP=-1" \
  -c:a libvorbis -q:a 6 output.ogg
```

来源：[FFmpeg官方文档](https://ffmpeg.org/documentation.html)、[DEV Community - Audio Production for Non-Musicians](https://dev.to/)

---

### 框架7: 文件组织与命名规范

**命名规则**：`{类型}_{场景}_{描述}_{变体号}.{格式}`

```
audio/
├── bgm/                    # 背景音乐
│   ├── bgm_lobby_jazz_lounge.ogg
│   ├── bgm_battle_tense_strategy.ogg
│   └── bgm_result_victory_fanfare.ogg
├── sfx/                    # 音效
│   ├── card/               # 牌类操作
│   │   ├── sfx_card_play_normal_v1.ogg
│   │   ├── sfx_card_play_normal_v2.ogg
│   │   └── sfx_card_play_bomb_v1.ogg
│   ├── ui/                 # UI通用
│   │   ├── sfx_ui_button_click.ogg
│   │   └── sfx_ui_popup_open.ogg
│   ├── merge/              # 合成类
│   │   ├── sfx_merge_success_v1.ogg
│   │   └── sfx_merge_levelup.ogg
│   └── special/            # 特殊演出
│       └── sfx_special_win_fanfare.ogg
├── vo/                     # 人声
│   ├── system/
│   │   ├── vo_system_call_landlord_v1.ogg
│   │   └── vo_system_bomb_v1.ogg
│   └── char01/
│       └── vo_char01_greeting.ogg
└── amb/                    # 环境音
    └── amb_casino_crowd.ogg
```

来源：FMOD/Wwise项目组织最佳实践、行业通用规范

---

## 决策启发式

1. **先列清单再动手**：开始生成前，先把所有音频资产列成表格（类型/场景/情绪/时长/优先级）。这张表就是整个项目的作战地图，防止遗漏和返工。

2. **BGM宁简勿繁**：休闲游戏BGM的第一原则是"不干扰"。旋律太强、节奏太快都会在反复循环后让玩家烦躁。BPM 80-110，以和弦铺底为主，少用记忆点太强的旋律。

3. **同一操作做3-5个变体**：出牌、点击等高频操作只有1个音效会很快产生疲劳。为每个高频操作准备3-5个变体随机播放，成本增加很小但体验提升巨大。

4. **Suno提示词用"描述"不用"命令"**：写"a gentle piano ballad"而不是"create a piano ballad"。加上"seamless loop, no fade in/out, no vocals"三件套确保生成可直接循环使用的BGM。

5. **ElevenLabs一站式覆盖音效+人声**：$5/月的Starter计划同时支持Sound Effects和TTS，对独立开发者来说是性价比最高的音频AI方案。不需要为音效和人声分别订阅不同工具。

6. **MP3永远不要用于循环BGM**：MP3编码器会在首尾添加静音填充，导致循环时出现可察觉的缝隙。BGM一律用OGG Vorbis。

7. **后处理不能省**：AI生成的原始音频在响度、格式、首尾静音上都不统一。即使只有10个音效，也要跑一遍FFmpeg后处理脚本。这是"听起来像游戏"和"听起来像Demo"的分界线。

8. **移动端响度打到-16 LUFS**：移动设备在嘈杂环境使用，-24 LUFS的主机标准太安静。休闲手游统一到-16 LUFS是安全选择。

9. **Web/小游戏平台必须处理自动播放限制**：Chrome和微信小游戏都禁止音频自动播放。必须在用户首次交互（点击/触摸）后才启动音频。在游戏入口加一个"点击开始"按钮是最简单的解决方案。

10. **$15搞定一款休闲游戏的全部音频**：Suno Pro ($10) + ElevenLabs Starter ($5) = 3首BGM + 50个音效 + 20句人声。传统外包同等内容需要$1,000-4,000。AI方案节省98%直接成本，代价是学习和筛选时间。

---

## 工具与资源推荐

### 核心工具链

| 环节 | 工具 | 用途 | 链接 |
|------|------|------|------|
| BGM生成 | Suno | AI作曲，支持循环模式和歌词 | [suno.com](https://suno.com/) |
| 音效生成 | ElevenLabs Sound Effects | 文本描述生成48kHz专业音效 | [elevenlabs.io](https://elevenlabs.io/) |
| 人声生成 | ElevenLabs TTS | 70+语言TTS，支持声音克隆 | [elevenlabs.io](https://elevenlabs.io/) |
| BGM生成（开源） | MusicGen (Meta) | MIT许可，零版权风险 | [github.com/facebookresearch/audiocraft](https://github.com/facebookresearch/audiocraft) |
| 音效生成（开源） | AudioGen (Meta) | MIT许可，环境音擅长 | [github.com/facebookresearch/audiocraft](https://github.com/facebookresearch/audiocraft) |
| 人声生成（开源） | Coqui XTTS v2.5 | 零样本声音克隆，17语言 | [github.com/coqui-ai/TTS](https://github.com/coqui-ai/TTS) |
| 中文人声（开源） | ChatTTS | 中文语音质量极高 | [github.com/2noise/ChatTTS](https://github.com/2noise/ChatTTS) |
| 后处理 | FFmpeg | 音频裁剪/标准化/格式转换 | [ffmpeg.org](https://ffmpeg.org/) |
| 音频编辑 | Audacity | 波形编辑/循环点调整/宏批处理 | [audacityteam.org](https://www.audacityteam.org/) |
| 音频中间件 | FMOD | 自适应音乐/混音管理（独立开发者免费） | [fmod.com](https://www.fmod.com/) |
| 免费素材库 | Freesound | CC协议音效素材补充 | [freesound.org](https://freesound.org/) |

### 成本估算（休闲棋牌游戏典型配置）

| 项目 | 工具 | 内容 | 月费 |
|------|------|------|------|
| BGM | Suno Pro | 3首BGM + 试听废弃约20首 | $10 |
| 音效+人声 | ElevenLabs Starter | 50音效 + 20句人声 | $5 |
| 后处理 | FFmpeg + Audacity | 全部后处理 | 免费 |
| **合计** | | | **$15/月** |

---

## 诚实边界

此工具的局限：

1. **AI生成音频不等于专业音频设计**：AI工具能快速产出"够用"的游戏音频，但与专业作曲家/音效设计师的作品在原创性、细腻度和情感表达上仍有差距。对音频要求极高的项目（如剧情重度RPG）仍建议与专业音频设计师合作。

2. **版权风险未完全消除**：Suno Pro计划允许商用，但AI生成音乐的版权归属在法律上仍有争议（UMG/Sony/Warner vs Suno诉讼进行中）。美国版权局要求"人类作者身份"。MusicGen（MIT）是零风险选项但质量稍逊。

3. **AI音效的"AI味"**：AI生成的音效有时过于平滑、缺乏真实感。通过叠加轻微噪声、微调pitch可缓解，但某些高度写实的音效（如特定乐器演奏）仍不如真实录音。

4. **人声表演力有限**：AI TTS在"念台词"方面已很出色，但在需要极端情感表演（大哭/嘶吼/特殊口音）的场景，效果不如真人配音演员。

5. **工具快速迭代**：Suno/ElevenLabs等工具更新频繁，本文档中的具体功能、定价和提示词技巧可能在数月后需要更新。建议以各工具官方文档为准。

6. **不覆盖音乐理论深度创作**：本Skill定位是"零基础快速产出"，不涉及和声学、编曲、混音等专业音乐制作知识。如果用户需要深度定制音乐风格，建议学习基础乐理或咨询专业人士。

7. **引擎适配以Unity和Cocos Creator为主**：Unreal Engine、Godot等引擎的音频系统有差异，本Skill未深入覆盖。如使用其他引擎请参考对应官方文档。

---

## 附录：调研来源

调研过程详见 `references/research/` 目录（6个文件）：

1. `01-game-audio-theory.md` — 游戏音频理论（功能分类/心理学/棋牌设计重点）
2. `02-ai-music-generation.md` — AI音乐生成（Suno/Udio/MusicGen对比/提示词工程/版权）
3. `03-ai-sound-effects.md` — AI音效生成（ElevenLabs/AudioGen/提示词模板库/后处理）
4. `04-voice-speech-synthesis.md` — 人声语音合成（ElevenLabs TTS/开源TTS/人声工作流）
5. `05-audio-specs-engine-integration.md` — 音频规范与引擎适配（格式/响度/Unity/Cocos Creator/FMOD）
6. `06-audio-design-workflow.md` — 音频设计工作流（全流程/批量生产/成本估算/QA）

### 核心参考来源

**AI工具官方**：
- [Suno](https://suno.com/) — AI音乐生成
- [ElevenLabs](https://elevenlabs.io/) — 音效+人声生成
- [Meta AudioCraft (MusicGen/AudioGen)](https://github.com/facebookresearch/audiocraft) — 开源音频AI

**引擎文档**：
- [Unity Manual - Audio Compression](https://docs.unity3d.com/Manual/AudioFiles-compression.html)
- [Cocos Creator 3.8 - AudioSource](https://docs.cocos.com/creator/3.8/manual/en/audio-system/audiosource.html)
- [FMOD Licensing](https://www.fmod.com/licensing)

**行业标准**：
- [GDC Vault - Game Loudness Standards](https://gdcvault.com/play/1017781/Game-Loudness-Industry)
- [Designing Sound - Loudness In Game Audio](https://designingsound.org/2013/02/28/loudness-in-game-audio/)
- [Game Dev Beginner - Unity Audio Optimisation](https://gamedevbeginner.com/unity-audio-optimisation-tips/)

**工具与评测**：
- [AIMagicx - Suno vs Udio vs ElevenLabs 2026](https://www.aimagicx.com/blog/suno-vs-udio-vs-elevenlabs-music-comparison-2026)
- [TeamDay.ai - Best AI Music Models 2026](https://www.teamday.ai/blog/best-ai-music-models-2026)
- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
