---
title: "PDF 批量处理器"
description: "一键批量处理 PDF 文件，提取文本和图像，输出 Markdown 格式，支持自动 OCR 检测"
pubDate: "Apr 12 2026"
heroImage: "/blog-placeholder-5.jpg"
---

## 项目简介

一键批量处理 PDF 文件，提取文本和图像，输出 Markdown 格式。

## 特性

- **一键批量处理**：放入 PDF，双击运行，自动处理
- **自动 OCR 检测**：智能判断每页是否需要 OCR
- **图像提取**：自动提取并压缩图片，保存为独立文件
- **表格识别**：使用 PaddleOCR PPStructure
- **中英文支持**

## 已知问题

- **公式识别不支持**：PaddleOCR 不支持数学公式，公式会变成乱码
- **公式类文档**：建议使用 MiniMax Vision MCP 或其他专用工具

## 安装依赖

```bash
# 创建 conda 环境
conda create -n ocr python=3.10
conda activate ocr

# 安装依赖
pip install pdf_oxide paddleocr PyMuPDF Pillow

# GPU 加速（可选，需要 CUDA）
pip install paddlepaddle-gpu -i https://www.paddlepaddle.org.cn/packages/stable/cu123/
```

## 使用方法

### 批量处理（推荐）

1. 将 PDF 文件放入 `input/` 文件夹
2. 双击 `run.bat`
3. 结果在 `output/`，原 PDF 移至 `archive/`

```
input/*.pdf → 处理 → output/*/
                      ├── *.md
                      └── images/
```

### 单文件处理

```bash
conda activate ocr
python pdf_extractor.py document.pdf --embed-images
```

## 目录结构

```
project/
├── run.bat              # 一键运行脚本
├── batch_process.py     # 批量处理程序
├── pdf_extractor.py     # 单文件处理程序
├── input/               # 放入待处理 PDF
├── output/              # 输出结果
│   └── paper_name/
│       ├── paper_name.md
│       └── images/
├── archive/             # 处理完的 PDF
└── README.md
```

## 输出说明

每个 PDF 生成一个文件夹：

```
output/
├── paper1/
│   ├── paper1.md        # 提取的文本
│   └── images/          # 提取的图片
│       ├── page1_img1.png
│       ├── page2_img1.jpeg
│       └── ...
└── paper2/
    └── ...
```

**图片处理：**
- 自动压缩，最大宽度 1200px
- JPEG 质量 70%
- 放在页面末尾

## 自动检测逻辑

每页评分规则：

| 条件 | 分数 |
|------|------|
| 文本质量低 (< 0.3) | +3 |
| 检测到表格 | +2 |
| 检测到图像 | +1 |

- 分数 ≥ 2 → 使用 OCR
- 分数 < 2 → 直接提取文本

## 单文件处理参数

| 参数 | 说明 |
|------|------|
| `input` | 输入 PDF 文件（必需） |
| `-o, --output` | 输出文件前缀 |
| `--mode` | 模式：`auto`（默认）、`basic`、`ocr` |
| `--pages` | 指定 OCR 页码（从 1 开始） |
| `--embed-images` | 提取文本并嵌入图像 |
| `--extract-images` | 仅提取图像 |

## 适用场景

| 场景 | 效果 | 说明 |
|------|------|------|
| 纯文字 PDF | 极佳 | 秒速处理 |
| 数据表格（无公式） | 很好 | 自动识别表格 |
| 扫描件 PDF | 好 | 自动 OCR |
| 学术论文（含图表） | 很好 | 图片独立保存 |
| 学术论文（含公式） | 一般 | 公式需其他工具 |
