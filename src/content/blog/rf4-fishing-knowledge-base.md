---
title: "RF4 钓鱼知识库"
description: "自动爬取、解析俄罗斯钓鱼4点位数据的知识库系统，支持ADB自动化爬取和OCR识别"
pubDate: "Apr 05 2026"
heroImage: "/blog-placeholder-2.jpg"
---

## 项目简介

自动爬取、解析俄罗斯钓鱼4（Russian Fishing 4）点位数据的知识库系统。

## 项目状态

| 阶段 | 状态 | 说明 |
|------|:----:|------|
| 爬虫开发 | 完成 | v5.1 稳定版，100%成功率 |
| 数据采集 | 完成 | 40张 detail + 41张 gear |
| Detail OCR | 完成 | 图像去重，5.5秒/张 |
| Gear 解析 | 完成 | 拼接+MiniMax批处理 |
| 结构化存储 | 完成 | 16个有效点位 |
| 查询工具 | 完成 | 交互模式已上线 |

## 数据统计

| 指标 | 数值 |
|------|:----:|
| 有效点位 | **16个** |
| 鱼种数 | **13种** |
| 地图数 | **6张** |
| OCR去重 | -45% |
| Gear去重 | -39% |

## 目录结构

```
rf4_knowledge/
├── scripts/              # 核心脚本
│   ├── pipeline.py       # 主流程入口
│   ├── crawler.py        # 爬虫
│   ├── query.py          # 查询工具
│   ├── process_all.py    # OCR + 解析
│   └── merge_gear_results.py
│
├── data/
│   ├── crawl/            # 爬虫原始截图
│   ├── processed/        # 处理后数据
│   └── reference/        # 参考数据
│       ├── fish/         # 鱼种信息（200+）
│       ├── maps/         # 地图信息
│       └── fishing_spots/
│
├── docs/                 # 详细文档
└── logs/                 # 运行日志
```

## 快速开始

```bash
# 完整流程：爬虫 → 处理 → 合并
python3 scripts/pipeline.py --count 10

# 查询数据（交互模式）
python3 scripts/query.py --stats -i

# 仅处理已有数据
python3 scripts/pipeline.py --skip-crawl
```

## 查询用法

```bash
# 统计信息
python3 scripts/query.py --stats

# 交互模式（输入序号查看详情）
python3 scripts/query.py --stats -i

# 按鱼种查询
python3 scripts/query.py --fish 鲤鲫

# 按地图查询
python3 scripts/query.py --map "旧奥斯特罗格湖"
```

## 技术栈

| 组件 | 技术 | 说明 |
|------|------|------|
| 爬虫 | Python + ADB | 状态机 + 模板匹配 |
| Detail OCR | EasyOCR + imagehash | 哈希去重，5.5s/张 |
| Gear 解析 | MiniMax 视觉 | 拼接批处理，~30s |
| 存储 | JSON | 后续迁移 SQLite |

## 更新日志

### 2026-04-05
- 项目结构优化
- 查询工具交互模式
- 流水线脚本 `pipeline.py`
- 图像去重优化（-45%）

### 2026-04-04
- 爬虫 v5.1 稳定版
- 模板匹配替代OCR分类
- 智能等待机制
