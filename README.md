# 足球比赛分析

研究近期状态与历史交锋对胜平负预测的帮助，以及它们与赔率的关系。近期固定最近5场，交锋使用此前全部记录。全部代码放在Jupyter Notebook中。

## 安装与数据

按[原始数据说明](data/raw/README.md)下载 `Matches.csv` 和 `EloRatings.csv`，放到 `data/raw/`。

```bash
python3.12 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m ipykernel install --sys-prefix --name aml-football --display-name "AML Football (.venv)"
.venv/bin/jupyter lab
```

LSTM使用PyTorch，自动选择GPU或CPU。当前依赖安装CUDA 12.8版本；也可以按PyTorch官方说明安装CPU版本。

## 按顺序运行

| Notebook | 做什么 | 主要输出 |
|---|---|---|
| [02_features.ipynb](notebooks/02_features.ipynb) | 删除核心信息无效的比赛、补全Elo、重算Form、筛选共同样本 | `cleaned.csv` |
| [base_line.ipynb](notebooks/base_line.ipynb) | 普通统计＋逻辑回归，比较近期、交锋和赔率 | `baseline_features.csv`、模型和测试结果 |
| [time_decay.ipynb](notebooks/time_decay.ipynb) | 时间衰减统计＋逻辑回归，并建立逐场历史序列 | `time_decay_features.csv`、`history_sequences.npz`、模型和测试结果 |
| [lstm_multitask.ipynb](notebooks/lstm_multitask.ipynb) | 对比单任务LSTM和硬参数共享多任务模型 | 模型、逐场测试概率、各seed分数和最终训练损失 |
| [post_analysis.ipynb](notebooks/post_analysis.ipynb) | 解释逻辑回归系数、时间权重、历史信息贡献和多任务λ | 系数表、贡献表和解释报告 |

每本Notebook从上到下完整运行。前一步的特征、历史序列、预测和模型会被后一步读取，这些文件需要保留。

## 当前数据处理规则

- 核心身份、日期或赛果信息缺失/无效的比赛删除。
- 已有Elo保留；缺失的一侧从评分表查找严格早于比赛日的最近评分，没有时长上限；仍缺Elo的比赛删除。
- 用全部有效赛果重算Form，避免因Elo或赔率筛选而丢失已知历史。
- 预测样本必须双方最近5场完整，三项普通赔率都为有限数且大于1。
- 删除此前指定的23列，直接生成[cleaned.csv](data/processed/cleaned.csv)。字段说明见[cleaned.README.md](data/processed/cleaned.README.md)。

当前固定数据下，原始230,557场，最终cleaned为133,512场、46列。

## 训练与结果

2000—2020训练，2021—2022验证选参数，2023—2024测试，2025暂不参与。验证集选择C、λ及训练轮数的步骤保留；模型固定后再评价测试集。历史只使用严格早于比赛日的赛果。

- [普通逻辑回归报告](reports/baseline/RESULTS.zh-CN.md)
- [时间衰减报告](reports/time_decay/RESULTS.zh-CN.md)
- [LSTM报告](reports/lstm_multitask/RESULTS.zh-CN.md)
- [模型解释报告](reports/post_analysis/RESULTS.zh-CN.md)

结果目录按实验保存最终分数、逐年结果、配对比较、赔率相关性、输入清单和模型。已移除人工测试、重复抽样核查、梯度演示、模型重载复核、哈希检查及检查报告；不再导出补全缺失值中间表和排除/填补明细。

原始/处理后CSV、历史缓存、逐场预测、模型权重和虚拟环境不提交Git，按上述顺序在本地生成。Notebook保留主要结果，减少过程输出。
