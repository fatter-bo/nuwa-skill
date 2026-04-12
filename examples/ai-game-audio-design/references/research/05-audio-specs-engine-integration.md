# 研究维度五：音频规范与引擎适配

## 1. 游戏音频格式规范

### 格式选型矩阵

| 格式 | 压缩类型 | 音质 | 文件大小 | 循环支持 | 适用场景 |
|------|---------|------|---------|---------|---------|
| WAV | 无压缩 | 最高 | 最大 | 完美 | 原始素材、短音效 |
| OGG Vorbis | 有损压缩 | 高 | 小（约WAV的1/10） | 完美 | BGM、长音频 |
| MP3 | 有损压缩 | 高 | 小 | 有缝隙（不推荐循环） | 兼容性备用 |
| ADPCM | 压缩 | 中高 | 中（约WAV的1/4） | 良好 | 频繁播放的短音效 |
| AAC | 有损压缩 | 高 | 小 | 良好 | iOS平台 |

**关键决策规则**：
- **BGM** → OGG Vorbis（文件小、音质好、完美循环）
- **短音效**（<1s）→ WAV 或 ADPCM（低延迟、CPU开销小）
- **长音效**（>2s）→ OGG Vorbis
- **人声** → OGG Vorbis
- **绝对不用MP3做循环BGM**（编码器会在首尾添加静音填充）

**来源**：Unity官方文档 - "Audio file compression in Unity"；Unity Discussions - "Audio Compression Best Practices"
**可信度**：高 -- 引擎官方文档

### 采样率与位深度

| 音频类型 | 推荐采样率 | 位深度 | 说明 |
|---------|-----------|--------|------|
| BGM | 44,100 Hz | 16-bit | 标准CD质量 |
| 音效 | 44,100 Hz | 16-bit | 与BGM统一 |
| 人声 | 44,100 Hz | 16-bit | 保证清晰度 |
| UI微音效 | 22,050 Hz | 16-bit | 可降采样率节省空间 |
| 环境音循环 | 44,100 Hz | 16-bit | 需要自然感 |

**统一到44.1kHz/16-bit是最安全的选择**，仅在包体极度紧张时对UI音效降采样率。

**来源**：Game Dev Beginner - "10 Unity Audio Optimisation Tips"；Gearspace - "Music levels for game delivery"
**可信度**：高 -- 开发者社区最佳实践

## 2. 响度标准

### 游戏行业响度规范

| 平台 | 目标响度 | 容差 | 峰值上限 | 标准依据 |
|------|---------|------|---------|---------|
| 主机（PS/Xbox/Switch） | -24 LUFS | ±2 LU | -1 dBTP | ITU-R BS.1770-3 |
| 移动端/掌机 | -16 到 -18 LUFS | ±2 LU | -1 dBTP | GANG推荐 |
| PC | -23 LUFS | ±2 LU | -1 dBTP | EBU R128参考 |

**移动休闲游戏建议**：-16 LUFS（移动端环境噪声大，需要更高响度）

### 各音频层响度分配

基于总输出-16 LUFS的分配方案：

| 音频层 | 相对音量 | 绝对目标 | 说明 |
|--------|---------|---------|------|
| BGM | -18 to -24 dB | 背景铺底 | 不干扰操作 |
| 环境音 | -20 to -26 dB | 次底层 | 若有 |
| UI音效 | -12 to -16 dB | 中层 | 清脆可闻 |
| 操作反馈音效 | -8 to -12 dB | 高层 | 即时反馈 |
| 演出音效 | -6 to -10 dB | 最高层 | 关键时刻 |
| 人声 | -10 to -14 dB | 动态 | 播放时ducking BGM |

**来源**：GDC Vault - "Game Loudness, Industry Standards"；Designing Sound - "Loudness In Game Audio"；Audiokinetic Blog - "Mastering a Game with Wwise"；Stephen Schappler - "Listening for Loudness in Video Games"
**可信度**：高 -- GDC演讲+行业标准组织

## 3. 主流引擎音频系统

### Unity音频

**支持格式**：WAV、OGG Vorbis、MP3、AIFF、MOD、IT、S3M、XM

**压缩策略**：
| 音频类型 | Load Type | Compression | 说明 |
|---------|-----------|-------------|------|
| 短音效（<200KB） | Decompress On Load | ADPCM | 低延迟，常驻内存 |
| 中等音效（200KB-1MB） | Compressed In Memory | Vorbis (Quality 70%) | 平衡内存和CPU |
| BGM/长音频（>1MB） | Streaming | Vorbis (Quality 100%) | 不占内存，流式播放 |

**移动端优化要点**：
- 单声道（Mono）用于音效（省50%内存，游戏音效通常不需要立体声）
- 立体声（Stereo）仅用于BGM
- `Force To Mono`选项：Unity导入设置中勾选
- 避免同时Streaming多个AudioClip（移动端CPU开销大）

**来源**：Unity Manual - "Audio file compression"；Game Dev Beginner - "10 Unity Audio Optimisation Tips"；Gamedeveloper - "Unity Audio Import Optimisation"
**可信度**：高 -- Unity官方文档和专业开发者指南

### Cocos Creator音频

**支持格式**：WAV、OGG Vorbis、MP3

**播放引擎**：
- Web平台：Web Audio API（首选）/ DOM Audio（降级）
- 微信小游戏：InnerAudioContext API
- 原生平台：对应平台原生音频系统

**关键限制**：
- Web平台自动播放受限：必须等待用户首次交互后才能播放音频（Chrome策略）
- 微信小游戏：同时播放音频数有上限（建议不超过10个并发）
- `playOnAwake`在Web端不会立即生效，需用户交互触发

**AudioSource组件**：
- `play()` — 播放BGM（长音频）
- `playOneShot()` — 播放音效（短音频，可叠加）
- `volume` — 音量控制（0-1）
- `loop` — 循环播放

**来源**：Cocos Creator 3.8 Manual - AudioSource组件；Cocos Creator - Audio Playback Examples
**可信度**：高 -- 官方文档

### FMOD / Wwise 中间件（进阶）

**适用场景**：音频需求复杂的中大型项目

| 中间件 | 独立授权 | Unity集成 | Unreal集成 | 核心优势 |
|--------|---------|----------|-----------|---------|
| FMOD | 年收入<$200K免费 | 官方插件 | 官方插件 | 界面直觉（类DAW），自适应音乐 |
| Wwise | 500音效以下免费 | 官方插件 | 官方插件 | 完整音频管线，空间音频强 |

**FMOD对独立开发者的价值**：
- 免费授权门槛低（年收入<$200K且预算<$500K）
- 可视化编辑器，非音频专业人员也能上手
- 自适应音乐系统：根据游戏状态动态切换BGM层
- 注意：赌博/模拟类项目不适用免费授权

**对于休闲/棋牌游戏的建议**：
- 音效数量<100个 → 直接用引擎内置音频系统即可
- 音效数量>100个或需要复杂自适应音乐 → 考虑FMOD
- AAA级音频需求 → Wwise

**来源**：FMOD Licensing页面；Wwise 2025.1 Blog；Generalist Programmer - "Game Audio Design Guide 2025"
**可信度**：高 -- 官方授权说明和专业教程

## 4. 音频文件命名与目录规范

### 推荐命名规则

```
{类型}_{场景}_{描述}_{变体号}.{格式}

示例：
bgm_lobby_jazz_lounge.ogg
bgm_battle_intense_drums.ogg
sfx_card_play_normal_v1.ogg
sfx_card_play_normal_v2.ogg
sfx_card_play_bomb_v1.ogg
sfx_ui_button_click.ogg
sfx_ui_popup_open.ogg
sfx_win_fanfare.ogg
sfx_lose_gentle.ogg
vo_system_call_landlord_v1.ogg
vo_system_call_landlord_v2.ogg
vo_char01_greeting.ogg
amb_casino_crowd.ogg
```

### 推荐目录结构

```
audio/
├── bgm/                    # 背景音乐
│   ├── lobby/
│   ├── battle/
│   └── result/
├── sfx/                    # 音效
│   ├── card/               # 牌类操作音效
│   ├── chip/               # 筹码音效
│   ├── ui/                 # UI通用音效
│   ├── merge/              # 合成类音效
│   └── special/            # 特殊演出音效
├── vo/                     # 人声
│   ├── system/             # 系统播报
│   ├── char01/             # 角色1语音包
│   └── char02/             # 角色2语音包
└── amb/                    # 环境音
    ├── casino/
    └── nature/
```

**来源**：行业实践经验；FMOD/Wwise项目组织最佳实践
**可信度**：中高 -- 综合多个项目实践
