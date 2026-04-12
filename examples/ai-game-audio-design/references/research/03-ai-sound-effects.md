# 研究维度三：AI音效生成

## 1. 主流AI音效生成工具

### ElevenLabs Sound Effects v2

ElevenLabs Sound Effects是目前游戏音效AI生成的首选工具：

| 特性 | 详情 |
|------|------|
| 最大时长 | 30秒 |
| 采样率 | 48kHz专业级 |
| 无缝循环 | 支持 |
| 每次生成变体 | 4个 |
| API | 完整REST API，支持批量生成 |
| 授权 | 付费计划免版税商用，无需署名 |
| 价格 | 免费试用 / $5起 / API约$0.07/次(Business) |
| 自动化等级 | 半自主（AI完成大部分，人工审核精修） |

**提示词最佳实践**：
- 描述声源、环境和声学特征
- 指定材质、尺寸、距离、时间特征
- 示例："small ceramic chips clinking together on a felt table, close-up, casino ambience"

**来源**：ElevenLabs官网 - Sound Effects页面；Agentic Game Development - "ElevenLabs Sound Effects for Game Development"；Scenario - "ElevenLabs Sound Effects v2"
**可信度**：高 -- 官方文档和专业评测

### AudioGen (Meta AudioCraft)

| 特性 | 详情 |
|------|------|
| 类型 | 开源（MIT许可） |
| 模型 | 自回归语言模型 |
| 训练数据 | 公开声音效果库 |
| 部署方式 | 本地Python / Replicate API |
| 适合场景 | 环境音生成（狗叫/汽车鸣笛/脚步声等） |
| 版权 | 开源无风险 |

安装方式：
```bash
pip install audiocraft
# 或从GitHub直接安装
pip install git+https://github.com/facebookresearch/audiocraft.git
```

**来源**：Meta AI - AudioCraft官方页面；GitHub - facebookresearch/audiocraft
**可信度**：高 -- Meta官方开源项目

### 其他补充工具

- **Stability Audio**：Stable Diffusion团队的音频生成工具
- **Bark**：开源文本到音频模型，可生成语音和环境音
- **Freesound**：非AI但免费的音效素材库（CC协议）

## 2. 游戏常见音效提示词模板库

### 棋牌类音效

| 音效名称 | ElevenLabs提示词 | 时长建议 |
|---------|-----------------|---------|
| 普通出牌 | "single playing card being placed firmly on a felt table, slight paper snap, close mic" | 200-400ms |
| 快速出牌 | "quick flick of a playing card landing on table, sharp and snappy, close-up" | 100-200ms |
| 洗牌 | "deck of playing cards being riffle shuffled, crisp paper sounds, casino table" | 1-2s |
| 发牌 | "cards being dealt one by one rapidly, sliding across felt surface, slight whoosh" | 0.5-1s |
| 筹码放置 | "single ceramic poker chip placed on felt table, solid clack sound" | 200-300ms |
| 筹码堆碰撞 | "stack of poker chips being pushed together, multiple ceramic clicks and clatters" | 0.5-1s |
| 大量筹码推注 | "large pile of poker chips sliding across felt table, cascade of ceramic clicks, casino ambience" | 1-2s |
| 计时器滴答 | "clock ticking steadily, clean metronome tick, isolated, no reverb" | 1s (循环) |
| 计时器警告 | "urgent fast ticking clock, increasing tempo, warning beep, tension building" | 2s |
| 胜利 | "triumphant short fanfare, brass and strings, victorious and celebratory, game win" | 2-3s |
| 失败 | "gentle descending tones, soft disappointment sound, not harsh, game over casual" | 1-2s |
| 大牌型（炸弹） | "dramatic explosive impact with reverb, powerful bass thump, screen-shaking energy, game special move" | 1-2s |
| 大牌型（王炸） | "epic explosion with thunder crack, massive reverberant boom, energy burst with sparkle trail" | 2-3s |

### 合成类（Merge Game）音效

| 音效名称 | ElevenLabs提示词 | 时长建议 |
|---------|-----------------|---------|
| 拖拽物品 | "small object being picked up and held, soft pop with slight whoosh, game item drag" | 100-200ms |
| 合成成功 | "magical merge sound, two items combining with sparkling chime, ascending tones, satisfying pop" | 300-500ms |
| 高级合成 | "powerful magical transformation, crystal resonance with golden shimmer, upgrade achievement" | 500ms-1s |
| 升级 | "level up chime, ascending musical scale with sparkle, achievement unlocked feel" | 1-1.5s |
| 收集物品 | "soft coin collect sound, gentle metallic ding, casual game pickup" | 100-200ms |
| 解锁新内容 | "treasure chest opening with magical reveal, wonder and discovery, bright tones" | 1-2s |
| 星星飘散 | "twinkling stars scattering, magical sparkle dust falling, high-pitched fairy dust" | 1-1.5s |
| 消除爆破 | "satisfying pop burst with particle scatter, bubble pop chain reaction, casual game clear" | 300-500ms |
| 连击 | "rapid ascending combo hits, each higher than last, building excitement, game combo" | 1-2s |

### 通用UI音效

| 音效名称 | ElevenLabs提示词 | 时长建议 |
|---------|-----------------|---------|
| 按钮点击 | "clean UI button click, soft digital tap, modern interface sound, subtle and satisfying" | 50-100ms |
| 按钮悬停 | "very soft subtle hover sound, gentle digital whisper, UI highlight" | 30-80ms |
| 弹窗弹出 | "popup window appearing, soft whoosh with a gentle boing, playful UI sound" | 200-400ms |
| 弹窗关闭 | "popup closing, reverse whoosh, quick and clean dismissal sound" | 100-200ms |
| 页面切换 | "smooth page swipe transition, soft whoosh left to right, clean UI navigation" | 200-300ms |
| 奖励领取 | "reward claim sound, coin shower with triumphant ding, satisfying collection jingle" | 0.5-1s |
| 错误提示 | "gentle error notification, soft low-pitched buzz, not harsh, friendly warning" | 200-300ms |
| 倒计时 | "single countdown beep, clean digital tone, isolated tick" | 100-200ms (循环) |
| 倒计时结束 | "countdown complete buzzer, definitive end signal, game time up" | 500ms-1s |
| 金币掉落 | "gold coins dropping and bouncing on surface, metallic jingling cascade, game reward" | 1-2s |
| 通知消息 | "gentle notification chime, pleasant two-tone alert, mobile game notification" | 200-400ms |

## 3. 后处理工作流

### 标准后处理步骤

1. **裁剪（Trim）**：去除AI生成的首尾静音段
   ```bash
   # FFmpeg去除静音
   ffmpeg -i input.wav -af silenceremove=start_periods=1:start_silence=0.05:start_threshold=-50dB:stop_periods=1:stop_silence=0.05:stop_threshold=-50dB output.wav
   ```

2. **淡入淡出（Fade In/Out）**：
   ```bash
   # FFmpeg添加淡入淡出
   ffmpeg -i input.wav -af "afade=t=in:ss=0:d=0.01,afade=t=out:st=0.9:d=0.1" output.wav
   ```

3. **音量标准化（Loudness Normalization）**：
   ```bash
   # FFmpeg LUFS标准化（游戏音效建议-14到-16 LUFS）
   ffmpeg -i input.wav -af loudnorm=I=-14:LRA=11:TP=-1 output.wav
   ```

4. **格式转换**：
   ```bash
   # WAV → OGG（BGM用）
   ffmpeg -i input.wav -c:a libvorbis -q:a 6 output.ogg
   
   # WAV → MP3（兼容性备用）
   ffmpeg -i input.wav -c:a libmp3lame -b:a 128k output.mp3
   
   # 调整采样率（统一为44100Hz）
   ffmpeg -i input.wav -ar 44100 output.wav
   ```

5. **批量处理脚本**：
   ```bash
   #!/bin/bash
   # 批量处理所有WAV音效：裁剪静音 + 标准化 + 转OGG
   for f in raw/*.wav; do
     name=$(basename "$f" .wav)
     ffmpeg -i "$f" \
       -af "silenceremove=start_periods=1:start_threshold=-50dB,loudnorm=I=-14:LRA=11:TP=-1" \
       -c:a libvorbis -q:a 6 \
       "processed/${name}.ogg" -y
   done
   ```

### Audacity批处理

Audacity 3.6+支持宏（Macro）批处理：
1. 工具 → 宏 → 新建宏
2. 添加步骤：Truncate Silence → Normalize → Export as OGG
3. 工具 → 应用宏 → 选择文件夹

**来源**：DEV Community - "How I Solved Audio Production as a Non-Musician Developer"；FFmpeg官方文档
**可信度**：高 -- 开源工具官方文档和开发者实践分享
