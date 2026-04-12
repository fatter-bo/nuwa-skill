# 研究维度四：人声语音合成

## 1. 主流TTS工具对比（2026年）

### ElevenLabs — 游戏人声首选

| 特性 | 详情 |
|------|------|
| 核心模型 | Eleven v3（最高质量）/ Flash v2.5（低延迟）/ Turbo v2.5（平衡） |
| 语言支持 | v3: 70+语言；Flash/Turbo: 32语言 |
| 延迟 | Flash v2.5 ~75ms；Turbo v2.5 ~250ms；v3较高（非实时） |
| 声音克隆 | 仅需几秒音频即可克隆，支持29语言跨语言克隆 |
| 情感控制 | v3支持表情丰富的语调变化，可通过提示词控制情绪 |
| 商用授权 | 付费计划免版税商用 |
| 定价 | v3: $0.17-$0.30/千字符（按计划层级） |
| API | 完整REST API，支持流式输出 |

**游戏应用场景**：
- NPC对白：Flash v2.5适合实时对话（<100ms延迟）
- 旁白/剧情语音：v3提供最高质量
- 系统提示音：短句TTS生成"请出牌""时间到"等
- 角色语音包：克隆特定声线，批量生成台词

**来源**：ElevenLabs官方文档；Inworld - "Best Voice AI for NPC and Interactive Entertainment (2026)"；Coval.dev - "ElevenLabs Review 2026"
**可信度**：高 -- 官方文档和专业评测

### 开源TTS方案

| 模型 | 特点 | 语言 | 许可证 | 适合场景 |
|------|------|------|--------|---------|
| Coqui XTTS v2.5 | 零样本声音克隆（~6秒音频），情感迁移 | 17语言 | MPL-2.0 | 本地部署，完全控制 |
| Bark (Suno AI) | 可生成语音+笑声/叹息/背景音/简单音乐 | 多语言 | MIT | 需要非语言音效的场景 |
| ChatTTS | 中文语音质量极高，韵律自然 | 中英文 | AGPL-3.0 | 中文游戏项目 |
| Piper | 轻量级，CPU可运行，延迟极低 | 30+语言 | MIT | 端侧/嵌入式场景 |

**来源**：BentoML - "The Best Open-Source Text-to-Speech Models in 2026"；Apatero - "Open Source Text to Speech 2026"；GitHub各项目仓库
**可信度**：高 -- 开源项目官方文档

## 2. 游戏人声设计规范

### 人声类型分类

| 类型 | 用途 | 技术要求 | AI工具选择 |
|------|------|---------|-----------|
| 旁白（Narrator） | 剧情推进、教程引导 | 清晰、专业、中性语调 | ElevenLabs v3 |
| 角色对白（Dialogue） | NPC交互、角色表演 | 个性化声线、情感丰富 | ElevenLabs + 声音克隆 |
| 系统提示（System） | 操作确认、状态通知 | 简短、清脆、不干扰 | 任意TTS，短句即可 |
| 战斗呼声（Battle Cry） | 攻击、受伤、胜利 | 爆发力、情绪极端 | 人工录制+AI增强 |
| 环境人声（Crowd） | 背景人群嘈杂声 | 模糊、自然、层次感 | Bark（含非语言音效） |

### 棋牌/休闲游戏人声要点

棋牌和休闲游戏的人声需求相对简单但高频：

1. **系统播报**：
   - "叫地主""不叫""抢地主" — 短促、清晰
   - "大牌""炸弹""王炸""春天" — 带兴奋感
   - "你赢了""再来一局" — 温暖、鼓励

2. **角色语音包**：
   - 每个角色准备20-50条常用语音
   - 同一句话准备2-3个变体避免重复感
   - 使用ElevenLabs声音克隆保持角色声线一致性

3. **技术规格**：
   - 采样率：44.1kHz（与其他音效统一）
   - 格式：WAV（原始）→ OGG（打包）
   - 响度：-16 LUFS（人声优先级高于BGM）
   - 时长：系统提示 0.5-2s，对白 1-5s

**来源**：Sonarworks - "Voice AI in Game Audio & Film"；Respeecher - "Top Sound Effects Tools for Indie Game Developers"
**可信度**：高 -- 专业音频/游戏行业媒体

## 3. 人声生成工作流

### 标准流程

```
Step 1: 脚本准备
  → 整理所有需要的台词列表（Excel/CSV）
  → 标注每句的情感/语速/音量要求
  → 按角色分组

Step 2: 声线设计
  → ElevenLabs: 从Voice Library选择基础声线
  → 或上传参考音频克隆特定声线
  → 开源方案: XTTS v2 + 参考音频

Step 3: 批量生成
  → 通过API批量提交台词列表
  → 每句生成2-3个变体
  → 脚本自动命名：{角色}_{台词ID}_{变体号}.wav

Step 4: 筛选与后处理
  → 人工听审，每句选最佳变体
  → 去除首尾静音（FFmpeg silenceremove）
  → 响度标准化到-16 LUFS
  → 格式转换（WAV → OGG）

Step 5: 引擎集成
  → 按角色/场景组织文件目录
  → 引擎中设置人声音量通道（独立于BGM和音效）
  → 人声播放时自动ducking BGM（降低-6到-12dB）
```

### ElevenLabs API批量生成示例

```python
import requests
import os

API_KEY = "your_api_key"
VOICE_ID = "your_voice_id"

lines = [
    {"id": "call_landlord", "text": "叫地主", "emotion": "confident"},
    {"id": "pass", "text": "不叫", "emotion": "neutral"},
    {"id": "bomb", "text": "炸弹！", "emotion": "excited"},
    {"id": "win", "text": "你赢了！", "emotion": "happy"},
]

for line in lines:
    for variant in range(3):
        response = requests.post(
            f"https://api.elevenlabs.io/v1/text-to-speech/{VOICE_ID}",
            headers={"xi-api-key": API_KEY},
            json={
                "text": line["text"],
                "model_id": "eleven_multilingual_v2",
                "voice_settings": {
                    "stability": 0.5,
                    "similarity_boost": 0.8,
                }
            }
        )
        filename = f"{line['id']}_v{variant+1}.wav"
        with open(f"output/{filename}", "wb") as f:
            f.write(response.content)
```

**来源**：ElevenLabs API Documentation；开发者实践经验
**可信度**：高 -- 基于官方API文档

## 4. 版权与伦理考量

### 声音克隆的法律边界

- **自有声音**：克隆自己的声音用于项目，无法律风险
- **授权声音**：获得声音提供者书面授权后克隆，需明确使用范围
- **公共人物声音**：多数司法管辖区禁止未经授权模仿名人声音
- **AI生成的原创声音**：使用ElevenLabs/XTTS设计的全新声线，版权风险最低

### 最佳实践

1. 优先使用平台提供的预设声音（无版权风险）
2. 如需克隆，仅克隆团队成员或已签约配音演员的声音
3. 保留所有授权文档
4. 在游戏Credits中标注"AI-generated voice"

**来源**：ElevenLabs Terms of Service；各开源项目许可证说明
**可信度**：中高 -- 基于平台条款，但法律领域仍在演进中
