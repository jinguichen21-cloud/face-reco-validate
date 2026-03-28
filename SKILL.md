---
name: face-reco-validate
description: >
  人脸识别校验。对用户传入的图片URL进行校验，检查是否符合人脸识别数据入库的规范要求。
  支持检测图片中是否存在人脸、人脸质量是否达标（清晰度、光照、姿态等）。
  每当用户需要"校验"、"检查"图片是否可用于人脸识别数据入库时，都应触发本技能。
metadata:
  label: 人脸识别校验
---

# 人脸识别校验

## 概述

对用户传入的图片URL进行校验，检查是否符合人脸识别数据入库的规范要求。

## MCP 服务配置

本技能依赖以下 MCP 服务，运行时**必须**先读取 `mcp-config.json` 获取服务 URL：

| 环境变量名 | MCP 服务 | mcpId |
|------------|----------|-------|
| `$VISION_AI_URL` | 视觉AI | 9724 |

### 配置读取方式

**约束：调用方 agent 必须在执行任何 MCP 调用前，先读取 mcp-config.json 文件**

```python
# 示例：读取 mcp-config.json 获取 MCP URL
import json

with open("mcp-config.json", "r", encoding="utf-8") as f:
    config = json.load(f)

# 获取视觉AI服务的 URL
vision_ai_url = config["VISION_AI_URL"]["url"]
```

## 工作流程

### 1. 读取配置

首先读取 `mcp-config.json` 获取视觉AI服务的 URL：

```python
import json

with open("mcp-config.json", "r", encoding="utf-8") as f:
    config = json.load(f)

vision_ai_url = config["VISION_AI_URL"]["url"]
if not vision_ai_url:
    print("错误：视觉AI服务URL未配置")
    exit(1)
```

### 2. 获取图片URL

从用户输入中提取待校验的图片URL。

### 3. 调用人脸检测

使用视觉AI服务检测图片中的人脸质量：

```bash
# 先查看视觉AI服务提供的工具
python3 scripts/call_mcp.py list "$VISION_AI_URL"

# 调用人脸检测/质量校验工具
python3 scripts/call_mcp.py call "$VISION_AI_URL" validate_face --params '{"image_url": "用户提供的图片URL"}'
```

### 4. 处理校验结果

根据视觉AI返回的结果，判断图片是否符合人脸识别入库规范：

- **通过**：人脸质量合格，可用于人脸识别数据入库
- **不通过**：返回具体原因（如：无人脸、人脸模糊、光线不足、姿态不正等）

### 5. 输出校验报告

向用户返回清晰的校验结果：

```
✅ 校验通过：该图片符合人脸识别数据入库规范

或

❌ 校验不通过：
   - 原因1：图片中未检测到人脸
   - 原因2：人脸清晰度不足
   - 建议：请重新上传符合要求的图片
```

## 使用示例

### 示例1：校验单张人脸图片

用户："帮我检查一下这张图片是否能用于人脸识别入库 https://example.com/photo.jpg"

执行流程：
1. 读取 `mcp-config.json` 获取 `$VISION_AI_URL`
2. 调用视觉AI服务检测图片
3. 返回校验结果

### 示例2：批量校验多张图片

用户："校验这几张图片是否都符合人脸规范"

执行流程：
1. 逐一获取每张图片的URL
2. 循环调用视觉AI服务进行检测
3. 汇总输出所有图片的校验结果

## 参考文档

| 文档 | 说明 | 何时阅读 |
|------|------|----------|
| [mcp-config.json](mcp-config.json) | MCP 服务配置模板 | 查看本技能依赖的 MCP 服务 |
