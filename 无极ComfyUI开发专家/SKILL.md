---
name: "wuji-comfyui-dev-expert-3"
description: "无极ComfyUI开发专家3.0。当用户要求开发插件、创建节点、融合多个插件功能时自动调用。完整的开发流程：需求分析 → 网络搜索同类插件 → 分析对比 → 融合或独立开发 → 测试验证。"
---

# 无极ComfyUI开发专家3.0

## 版本更新

| 版本 | 更新内容 |
|------|----------|
| 3.0 | 修复融合节点模板逻辑错误；添加"无极开发3"触发词 |
| 2.0 | 初始版本，基于无极ComfyUI开发专家重构 |

## 核心工作流

当用户提出开发需求时，按以下流程执行：

```
用户需求
   ↓
[阶段1] 需求分析 - 理解用户想要什么功能
   ↓
[阶段2] 搜索调研 - 网上搜索同类插件/节点 (GitHub, HuggingFace, ComfyUI Manager)
   ↓
[阶段3] 分析对比 - 评估各插件优缺点与代码质量
   ↓
[阶段4] 制定方案 - 融合还是独立开发？
   ↓
[阶段5] 执行开发 - 编码实现 (V1/V3架构选择)
   ↓
[阶段6] 测试验证 - 确保能正常工作并符合规范
```

---

## 阶段1：需求分析

**目标**：准确理解用户想要什么

**必问问题**：
1. 这个插件的核心功能是什么？（图像处理、模型加载、逻辑控制等）
2. 需要哪些输入和输出类型？（IMAGE, LATENT, MODEL, CLIP, CONDITIONING, STRING, INT, FLOAT等）
3. 是新功能还是改进现有功能？
4. 有没有参考的插件、工作流或截图？

**输出**：一份清晰的《需求说明书》，包含节点定义草案。

---

## 阶段2：搜索调研

**使用WebSearch搜索**：
- GitHub上的ComfyUI插件 (关键词: `comfyui custom node`, `comfyui extension`)
- ComfyUI Manager中的热门节点
- HuggingFace上的相关模型或工具
- 关键词：`功能名 + ComfyUI + custom node`

**搜索维度**：
| 维度 | 说明 |
|------|------|
| 功能匹配度 | 是否实现用户需要的功能 |
| 代码质量 | 是否清晰易懂，遵循最佳实践 |
| 维护状态 | 最近更新时间，Issue处理情况 |
| 用户评价 | Star数，下载量，社区反馈 |

**输出**：候选插件列表（含GitHub链接和主要功能摘要）。

---

## 阶段3：分析对比

**对每个候选插件进行深度分析**：

```python
# 分析维度
plugins_analysis = {
    "功能完整性": ["列出覆盖的功能点"],
    "特色功能": ["独有的竞争力功能"],
    "代码质量": ["可读性、注释、架构、类型提示"],
    "性能": ["速度、内存占用、批处理支持"],
    "稳定性": ["报错率、兼容性、边缘情况处理"],
    "维护状态": ["最后更新时间、Issue处理速度"],
}
```

**融合价值评估**：
- ✅ 高融合价值：代码独立、接口清晰、依赖官方API、MIT/Apache协议
- ⚠️ 中融合价值：有一定耦合但可提取、协议允许引用
- ❌ 低融合价值：强耦合、定制化程度高、协议限制、代码混乱

**输出**：《插件对比分析表》与推荐方案。

---

## 阶段4：制定方案

**决策树**：

```
用户需求
    │
    ├─→ 已有插件完全满足？ ──→ 直接使用，不开发
    │
    ├─→ 多个插件组合能满足？ ──→ 融合方案
    │     (保留原插件代码，只做引用封装)
    │
    ├─→ 部分功能满足？ ──→ 融合 + 新增开发
    │
    └─→ 没有类似插件？ ──→ 全新独立开发
```

**融合原则**：
```python
# ✅ 正确做法：直接引用，不复制代码
import comfy.sd
from nodes import LoraLoader

class MyFusionNode:
    def execute(self, ...):
        lora_loader = LoraLoader()  # 直接用官方节点
        model, clip = lora_loader.load_lora(...)
        return model, clip

# ❌ 错误做法：复制他人代码
# class MyFusionNode:
#     def load_lora(self, ...):  # 不要复制！
#         # 复制粘贴的代码...
```

**好处**：原插件更新 → 融合节点自动更新，减少维护成本。

---

## 阶段5：执行开发

### 5.1 确定项目结构

```
your-plugin/
├── __init__.py          # 插件入口，定义NODE_CLASS_MAPPINGS
├── nodes.py              # 节点实现（融合节点放这里）
├── nodes_custom.py       # 独立开发的新节点
├── utils.py              # 工具函数
├── web/
│   └── js/               # 前端增强（可选，如自定义UI组件）
├── docs/                 # 文档
│   ├── node1.md
│   └── node2.md
└── requirements.txt      # 额外依赖
```

### 5.2 节点开发模板

**V3架构模板（推荐新项目使用）**：
V3 API 提供了更严格的类型检查和更好的IDE支持。

```python
from comfy_api.latest import ComfyExtension, io, ui

class MyNode(io.ComfyNode):
    @classmethod
    def define_schema(cls) -> io.Schema:
        return io.Schema(
            node_id="YourPrefix_MyNode",
            display_name="你的节点名称",
            category="your_category",
            description="节点功能描述",
            inputs=[
                io.Image.Input("image"),
                io.Float.Input("strength", default=1.0, min=0.0, max=2.0),
            ],
            outputs=[io.Image.Output()],
        )

    @classmethod
    def execute(cls, image, strength) -> io.NodeOutput:
        result = image * strength
        return io.NodeOutput(result, ui=ui.PreviewImage(result, cls=cls))

class MyExtension(ComfyExtension):
    async def get_node_list(self) -> list[type[io.ComfyNode]]:
        return [MyNode]

async def comfy_entrypoint() -> MyExtension:
    return MyExtension()
```

**V1架构模板（兼容性、现有项目维护）**：
V1 是目前最广泛支持的架构，适合快速开发和兼容旧版ComfyUI。

```python
class MyNode:
    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "image": ("IMAGE",),
                "strength": ("FLOAT", {"default": 1.0, "min": 0.0, "max": 2.0}),
            },
            "optional": {
                "mask": ("MASK",),
            }
        }

    RETURN_TYPES = ("IMAGE",)
    RETURN_NAMES = ("processed_image",) # V1也支持命名输出
    FUNCTION = "execute"
    CATEGORY = "MyCategory/SubCategory"
    DESCRIPTION = "节点功能描述" # 可选，用于悬停提示

    def execute(self, image, strength, mask=None):
        # 处理逻辑
        if mask is not None:
            # 处理mask
            pass
        result = image * strength
        return (result,)

NODE_CLASS_MAPPINGS = {"MyNode": MyNode}
NODE_DISPLAY_NAME_MAPPINGS = {"MyNode": "我的节点"}
```

### 5.3 融合节点模板

```python
# 融合多个插件的节点
class WujiFusionNode:
    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "model": ("MODEL",),
                "clip": ("CLIP",),
            },
            "optional": {
                "lora_name": (["none", "sdxl_style_slider_v2.safetensors"],),
                "lora_strength": ("FLOAT", {"default": 1.0, "min": -10.0, "max": 10.0}),
            }
        }

    RETURN_TYPES = ("MODEL", "CLIP")
    RETURN_NAMES = ("model", "clip")
    FUNCTION = "execute"
    CATEGORY = "☯️无极/融合"
    DESCRIPTION = "融合LoRA加载与模型处理"

    def execute(self, model, clip, lora_name="none", lora_strength=1.0):
        # ✅ 正确做法：直接引用官方节点，不复制代码
        if lora_name != "none":
            from nodes import LoraLoader
            lora = LoraLoader()
            model, clip = lora.load_lora(model, clip, lora_name, lora_strength)

        return (model, clip)

NODE_CLASS_MAPPINGS = {"WujiFusionNode": WujiFusionNode}
NODE_DISPLAY_NAME_MAPPINGS = {"WujiFusionNode": "☯️无极融合节点"}
```

### 5.4 插件入口模板

```python
# __init__.py
WEB_DIRECTORY = "./web/js"
NODE_CLASS_MAPPINGS = {}
NODE_DISPLAY_NAME_MAPPINGS = {}
__all__ = ["NODE_CLASS_MAPPINGS", "NODE_DISPLAY_NAME_MAPPINGS", "WEB_DIRECTORY"]

# 导入并注册节点
from .nodes import (
    WujiFusionNode,
)
from .nodes_custom import (
    WujiCustomNode,
)

NODE_CLASS_MAPPINGS = {
    "WujiFusionNode": WujiFusionNode,
    "WujiCustomNode": WujiCustomNode,
}

NODE_DISPLAY_NAME_MAPPINGS = {
    "WujiFusionNode": "☯️无极融合节点",
    "WujiCustomNode": "☯️自定义节点",
}
```

### 5.5 开发最佳实践

1. **类型提示**：尽可能使用Python类型提示，提高代码可读性和IDE支持。
2. **错误处理**：在`execute`方法中添加适当的try-except块，提供友好的错误信息。
3. **进度条**：对于耗时操作，使用`comfy.utils.ProgressBar`更新进度。
4. **缓存**：对于重复计算的结果，考虑使用简单的缓存机制。
5. **日志**：使用`logging`模块记录调试信息，避免使用`print`。
6. **资源管理**：及时释放不再需要的GPU内存（如`del tensor`，`torch.cuda.empty_cache()`）。

---

## 阶段6：测试验证

### 6.1 开发时验证

完成每个节点后，自动检查：

```text
✅ 检查项：
   [1] 语法正确 - 运行 python -m py_compile nodes.py
   [2] 导入成功 - from your_plugin import nodes
   [3] NODE_CLASS_MAPPINGS 包含所有节点
   [4] INPUT_TYPES 返回正确格式的字典
   [5] RETURN_TYPES 是元组格式且数量与return语句匹配
   [6] 节点在ComfyUI中能正常显示
```

### 6.2 手动测试步骤

1. 重启ComfyUI
2. 在节点搜索栏搜索节点名称
3. 拖入画布
4. 连接输入输出
5. 运行工作流
6. 检查输出是否正确
7. 测试边缘情况（空输入、极端参数值等）

### 6.3 常见错误处理

| 错误 | 原因 | 解决方法 |
|------|------|----------|
| `KEYERROR` | RETURN_TYPES数量与实际输出不匹配 | 检查return语句返回的元组元素数量 |
| `图片黑色/全零` | 处理逻辑错误或数据类型不对 | 检查计算过程，确保输出范围正确（如0-1或0-255） |
| `找不到节点` | __init__.py没注册或文件名错误 | 检查NODE_CLASS_MAPPINGS和文件导入路径 |
| `类型不匹配` | 连接了错误类型的端口 | 检查INPUT_TYPES定义的类型与上游输出是否一致 |
| `CUDA Out of Memory` | 显存不足 | 优化内存使用，分批处理，或提示用户降低分辨率 |

---

## 开发决策参考

### 什么时候融合？

| 情况 | 推荐 |
|------|------|
| 功能分散在多个插件 | ✅ 融合 |
| 需要保持同步更新 | ✅ 融合 |
| 只需要部分功能 | ✅ 融合 + 裁剪 |
| 功能完全独立 | ❌ 独立开发 |
| 需要深度定制 | ❌ 独立开发 |

### V1 vs V3 选择

| 情况 | 推荐 |
|------|------|
| 新项目/新节点 | ✅ V3 (如果ComfyUI版本支持) |
| 现有V1项目维护 | V1继续 |
| 需要最新特性 | ✅ V3 |
| 需要兼容旧插件/旧版ComfyUI | V1 |

---

## 调用其他Skill/MCP

在开发过程中，可以调用：

- **WebSearch** - 搜索最新的ComfyUI插件和最佳实践
- **Read/Glob** - 分析现有插件代码结构
- **其他分析skill** - 辅助代码分析和优化

---

## 触发词

- "无极开发3"
- "开发一个ComfyUI插件"
- "做一个ComfyUI节点"
- "融合这几个ComfyUI插件"
- "帮我实现xxx ComfyUI功能"
- "创建一个ComfyUI custom node"
- "无极ComfyUI开发"

---

🌟 **无极ComfyUI开发专家3.0** - 让ComfyUI开发更简单，从需求到实现，一站式完成
