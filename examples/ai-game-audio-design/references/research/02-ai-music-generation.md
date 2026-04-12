# 研究维度二：AI音乐生成

## 1. 主流AI音乐生成工具能力对比

### Suno / Udio / MusicGen 三强对比（2026年4月）

| 维度 | Suno (v4.5+) | Udio (2026) | MusicGen (Meta) |
|------|-------------|-------------|-----------------|
| 类型 | 在线SaaS | 在线SaaS | 开源本地部署 |
| 音乐质量 | 极高，接近人类作曲 | 高，擅长流行/摇滚 | 中高，可定制性强 |
| 最大时长 | 4分钟（可扩展） | 2分钟 | 30秒（可拼接） |
| 人声生成 | 支持（含歌词） | 支持（含歌词） | 纯器乐 |
| BPM控制 | 支持（prompt/metatag） | 有限 | 支持（参数） |
| 循环模式 | 支持（seamless loop） | 有限 | 需后处理 |
| API | 提供REST API | 提供API | 本地Python调用 |
| 商用授权 | Pro/Premier可商用 | 受限（"围墙花园"） | MIT许可证，完全商用 |
| 价格 | 免费/10/30 USD月 | 免费/10/30 USD月 | 免费（自部署算力） |
| 适合场景 | 全能型游戏BGM | 流行风格BGM | 对版权零风险要求场景 |

**来源**：Billboard - "What Do the Suno and Udio Licensing Deals Mean for the Future of AI Music?"；SoundGuys - "Best AI Music Generators in 2026"；TeamDay.ai - "Best AI Music Models 2026"
**可信度**：高 -- 权威音乐行业/科技媒体

### 重要补充工具

- **ElevenLabs Music**：2026年新入场，质量追赶Suno
- **Beatoven.ai**：面向商用配乐场景，强调无版权风险
- **AIVA**：古典/管弦乐风格突出，适合RPG/策略游戏

## 2. 提示词工程（Prompt Engineering）

### Suno提示词结构

Suno的提示词采用"描述式"而非"指令式"，核心要素：

```
[Genre/Style] + [Mood/Emotion] + [Instruments] + [BPM/Tempo] + [Additional Details]
```

**关键原则**（来自Suno官方Hub和MusicSmith.ai指南）：

1. **描述，不要命令**：写"a gentle piano ballad"而非"create a piano ballad"
2. **指定子风格和年代**：如"1980s synth-pop"比"electronic"更精准
3. **Metatag是关键**：在歌词字段中用方括号标注结构
   ```
   [Intro]
   [Verse]
   [Chorus]
   [Bridge]
   [Outro]
   [Fade Out]
   ```
4. **BPM和Key明确指定**：如"120 BPM, key of C major"
5. **负面提示**：描述不想要的元素，如"no vocals, no drums"

### 游戏BGM专用提示词模板

**休闲游戏主界面BGM**：
```
Style: cheerful casual game soundtrack, light and playful
Instruments: piano, xylophone, ukulele, light synth pads, gentle bells
Tempo: 110 BPM
Mood: happy, carefree, warm
Additional: seamless loop, no fade in/out, no vocals, no reverb tails
```

**棋牌游戏BGM**：
```
Style: smooth jazz lounge, sophisticated card game atmosphere
Instruments: brushed drums, upright bass, muted trumpet, piano
Tempo: 85 BPM
Mood: relaxed, classy, understated
Additional: seamless loop, background music that doesn't distract from thinking, minimal melodic movement
```

**合成游戏关卡BGM**：
```
Style: whimsical puzzle game soundtrack, magical and curious
Instruments: marimba, pizzicato strings, celesta, soft synth
Tempo: 100 BPM
Mood: curious, delightful, slightly mysterious
Additional: seamless loop, evolving subtle variations
```

**来源**：Suno Hub - "The Best Ways To Create Music With AI Using Suno"；MusicSmith.ai - "Suno AI Prompt Guide"；Song AI Farm - "Suno AI Lyrics Best Practices & Prompt Engineering Guide 2026"
**可信度**：高 -- 含Suno官方资源

## 3. 版权归属和商用授权

### Suno版权条款（2026年3月最新）

| 计划 | 所有权 | 商用 | 流媒体发行 | YouTube变现 | 广告使用 |
|------|--------|------|-----------|------------|---------|
| Free | Suno持有 | 不可 | 不可 | 不可 | 不可 |
| Pro ($10/月) | 用户持有 | 可以 | 可以 | 可以 | 可以 |
| Premier ($30/月) | 用户持有 | 可以 | 可以 | 可以 | 可以 |

**关键风险点**：
- 无赔偿保障（标准计划无indemnification），用户自担侵权风险
- Suno保留使用订阅用户作品用于"运营、改进和推广服务"的许可
- 美国版权局要求"人类作者身份"，纯AI生成音乐的版权注册存在不确定性
- 2024年6月UMG/Sony/Warner联合诉讼仍在进行中

### Udio版权现状（2026年）

- 2025年10月与Universal Music Group达成和解
- 转型为"围墙花园"模式：创作内容不能离开平台
- 2025-2026过渡期间曾暂停所有下载功能
- 商用风险较高，不建议作为游戏BGM主力工具

### MusicGen版权优势

- MIT许可证，代码和模型权重完全开源
- 训练数据为Meta自有或专门授权的音乐（约40万录音/2万小时）
- 生成物无版权纠纷风险
- 缺点：音质和创意性不及Suno

**来源**：Terms.Law - "Can You Sell Suno AI Music? Commercial Rights Guide 2026"；Billboard - "What Do the Suno and Udio Licensing Deals Mean for the Future of AI Music?"
**可信度**：高 -- 法律分析网站+权威行业媒体

## 4. 循环BGM制作技巧

### AI生成 + 后处理的标准流程

1. **生成阶段**：在提示词中加入"seamless loop, no fade in/out, no reverb tails"
2. **检查循环点**：用Audacity打开生成的音频，检查首尾波形
3. **Crossfade处理**：如果循环点不自然，截取首尾各0.5-1秒做crossfade
4. **验证循环**：将音频复制拼接2-3份连续播放，检查循环点是否有断裂感
5. **标准化**：用Loudness Normalization统一到-14 LUFS（游戏BGM标准）

### Suno循环专用功能（2025年更新）

Suno在2025年4月更新后支持：
- Loop模式：生成无缝循环音频片段
- 可指定BPM和Key
- Suno Studio提供多轨编辑环境，含BPM、音量、音高的专业控制
- "Style Influence"滑块控制风格一致性

**来源**：Suno Hub - Loop功能说明；Suno Help - "Suno Sounds: Generate Custom Audio Samples"
**可信度**：高 -- Suno官方文档

## 5. 多首BGM风格一致性

### 确保一致性的方法

1. **固定Style Description**：多首BGM使用相同的风格描述基底，只修改情绪/场景参数
2. **使用Reference Audio**：Suno Studio支持上传参考音频，生成风格相似的新曲
3. **统一后处理**：所有BGM统一通过相同的EQ/压缩/响度标准化链处理
4. **建立音色表**：预先定义使用的乐器组合，每首曲目都从中选取
5. **Key一致性**：同一游戏的BGM使用相近的调性（如都在C/F/G大调之间切换）

### 风格一致性提示词模板

```
[基底不变部分]
Style: [游戏名] game soundtrack, [统一风格关键词]
Instruments: [固定乐器列表]
Key: [统一调性]

[每首变化部分]
Mood: [具体场景情绪]
Tempo: [具体BPM]
Scene: [具体使用场景]
```

**来源**：MusicSmith.ai - "Suno AI Prompt Guide"；实践经验总结
**可信度**：中高 -- 结合官方指南和行业实践
