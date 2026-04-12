# 维度4：批量生产Pipeline

## 研究目标
设计从AI批量生成到引擎可用资源的完整Pipeline，覆盖ComfyUI工作流、自动化脚本、后处理、打包导出全流程。

---

## 1. ComfyUI工作流设计

### API驱动的批量生成
- ComfyUI本质是一个HTTP服务器，UI点击"Queue Prompt"等同于向`http://127.0.0.1:8188/prompt`发送POST请求
- 所有UI操作都可通过API编程完成
- 工作流可导出为API格式JSON（开启Dev Mode后点击"Save (API Format)"）
- 队列系统使用FIFO（先进先出）模型
- 来源：[Xentoo Blog - ComfyUI Batch Run](https://blog.xentoo.info/2023/07/22/comfyui-batch-run-from-command-line-with-api/) | 可信度：高（一手实践）

### Python批量脚本核心逻辑
```python
import json, requests

def queue_prompt(workflow, server="http://127.0.0.1:8188"):
    response = requests.post(f"{server}/prompt", json={"prompt": workflow})
    return response.json()

# 批量生成：遍历素材列表，修改workflow中的参数，逐个入队
for asset in asset_list:
    workflow["6"]["inputs"]["text"] = asset["prompt"]
    workflow["3"]["inputs"]["seed"] = asset["seed"]
    queue_prompt(workflow)
```

### 错误处理要点
- **VRAM OOM**：自动重启或降低分辨率重试
- **服务器崩溃**：心跳检测+自动重启ComfyUI进程
- **网络超时**（远程实例）：重试机制+指数退避
- **生成中断导致的损坏输出**：验证输出文件完整性
- 来源：[Apatero - ComfyUI Batch Processing](https://apatero.com/blog/comfyui-batch-processing-1000-images-automation-2026) | 可信度：高

### WebSocket实时监控
- 通过WebSocket连接跟踪生成进度
- 可实现实时反馈和进度显示

---

## 2. 自动去背景

### rembg（推荐方案）
- 基于U-2-Net机器学习模型的开源去背景工具
- 支持CLI、Python库、HTTP服务器、Docker四种使用方式
- 全离线处理，图片不上传云端
- 处理速度：每张2-3秒
- 支持Python 3.10-3.13
- 来源：[GitHub - rembg](https://github.com/danielgatis/rembg) | 可信度：极高（官方仓库）

### 批量处理代码
```python
from rembg import remove, new_session
from pathlib import Path
from PIL import Image

session = new_session()  # 复用session提升性能
for img_path in Path("./input").glob("*.png"):
    input_img = Image.open(img_path)
    output_img = remove(input_img, session=session)
    output_img.save(f"./output/{img_path.name}")
```

### ComfyUI去背景节点
- **ComfyUI-RMBG**：支持RMBG-2.0、INSPYRENET、BEN、BEN2、BiRefNet等多种模型
- **Batch Rembg**：专门的批量处理节点，可在工作流中直接集成
- 来源：[RunComfy - Batch Rembg](https://www.runcomfy.com/comfyui-nodes/batchImg-rembg-ComfyUI-nodes) | 可信度：高

---

## 3. 尺寸标准化与色调校正

### 尺寸标准化
- 按游戏引擎要求统一分辨率（通常为2的幂次：64, 128, 256, 512, 1024）
- 保持宽高比，用padding或裁切适配
- 使用Lanczos重采样保持质量

### 色调校正
- 批量颜色校正确保色板一致性
- 使用直方图匹配（Histogram Matching）对齐到参考色调
- Python OpenCV: `cv2.createCLAHE()` 做自适应直方图均衡
- 色温微调：通过HSL空间的H通道偏移统一色温

---

## 4. TexturePacker图集打包

### TexturePacker核心功能
- 将多个散图自动排布为sprite sheet（精灵图集）
- 减少游戏包体大小和内存占用
- 同一图集的精灵可合并为同一渲染批次，显著降低CPU开销
- 来源：[TexturePacker官方文档](https://www.codeandweb.com/texturepacker/documentation) | 可信度：极高（官方文档）

### 引擎导出配置

#### Cocos Creator
- 数据格式选择：**cocos2d-x**（.plist + .png）
- TexturePacker使用v4.x（不兼容v3.x及更早）
- 将.plist和.png文件同时拖入Assets面板
- Cocos Creator也支持Auto Atlas（自动图集）功能
- 来源：[Cocos Creator Docs - Atlas](https://docs.cocos.com/creator/3.8/manual/en/asset/atlas.html) | 可信度：极高（官方文档）

#### Unity
- 数据格式选择：**Unity - Texture2D**
- 安装TexturePacker Importer插件（免费）
- Data file路径指向Assets目录
- 每次发布更新时自动重新导入
- Unity也自带Sprite Atlas打包功能
- 来源：[TexturePacker - Unity Tutorial](https://www.codeandweb.com/texturepacker/tutorials/using-spritesheets-with-unity) | 可信度：极高（官方教程）

---

## 5. 完整Pipeline流程

```
阶段1: 素材准备
  ├── 收集原版游戏素材
  ├── 分类整理（角色/道具/场景/UI/特效）
  └── 记录每类素材的规格要求（尺寸/格式/命名）

阶段2: 风格定义
  ├── 风格分析（维度1的方法）
  ├── 准备风格参考图
  ├── 编写提示词模板（维度2的模板）
  └── 选择一致性控制方案（维度3的决策树）

阶段3: ComfyUI工作流搭建
  ├── 构建基础生成工作流
  ├── 集成ControlNet/IP-Adapter节点
  ├── 添加去背景后处理节点
  └── 导出API格式JSON

阶段4: 批量生成
  ├── Python脚本批量入队
  ├── WebSocket监控进度
  ├── 错误处理与重试
  └── 输出文件完整性验证

阶段5: 后处理
  ├── 去背景（rembg/ComfyUI-RMBG）
  ├── 尺寸标准化（Pillow/OpenCV）
  ├── 色调校正（直方图匹配）
  └── 人工质检筛选

阶段6: 打包导出
  ├── TexturePacker图集打包
  ├── 按引擎格式导出（Cocos/Unity）
  ├── 命名规范化
  └── 资源清单生成
```

---

## 6. 工具链速查

| 环节 | 工具 | 用途 |
|------|------|------|
| 生成 | ComfyUI | 核心AI生成引擎 |
| 批量化 | Python + ComfyUI API | 自动化批量入队 |
| 去背景 | rembg / ComfyUI-RMBG | 透明背景处理 |
| 图像处理 | Pillow / OpenCV | 尺寸/色调/格式转换 |
| 超分辨率 | Real-ESRGAN | 放大低分辨率输出 |
| 图集打包 | TexturePacker / ShoeBox | 精灵图集生成 |
| PBR材质 | ComfyUI-TextureAlchemy | PBR贴图集生成 |
| 3D辅助 | Blender（自动化pipeline） | 重拓扑/UV/烘焙/图集 |
