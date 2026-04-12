# 维度2：提示词工程体系

## 研究目标
建立针对游戏素材的AI图像生成提示词模板库，覆盖SD/Midjourney/Flux主流工具。

---

## 1. 通用提示词结构

### SD/SDXL提示词公式
```
[主体描述], [风格修饰], [质量修饰], [技术参数]
```

**关键原则**：
- 提示词应具体、描述性，包含关键视觉元素的精确细节
- 使用游戏特定术语如"8-bit pixel art""low-poly 3D render"引导AI风格
- 引用相关的美术风格、游戏、艺术家或工作室名称可召唤特定美学
- 来源：[Galaxy.ai - SD Prompts for Game Assets](https://blog.galaxy.ai/stable-diffusion-prompts-for-game-asset) | 可信度：高（一手实践教程）

### Midjourney特有参数
- `--sref [URL]`：风格参考，锁定画面的"镜头感"和美术方向
- `--cref [URL]`：角色参考，锁定角色身份（面部、发型、配色）
- `--sref` + `--cref` 可组合使用，实现"同一角色+同一风格"的批量生成
- `--sw [0-1000]`：风格参考强度，默认100
- 来源：[Midjourney Docs - Style Reference](https://docs.midjourney.com/hc/en-us/articles/32180011136653-Style-Reference) | 可信度：极高（官方文档）

### Flux提示词特点
- Flux更依赖自然语言描述而非关键词堆叠
- 支持更长的prompt上下文
- 对具体的画面构图描述响应更好

---

## 2. 按素材类型分类的提示词模板

### 2.1 角色立绘
```
[角色描述], [姿态], [服装细节], [表情],
game character art, full body, front view,
[风格关键词: anime/cel-shaded/semi-realistic],
white background, clean lines, high detail,
masterpiece, best quality
```

**负面提示词**：
```
bad anatomy, extra limbs, deformed, blurry,
low quality, watermark, text, signature,
3d render (如需2D), photo (如需插画风)
```

### 2.2 道具图标
```
[道具名称], [材质描述], game item icon,
centered composition, single object,
[风格: flat design/hand-painted/realistic],
simple background, clean edges,
high resolution icon, game UI asset
```

**关键技巧**：
- 图标通常需要`centered composition`和`simple/transparent background`
- 使用`isometric view`可生成等轴视角道具
- 来源：[PromptBase - SD Game Prompts](https://promptbase.com/stable-diffusion-games) | 可信度：中高

### 2.3 场景背景
```
[场景描述], [时间/天气], [氛围],
game background, wide shot, landscape,
[风格: painted/anime/pixel art],
no characters, environment concept art,
[引擎适配: side-scrolling/top-down/isometric]
```

### 2.4 UI元素
```
[UI元素类型: button/frame/panel],
[风格: fantasy/sci-fi/casual],
game UI element, flat design, clean edges,
transparent background, vector style,
high contrast, readable
```

### 2.5 特效贴图
```
[特效类型: fire/lightning/magic circle],
[颜色], VFX texture, game effect,
black background, glowing, particle effect,
seamless, transparent edges
```

---

## 3. Image2Image工作流

### 核心参数
- **Denoising Strength（去噪强度）**：0→完全复制原图，1→完全重新生成
  - 0.3-0.5：保持原版构图和主要特征，改变风格
  - 0.5-0.7：较大的风格变化，构图基本保持
  - 0.7-0.9：大幅重新生成，仅保留基本形状
- 来源：[Hugging Face - ML for Games](https://huggingface.co/blog/ml-for-games-4) | 可信度：高（一手技术文档）

### 风格迁移的img2img策略
1. 将原版素材作为输入
2. 使用目标风格的提示词
3. 去噪强度0.4-0.6（平衡风格变化和结构保持）
4. 多次生成取最佳，或使用batch后人工筛选

---

## 4. 专用平台

| 平台 | 特点 | 适合场景 |
|------|------|---------|
| Scenario.gg | 专为游戏资产设计的AI生成平台 | 角色/道具/场景批量生成 |
| Layer.ai | 游戏素材AI生成和编辑 | 快速原型和变体 |
| Leonardo.ai | 带有训练自定义模型功能 | 风格一致性要求高的项目 |

- 来源：[Hugging Face Blog](https://huggingface.co/blog/ml-for-games-4) | 可信度：高

---

## 5. 质量提升关键词速查

| 目的 | 正面提示词 | 负面提示词 |
|------|-----------|-----------|
| 提高质量 | masterpiece, best quality, high detail | worst quality, low quality, blurry |
| 清晰线条 | clean lineart, sharp edges | messy lines, rough sketch |
| 游戏适配 | game asset, sprite, icon | 3d render(如需2D), photo(如需插画) |
| 透明背景 | white background, simple background | complex background, busy |
| 一致性 | consistent style, uniform lighting | inconsistent, mixed styles |
