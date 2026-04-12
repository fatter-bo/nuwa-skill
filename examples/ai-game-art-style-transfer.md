---
name: ai-game-art-style-transfer
description: |
  AI游戏美术风格迁移专家。系统化完成从"原版风格分析"到"AI批量生成替换素材"到"质量验收"的完整流程。
  基于Stable Diffusion/ComfyUI生态的ControlNet、IP-Adapter、LoRA、InstantStyle等一致性控制技术，
  结合TexturePacker图集打包和Cocos Creator/Unity引擎导出规范，提供端到端的游戏美术风格迁移Pipeline。
  当用户说「风格迁移」「换皮」「美术替换」「AI生成素材」「风格转换」「美术Pipeline」时使用。
  也适用于「帮我把这套素材改成日系风格」「批量生成游戏图标」「怎么保持AI生成的风格一致」等场景。
---

# AI游戏美术风格迁移专家

> 不是从零创作，而是快速高质量地风格迁移——游戏美术的"义乌模式"。

## 使用说明

**这个Skill不扮演角色，是一个系统化工具。** 用户提出游戏美术风格迁移相关需求时，按照下面的工作流自动执行。

---

## 执行工作流（Agentic Protocol）

**用户说"帮我把这套素材做风格迁移"时，按以下流程执行：**

```
Phase 0: 需求澄清（确认输入输出）
  → 原版素材类型和数量（角色/道具/场景/UI/特效）
  → 原版风格标签（中国风/日系/韩系/欧美等）
  → 目标风格标签
  → 目标引擎和平台（Cocos Creator/Unity/Web）
  → 素材规格要求（分辨率/格式/命名规范）
  → 预算和时间约束（决定技术选型）

Phase 1: 原版风格分析
  → 用"风格分析五维度模型"拆解原版素材
  → 输出风格分析报告（色彩/线条/光影/比例/质感）
  → 确认与用户的理解一致

Phase 2: 迁移方案设计
  → 用"文化风格迁移矩阵"确定迁移路径
  → 用"一致性控制决策树"选择技术方案
  → 编写提示词模板（按素材类型分类）
  → 设计ComfyUI工作流

Phase 3: 试生成与校准（关键！）
  → 每类素材先生成3-5张样本
  → 用户确认风格方向
  → 根据反馈调整提示词和参数
  → 确认后进入批量生成

Phase 4: 批量生成
  → ComfyUI API批量入队
  → 实时监控生成进度
  → 错误处理与重试

Phase 5: 后处理
  → 去背景（rembg/ComfyUI-RMBG）
  → 尺寸标准化
  → 色调校正（直方图匹配）
  → 人工筛选（A/B/C/D分级）

Phase 6: 打包验收
  → TexturePacker图集打包
  → 引擎格式导出
  → 质量验收（自动指标+人工检查）
  → 输出验收报告
```

---

## 核心方法论框架

### 框架1: 风格分析五维度模型

对任意一套游戏美术做系统化风格拆解：

| 维度 | 分析内容 | 量化方法 |
|------|---------|---------|
| 色彩体系 | 主色调/辅助色/强调色的HSL分布，色温倾向 | Adobe Color提取色板 + OpenCV HSL直方图 |
| 线条特征 | 描边有无/粗细/颜色/圆润度 | Canny边缘检测 + 线宽统计 |
| 光影风格 | 五级分类：L1平涂→L5写实PBR | 目视判断 + 阴影边缘锐利度分析 |
| 比例规范 | 头身比（2-3头身极Q版→8+头身写实） | 头部占比测量 |
| 纹理质感 | 数字感/手绘感/像素感/厚涂感/水墨感 | 高频纹理分析 |

**光影五级分类**：
- **L1 纯平涂**：无明暗变化，纯色块（超休闲游戏）
- **L2 赛璐璐**：2-3级明暗阶梯，锐利阴影边缘（日系动漫）
- **L3 软赛璐璐**：阴影边缘有过渡，2.5D感（韩系手游）
- **L4 半写实**：渐变光影保留手绘感（《原神》等）
- **L5 写实渲染**：完整PBR光照（AAA写实）

来源：[GarageFarm Cel Shading Guide](https://garagefarm.net/blog/cel-shading-a-comprehensive-guide)、[Cel shading - Wikipedia](https://en.wikipedia.org/wiki/Cel_shading)

---

### 框架2: 一致性控制技术选型决策树

解决批量生成时风格漂移问题的技术选择：

```
需要风格一致性？
├── 有明确风格参考图（>15张）？
│   ├── 是 → 长期项目 → 训练LoRA（最高一致性）
│   │   参数: Rank 32-128, Alpha=Rank/2, LR=0.0001, 2000-5000步
│   └── 否 → IP-Adapter + ControlNet组合
│       参数: IP-Adapter Scale 0.5-0.7, ControlNet Scale 0.7-0.9
├── 仅有少量参考（1-5张）？
│   ├── 需要精确结构控制 → ControlNet(Lineart) + InstantStyle
│   └── 风格引导即可 → IP-Adapter Plus (Scale 0.6-0.8)
├── 使用Midjourney？
│   └── --sref(风格) + --cref(角色) 组合
└── 完全无参考？
    └── 详细文本描述 + 种子固定 + 批量生成后筛选
```

**技术方案对比**：

| 方案 | 训练需求 | 一致性 | 速度 | 适用场景 |
|------|---------|--------|------|---------|
| LoRA | 15-200张图+数小时 | 最高 | 训练慢/推理快 | 长期项目、明确风格 |
| IP-Adapter | 无 | 高 | 快 | 有参考图的快速迁移 |
| ControlNet | 无 | 结构高/风格中 | 快 | 保持构图结构 |
| InstantStyle | 无 | 高（纯风格） | 快 | 避免内容泄露的风格迁移 |
| MJ --sref/--cref | 无 | 高 | 快 | Midjourney生态 |

来源：[InstantStyle](https://instantstyle.github.io/)、[Stable Diffusion Art - IP-Adapter](https://stable-diffusion-art.com/ip-adapter/)、[Stable Diffusion Art - ControlNet](https://stable-diffusion-art.com/controlnet/)、[arxiv - ICAS](https://arxiv.org/html/2504.13224v1)

---

### 框架3: 文化风格迁移矩阵

常见迁移路径的核心调整维度：

| 从 → 到 | 色彩 | 线条 | 光影 | 比例 |
|---------|------|------|------|------|
| 中国风→日系清新 | +饱和度30%，加粉蓝绿 | 水墨线→均匀描边 | 渐变→赛璐璐 | 维持或微调 |
| 中国风→韩系Q版 | +粉彩色调 | 线条圆润化+细描边 | →软赛璐璐 | →2.5-3.5头身 |
| 欧美写实→日系动漫 | +饱和度大幅提高 | 加黑色描边 | PBR→赛璐璐 | 眼睛放大/五官简化 |
| 日系→韩系Q版 | +柔化+糖果色 | 圆润化 | →软赛璐璐 | →降头身比 |
| 欧美卡通→中国风 | 降饱和度+水墨色系 | 粗描边→飘逸线条 | →水墨渐变 | 正常化比例 |

**文化注意事项**：
- 云纹形状（中国祥云 vs 日本松形云）、花卉选择（牡丹/梅花 vs 樱花/菊花）差异显著
- 龙的爪数有文化含义（中国五爪、日本三四爪）
- 服饰领口/腰带/裙型在汉服/和服/韩服间完全不同
- 宗教符号和颜色含义需避免文化禁忌

来源：[SyncedReview - CartoonGAN](https://syncedreview.com/2018/07/14/reproducing-japanese-anime-styles-with-cartoongan-ai/)、游戏美术外包行业实践

---

### 框架4: 批量生产Pipeline架构

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│ 素材准备     │────→│ ComfyUI工作流 │────→│ 批量生成     │
│ 分类+规格记录│     │ ControlNet    │     │ API Queue    │
│             │     │ + IP-Adapter  │     │ Python脚本   │
└─────────────┘     └──────────────┘     └──────┬──────┘
                                                 │
                    ┌──────────────┐     ┌───────▼──────┐
                    │ 打包导出     │←────│ 后处理       │
                    │ TexturePacker│     │ 去背景+尺寸  │
                    │ 引擎格式     │     │ 色调校正     │
                    └──────────────┘     └──────────────┘
```

**关键技术组件**：
- **ComfyUI API**：所有UI操作可编程化，通过POST `http://127.0.0.1:8188/prompt` 入队
- **rembg**：基于U-2-Net的离线去背景，批量处理2-3秒/张
- **TexturePacker**：图集打包，减包体+合批渲染+降Draw Call
- **引擎导出**：Cocos Creator用cocos2d-x格式(.plist+.png)，Unity用Texture2D格式

来源：[ComfyUI API](https://blog.xentoo.info/2023/07/22/comfyui-batch-run-from-command-line-with-api/)、[rembg GitHub](https://github.com/danielgatis/rembg)、[TexturePacker Docs](https://www.codeandweb.com/texturepacker/documentation)

---

### 框架5: 提示词工程模板体系

按游戏素材类型分类的SD/SDXL提示词结构：

**通用公式**：`[主体描述], [风格修饰], [质量修饰], [技术参数]`

| 素材类型 | 核心提示词 | 关键参数 |
|----------|-----------|---------|
| 角色立绘 | `game character art, full body, front view, [风格], white background` | 高分辨率，clean lines |
| 道具图标 | `game item icon, centered, single object, [风格], simple background` | isometric view可选 |
| 场景背景 | `game background, wide shot, landscape, [风格], no characters` | 适配视角(side-scroll/top-down) |
| UI元素 | `game UI element, flat design, clean edges, transparent background` | 高对比度，可读性优先 |
| 特效贴图 | `VFX texture, game effect, black background, glowing, seamless` | 透明边缘 |

**img2img去噪强度参考**：
- 0.3-0.5：保持构图，改变风格（推荐起始值）
- 0.5-0.7：较大风格变化，构图基本保持
- 0.7-0.9：大幅重新生成

来源：[Galaxy.ai - SD Prompts for Game Assets](https://blog.galaxy.ai/stable-diffusion-prompts-for-game-asset)、[Midjourney Docs](https://docs.midjourney.com/hc/en-us/articles/32180011136653-Style-Reference)、[Hugging Face - ML for Games](https://huggingface.co/blog/ml-for-games-4)

---

### 框架6: 质量验收体系

**自动化指标**：

| 指标 | 用途 | 判断标准 |
|------|------|---------|
| LPIPS | 单张感知相似度 | <0.3相似，0.3-0.5可接受，>0.5需重做 |
| SSIM | 单张结构相似度 | >0.7相似，0.5-0.7有差异，<0.5差异大 |
| FID | 批量分布距离 | <50良好，50-100可接受，>100需调整 |

**素材分级**：
- **A级**（直接可用）+ **B级**（微调后可用）≥ 70% → 验收通过
- **D级**（不可用）< 10% → 验收通过
- 关键素材（主角/核心UI）必须达到A级

**人工检查清单**：
1. 色板一致性（HSL偏差检查）
2. 线条一致性（粗细/颜色/风格统一）
3. 光影一致性（级别统一）
4. 比例一致性（同类角色头身比偏差<0.5）
5. 透明通道完整性（无白边/毛边）
6. 引擎内渲染效果（实际尺寸清晰度）
7. 文化适配正确性

来源：[Hugging Face - Metrics for Image Generation](https://huggingface.co/blog/PrunaAI/objective-metrics-for-image-generation-assessment)、[NeoWork - Game Art Outsourcing](https://www.neowork.com/insights/game-art-outsourcing-guide)

---

## 决策启发式

1. **先试后批**：任何批量生成前，必须先生成3-5张样本让用户确认方向。省参数调整的大量返工。

2. **LoRA是杀手锏但有成本**：如果项目需要生成>100张同风格素材，投入数小时训练LoRA的ROI极高；如果只需要<20张，用IP-Adapter即可。

3. **去噪强度是风格迁移的核心旋钮**：img2img的denoising strength 0.4-0.6是风格迁移的甜蜜点——太低不变风格，太高丢结构。

4. **ControlNet控结构，IP-Adapter控风格**：两者正交，组合使用效果最好。ControlNet Scale 0.7-0.9 + IP-Adapter Scale 0.5-0.7 是常用起始参数。

5. **色板校正不能省**：AI生成的批量素材即使风格一致，色调也可能有微漂移。用直方图匹配做批量色调统一是必要的后处理步骤。

6. **2的幂次分辨率**：游戏引擎对2的幂次纹理（64/128/256/512/1024）有硬件优化。AI生成后一定要标准化到这些尺寸。

7. **文化迁移先查禁忌**：做跨文化风格迁移前，先检查文化符号禁忌清单。龙爪数、花卉选择、颜色含义都可能踩雷。

8. **FID评估批量，LPIPS评估单张**：不要用FID评估单张素材质量（它是分布指标）。单张对比用LPIPS + SSIM。

9. **ComfyUI工作流导出为JSON**：所有工作流都应保存API格式JSON。这是可复现性和自动化的基础。

10. **关键素材手动把关**：主角立绘、核心UI等高曝光素材不能纯靠自动化。A级标准 = 人工确认无异议。

---

## 工具与资源推荐

### 核心工具链

| 环节 | 工具 | 用途 | 链接 |
|------|------|------|------|
| AI生成引擎 | ComfyUI | 节点式工作流，API支持 | [github.com/comfyanonymous/ComfyUI](https://github.com/comfyanonymous/ComfyUI) |
| 模型仓库 | CivitAI | 下载LoRA/Checkpoint | [civitai.com](https://civitai.com/) |
| LoRA训练 | Kohya_ss | 最成熟的LoRA训练工具 | [github.com/bmaltais/kohya_ss](https://github.com/bmaltais/kohya_ss) |
| 去背景 | rembg | 离线批量去背景 | [github.com/danielgatis/rembg](https://github.com/danielgatis/rembg) |
| 超分辨率 | Real-ESRGAN | AI放大低分辨率输出 | [github.com/xinntao/Real-ESRGAN](https://github.com/xinntao/Real-ESRGAN) |
| 图集打包 | TexturePacker | 精灵图集生成 | [codeandweb.com/texturepacker](https://www.codeandweb.com/texturepacker) |
| 色板提取 | Adobe Color | 从图片提取色板 | [color.adobe.com](https://color.adobe.com/) |
| 图像处理 | Pillow / OpenCV | 批量尺寸/色调处理 | Python标准库 |

### 专用AI游戏资产平台

| 平台 | 特点 |
|------|------|
| [Scenario.gg](https://www.scenario.com/) | 专为游戏资产设计的AI生成平台 |
| [Leonardo.ai](https://leonardo.ai/) | 支持训练自定义模型 |
| [Layer.ai](https://layer.ai/) | 游戏素材AI生成和编辑 |

### CivitAI推荐模型方向

| 类型 | 参考方向 | 说明 |
|------|---------|------|
| 动漫风格 | Mistoon_Anime、Illustrij | 明亮2D动漫/2.5D风格 |
| 角色设计 | Charturn类LoRA | 多视角转身图/角色设计表 |
| 图标风格 | 搜索"game icon"相关LoRA | 道具图标专用 |

---

## 诚实边界

此工具的局限：

1. **AI无法完全替代美术师**：AI生成的素材在细节精度和创意表现上仍有局限。复杂的角色设计、叙事性场景、高度原创的视觉概念仍然需要人类美术师参与。本工具定位是"加速器"而非"替代品"。

2. **风格一致性无法100%保证**：即使使用LoRA+ControlNet+IP-Adapter的最强组合，批量生成仍会有微小的风格漂移。后处理校正和人工筛选不可省略。

3. **文化风格迁移的"神韵"难以量化**：HSL数值和线条参数可以量化，但一个风格是否"看起来对味"是主观判断。自动化指标（LPIPS/SSIM/FID）只是辅助，最终还需要目标文化背景的人来把关。

4. **模型和工具快速迭代**：SD/ComfyUI/IP-Adapter等工具的版本更新频繁，本文档中的具体参数和工作流可能在数月后需要更新。建议以官方文档为准。

5. **版权和合规风险**：AI生成素材的版权归属在法律上仍有争议。使用AI做风格迁移可能涉及原版素材的衍生作品问题。商业项目需咨询法律意见。

6. **硬件门槛**：SDXL级别的模型推理需要8GB+ VRAM的GPU。LoRA训练推荐RTX 3090/4090（24GB VRAM）。无GPU可使用云服务但会增加成本。

7. **不覆盖3D资产**：本框架聚焦2D游戏美术素材（立绘/图标/背景/UI/特效贴图）。3D模型、骨骼动画、粒子特效等不在本Skill覆盖范围内。

---

## 附录：调研来源

调研过程详见 `references/research/` 目录（6个文件）。

### 核心参考来源

**官方文档与工具**：
- [ComfyUI GitHub](https://github.com/comfyanonymous/ComfyUI) — 核心AI生成引擎
- [Stable Diffusion Art](https://stable-diffusion-art.com/) — ControlNet/IP-Adapter/LoRA权威教程
- [Midjourney Docs](https://docs.midjourney.com/) — --sref/--cref官方文档
- [TexturePacker Documentation](https://www.codeandweb.com/texturepacker/documentation) — 图集打包工具
- [Cocos Creator Docs - Atlas](https://docs.cocos.com/creator/3.8/manual/en/asset/atlas.html) — 引擎图集规范
- [Unity Manual - Sprite Packer](https://docs.unity3d.com/Manual/SpritePackerModes.html) — Unity精灵图集
- [rembg GitHub](https://github.com/danielgatis/rembg) — 去背景工具

**学术论文**：
- [InstantStyle](https://instantstyle.github.io/) — 风格-内容解耦的无需训练风格迁移
- [InstantStyle-Plus (arxiv 2407.00788)](https://arxiv.org/abs/2407.00788) — 增强版内容保持风格迁移
- [ICAS (arxiv 2504.13224)](https://arxiv.org/html/2504.13224v1) — IP-Adapter+ControlNet联合注意力结构
- [CartoonGAN (CVPR 2018)](https://syncedreview.com/2018/07/14/reproducing-japanese-anime-styles-with-cartoongan-ai/) — 照片转动漫风格开创性工作

**社区资源**：
- [CivitAI](https://civitai.com/) — LoRA/Checkpoint模型仓库
- [Hugging Face - ML for Games](https://huggingface.co/blog/ml-for-games-4) — 2D游戏资产生成指南
- [Hugging Face - Image Quality Metrics](https://huggingface.co/blog/PrunaAI/objective-metrics-for-image-generation-assessment) — 图像质量评估指标

**行业实践**：
- [NeoWork - Game Art Outsourcing Guide](https://www.neowork.com/insights/game-art-outsourcing-guide) — 美术外包风格规范
- [Galaxy.ai - SD Prompts for Game Assets](https://blog.galaxy.ai/stable-diffusion-prompts-for-game-asset) — 游戏素材提示词实践
- [Pav Creations - Color Theory for Game Art](https://pavcreations.com/color-theory-for-game-art-design-the-basics/) — 游戏美术色彩理论

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
