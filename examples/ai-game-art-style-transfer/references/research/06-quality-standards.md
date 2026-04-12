# 维度6：质量验收标准

## 研究目标
建立AI生成游戏素材的质量验收体系，包含自动化指标和人工检查清单。

---

## 1. 风格一致性检查

### 自动化指标

| 指标 | 全称 | 用途 | 判断标准 |
|------|------|------|---------|
| SSIM | Structural Similarity Index | 比较局部像素模式（亮度/对比度/结构） | >0.7为结构相似 |
| LPIPS | Learned Perceptual Image Patch Similarity | 基于深度学习的感知相似度 | <0.3为感知相似 |
| FID | Fréchet Inception Distance | 比较两组图像的特征分布距离 | <50为质量较好 |

**指标选用建议**：
- **单张素材对比原版**：用LPIPS（最贴近人类感知）和SSIM
- **批量素材整体评估**：用FID（评估一批生成图与一批参考图的分布差异）
- **FID适合大集合评估，单张对比不合适**
- 来源：[Hugging Face - Objective Metrics for Image Generation](https://huggingface.co/blog/PrunaAI/objective-metrics-for-image-generation-assessment) | 可信度：极高（权威技术平台）

### 人工风格一致性检查清单

```
□ 色板一致性
  - 主色调HSL偏差 < ±10°/±15%/±15%
  - 辅助色出现频率一致
  - 强调色使用位置和面积合理

□ 线条一致性
  - 描边粗细偏差 < ±1px
  - 描边颜色一致
  - 线条风格（圆润/锐利）统一

□ 光影一致性
  - 光影级别统一（同一批全为L2或全为L3）
  - 阴影颜色倾向一致
  - 高光处理方式一致

□ 比例一致性
  - 同类角色头身比偏差 < 0.5
  - 五官比例风格统一
  - 道具相对角色的比例合理

□ 质感一致性
  - 纹理叠加方式统一
  - 笔触风格一致
  - 抗锯齿处理统一
```

---

## 2. 资源规格合规性验证

### 通用规格检查

| 检查项 | 标准 | 自动化方法 |
|--------|------|-----------|
| 分辨率 | 符合目标引擎要求（2的幂次） | PIL检查image.size |
| 色彩模式 | RGBA（含透明通道）/ RGB | PIL检查image.mode |
| 文件格式 | PNG（透明）/ JPG（不透明）/ WebP | 扩展名+magic bytes验证 |
| 文件命名 | 符合项目命名规范（小写+下划线） | 正则表达式检查 |
| 文件大小 | 单张 < 阈值（如2MB） | os.path.getsize() |
| 透明通道 | 需要透明的素材确实有alpha通道 | PIL检查alpha通道 |
| 无白边/毛边 | 去背景后无残留白边 | 边缘像素alpha值检查 |

### 引擎特定检查

#### Cocos Creator
- 图集使用.plist + .png格式
- 单张纹理不超过2048x2048（移动端）
- 图集打包后Draw Call数量验证
- 来源：[Cocos Creator Docs](https://docs.cocos.com/creator/3.8/manual/en/asset/atlas.html) | 可信度：极高

#### Unity
- Sprite Atlas正确配置
- Pixels Per Unit设置合理
- 压缩格式匹配目标平台（ASTC/ETC2/PVRTC）
- 来源：[Unity Docs - Sprite Packer](https://docs.unity3d.com/Manual/SpritePackerModes.html) | 可信度：极高

---

## 3. 引擎内实际渲染效果验证

### 必须在引擎内检查的项目

```
□ 实际显示尺寸是否正确（不被缩放模糊）
□ 透明边缘是否干净（无黑边/白边）
□ 与其他素材的风格是否协调
□ 在目标分辨率下是否清晰
□ 动画帧切换是否流畅（如有序列帧）
□ UI元素交互态是否正常（normal/pressed/disabled）
□ 不同屏幕尺寸下的适配表现
□ 性能影响（Draw Call数、内存占用）
```

### 多设备验证
- 至少测试3种分辨率（低端/中端/高端设备）
- 验证不同DPI下素材的显示效果
- 来源：[Unity Testing Best Practices](https://unity.com/how-to/testing-and-quality-assurance-tips-unity-projects) | 可信度：高

---

## 4. A/B对比评估

### 与原版素材的差异度评估

#### 自动化对比
```python
# LPIPS对比（需要安装lpips库）
import lpips
loss_fn = lpips.LPIPS(net='alex')
distance = loss_fn(img_original, img_generated)
# distance < 0.3: 感知相似
# distance 0.3-0.5: 有明显差异但可接受
# distance > 0.5: 差异过大

# SSIM对比（需要安装scikit-image）
from skimage.metrics import structural_similarity as ssim
score = ssim(img_original, img_generated, multichannel=True)
# score > 0.7: 结构相似
# score 0.5-0.7: 有结构差异
# score < 0.5: 结构差异大
```

#### 人工A/B对比清单
```
□ 整体印象：新素材能否"无违和感"替换原素材？
□ 辨识度：角色/道具是否仍可辨识其功能/身份？
□ 情绪传达：素材传达的情绪/氛围是否合适？
□ 细节水平：细节量是否与原版相当？
□ 可读性：在游戏实际尺寸下是否清晰可辨？
□ 文化适配：目标风格的文化特征是否正确体现？
```

---

## 5. 质量分级标准

| 等级 | 标准 | 处理方式 |
|------|------|---------|
| A级 | 无需修改，直接可用 | 入库 |
| B级 | 需微调（色调/尺寸/小瑕疵） | AI后处理或人工微调后入库 |
| C级 | 有明显问题但可修复（局部变形/背景残留） | 人工修复或重新生成 |
| D级 | 不可用（严重变形/风格偏差/内容错误） | 丢弃，调整参数重新生成 |

### 验收通过标准
- A级 + B级占比 ≥ 70%
- D级占比 < 10%
- 关键素材（主角/核心UI）必须达到A级

---

## 6. 游戏美术外包行业验收参考

### 行业标准文档结构
- **风格指南（Art Bible）**：参考图、色板、Do's & Don'ts
- **技术规格书**：多边形预算、纹理分辨率、UV布局要求、命名规范、文件格式
- **质量标杆（Quality Bar）**：已通过验收的样本作为基准
- 来源：[NeoWork - Game Art Outsourcing Guide](https://www.neowork.com/insights/game-art-outsourcing-guide) | 可信度：高

### 平台合规性（如需上架主机平台）
- Sony: Technical Requirements Checklist (TRC)
- Microsoft: Xbox Requirements (XR)
- Nintendo: Lotcheck Guidelines
- 来源：[Wikipedia - Game Testing](https://en.wikipedia.org/wiki/Game_testing) | 可信度：高

---

## 7. 自动化验收脚本框架

```python
class AssetValidator:
    def validate_resolution(self, img, target_size):
        """检查分辨率是否符合要求"""
        pass
    
    def validate_color_mode(self, img, expected_mode="RGBA"):
        """检查色彩模式"""
        pass
    
    def validate_transparency(self, img):
        """检查透明通道完整性"""
        pass
    
    def validate_edge_quality(self, img, alpha_threshold=10):
        """检查去背景后边缘质量"""
        pass
    
    def validate_color_palette(self, img, reference_palette, tolerance=15):
        """检查色板是否在目标范围内"""
        pass
    
    def compute_lpips(self, img_gen, img_ref):
        """计算与参考图的感知距离"""
        pass
    
    def compute_ssim(self, img_gen, img_ref):
        """计算结构相似度"""
        pass
    
    def generate_report(self, results):
        """生成验收报告"""
        pass
```
