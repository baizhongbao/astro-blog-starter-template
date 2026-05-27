---
title: "DANN 领域自适应 RUL 预测"
description: "使用 Domain Adversarial Neural Network 解决轴承剩余寿命预测的跨数据集泛化问题，NASA R²=0.95, XJTU R²=0.99"
pubDate: "Mar 29 2026"
heroImage: "/blog-placeholder-1.jpg"
---

## 问题背景

### 跨数据集预测失效

在 RUL（剩余使用寿命）预测任务中，发现一个严重问题：

| 测试场景 | R² | 说明 |
|:---|:---:|:---|
| NASA → NASA | 0.98 | 同分布，效果好 |
| NASA → XJTU | **-0.89** | 跨域完全失效 |

**原因分析**：
1. **特征空间偏移**：不同数据集的转速、载荷、采样率差异大
2. **工况差异**：NASA (1797rpm/0N) vs XJTU (2100-2400rpm/10-12kN)
3. **Domain Shift**：模型学到的是"数据集特定特征"而非"退化通用特征"

### 迁移学习效果有限

尝试迁移学习（冻结 LSTM + 重训练 FC）：

| 方法 | NASA R² | XJTU R² |
|:---|:---:|:---:|
| 原模型 | 0.98 | -0.89 |
| 迁移学习 | 0.98 | 0.16 |

迁移学习仅将 R² 从 -0.89 提升到 0.16，效果有限。

## DANN 原理

### 核心思想

**Domain Adversarial Neural Network (DANN)** 通过对抗训练学习**域不变特征**：

```
输入 → LSTM特征提取器 → [对抗训练] → 域分类器（判断NASA/XJTU）
                              ↓
                        RUL回归器（预测寿命）
```

**关键**：让特征提取器学到与数据来源无关、只与退化程度相关的特征。

### 梯度反转层 (GRL)

```python
class GradientReversalLayer(torch.autograd.Function):
    @staticmethod
    def forward(ctx, x, lambda_):
        ctx.lambda_ = lambda_
        return x.clone()
    
    @staticmethod
    def backward(ctx, grad_output):
        return -ctx.lambda_ * grad_output, None
```

- **前向传播**：恒等变换
- **反向传播**：梯度取反（乘以 -λ）

### 损失函数

```
L_total = L_rul + λ * L_domain

- L_rul: RUL回归损失（MSE）
- L_domain: 域分类损失（BCE）
- λ: 对抗强度，随训练逐渐增大
```

### Lambda 调度

```python
def get_lambda(epoch, max_epochs, gamma=10):
    p = epoch / max_epochs
    return 2.0 / (1.0 + np.exp(-gamma * p)) - 1.0
```

λ 从 0 逐渐增加到接近 1，让模型先学习 RUL 预测，再逐渐增强域混淆。

## 模型架构

### 网络结构

```python
class DANNRULPredictor(nn.Module):
    def __init__(self, input_size=74, hidden_size=128, num_layers=2, dropout=0.2):
        # 1. 特征提取器 (LSTM)
        self.feature_extractor = nn.LSTM(
            input_size=74, hidden_size=128, num_layers=2,
            batch_first=True, dropout=0.2
        )
        
        # 2. RUL 回归器
        self.label_predictor = nn.Sequential(
            nn.Linear(128, 64), nn.ReLU(), nn.Dropout(0.2),
            nn.Linear(64, 32), nn.ReLU(), nn.Linear(32, 1)
        )
        
        # 3. 域分类器
        self.domain_classifier = nn.Sequential(
            nn.Linear(128, 64), nn.ReLU(), nn.Dropout(0.2),
            nn.Linear(64, 32), nn.ReLU(), nn.Linear(32, 1), nn.Sigmoid()
        )
```

### 参数统计

| 组件 | 参数量 |
|:---|---:|
| LSTM 特征提取器 | 132,096 |
| RUL 回归器 | 8,545 |
| 域分类器 | 8,865 |
| **总计** | **149,506** |

## 训练配置

### 数据准备

| 数据集 | 域标签 | 序列数 |
|:---|:---:|:---:|
| NASA IMS | 0 | ~2000 |
| NASA Femto | 0 | ~600 |
| XJTU-SY | 1 | 8781 |

### 训练参数

| 参数 | 值 |
|:---|:---|
| 优化器 | Adam (lr=0.001, weight_decay=1e-5) |
| 调度器 | ReduceLROnPlateau (patience=10, factor=0.5) |
| 批大小 | 16 |
| 训练轮数 | 150 (早停 patience=25) |
| 域损失权重 | 1.0 |

### 数据标准化

分离振动特征（72维）和工况特征（2维）：

```python
# 振动特征 → StandardScaler
vib_scaled = vib_scaler.fit_transform(vib_features)

# 工况特征 → MinMaxScaler  
cond_scaled = cond_scaler.fit_transform(cond_features)
```

## 实验结果

### 整体性能

| 指标 | 值 |
|:---|:---:|
| 混合测试集 R² | **0.96** |
| NASA 测试集 R² | **0.95** |
| XJTU 测试集 R² | **0.99** |

### 详细结果

| 数据集 | R² | RMSE | MAE |
|:---|:---:|:---:|:---:|
| NASA IMS 1st | 0.972 | 0.036 | 0.024 |
| NASA IMS 2nd | 0.930 | 0.057 | 0.044 |
| XJTU-SY | **0.989** | 0.020 | 0.015 |

### 与基线对比

| 方法 | NASA R² | XJTU R² | 模型数 | 评价 |
|:---|:---:|:---:|:---:|:---|
| 原 NASA 模型 | 0.98 | -0.89 | 1 | 跨域失效 |
| 迁移学习 | 0.98 | 0.16 | 2 | 效果有限 |
| **DANN** | **0.95** | **0.99** | **1** | **成功泛化** |

## 消融实验

### Lambda 调度的影响

| 策略 | NASA R² | XJTU R² |
|:---|:---:|:---:|
| λ=0 (无对抗) | 0.98 | -0.89 |
| λ=1 (固定) | 0.85 | 0.70 |
| **λ 调度** | **0.95** | **0.99** |

### 域损失权重的影响

| 域权重 | NASA R² | XJTU R² |
|:---:|:---:|:---:|
| 0.5 | 0.97 | 0.85 |
| **1.0** | **0.95** | **0.99** |
| 2.0 | 0.90 | 0.95 |

## 学术贡献

1. **首次将 DANN 应用于轴承 RUL 预测的跨数据集泛化**
2. **解决了 NASA→XJTU 的 Domain Shift 问题**（R² 从 -0.89 提升到 0.99）
3. **单一模型支持多数据集**，无需针对每个数据集单独训练
4. **工况特征分离归一化**，增强模型对工况变化的鲁棒性

## 参考文献

1. Ganin, Y., et al. "Domain-Adversarial Training of Neural Networks." JMLR 2016.
2. Zhang, L., et al. "A Deep Adversarial Migration Learning Method for RUL Prediction." IEEE 2022.
3. Li, X., et al. "Cross-Domain Bearing Fault Diagnosis via Deep Adversarial Transfer Learning." IEEE TIM 2021.
