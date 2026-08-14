# Vision API SOP

## ⚠️ 前置规则（必须遵守）

1. **先枚举窗口**：调用 vision 前必须先用 `pygetwindow` 枚举窗口标题，确认目标窗口存在且已激活到前台。窗口不存在就不要截图。
2. **🚫 禁止全屏截图**：必须先利用ljqCtrl截取窗口区域。能截局部（如标题栏）就不截整窗口，能截窗口就绝不全屏。全屏截图在任何场景下都不允许。
3. **能不用 vision 就不用**：如果窗口标题/本地 OCR（`ocr_utils.py`）能获取所需信息，就不要调用 vision API，省 token 且更可靠。Vision 是最后手段。

## 快速用法

```python
from memory.vision_api import ask_vision
result = ask_vision(image, prompt="描述图片内容", timeout=60, max_pixels=1_440_000)
# image: 文件路径(str/Path) 或 PIL.Image
# backend仅支持已验证的 'modelscope'（默认）
# 返回str：成功为模型回复，失败为 'Error: ...'
```

## 已验证的 ModelScope 实现

- 接口：`POST https://api-inference.modelscope.cn/v1/chat/completions`
- 模型：`Qwen/Qwen3-VL-235B-A22B-Instruct`
- 协议：OpenAI-compatible；图片作为 JPEG data URL 放入 `image_url`
- Token读取顺序：`MODELSCOPE_API_KEY`代码配置 → 同名环境变量 → `memory.keychain` 的 `modelscope_token`
- Token禁止写入SOP、日志或L1；推荐存入keychain。
- 免费接口可能返回HTTP 429；不要高频重试，间隔后再调用。

## 验收

用含已知文字的最小PIL图片调用 `ask_vision`，确认返回文字匹配；同时验证文件路径输入和超像素缩放。
