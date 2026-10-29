# 足球比赛分析：特征工程

运行[02_features.ipynb](notebooks/02_features.ipynb)，从上到下执行即可生成最终的[cleaned.csv](data/processed/cleaned.csv)。全部处理代码都在Notebook内；本步骤不训练预测模型。

## 当前流程

1. 删除七个核心字段任一项缺失或无效的比赛。
2. 保留已有Elo；缺失的一侧从评分表查找同一国家和球队、严格早于比赛日的最近评分，没有时长上限。
3. 补全后任意一方Elo仍缺失，就删除该场。
4. 按此前有效比赛重算四个Form字段。历史池使用全部比分和赛果有效的比赛，完整窗口不足时留空并记录实际场数。
5. 保存补全缺失值.csv中间表及处理明细。
6. 排除任意一方最近5场不足，以及任意一项普通赔率缺失、非有限数或不大于1的比赛。
7. 删除指定的17列暂不使用字段和6列填补记录，生成cleaned.csv。

## 当前结果

原始230,557场，核心信息排除3场，补全后仍缺Elo排除92,951场。在补全缺失值.csv基础上，再排除最近5场不足的2,101场，以及其余赔率无效的1,990场。最终cleaned保留133,512场、46列；双方Elo、四个Form和三项普通赔率均完整，赔率全部大于1。未使用的技术统计和评分日期仍可为空。

[cleaned字段说明](data/processed/cleaned.README.md)列出删除与保留字段。原始两份CSV不变，来源、删除原因和检查结果在reports/feature_engineering/。

46列包含身份、目标和检查信息，不代表模型需要输入46个特征。

## 运行

使用.venv虚拟环境和AML Football (.venv)内核。从项目根目录启动：

```bash
.venv/bin/jupyter lab
```

依赖在requirements.txt，原始数据在data/raw/，输出在data/processed/。

## 第一版 baseline

运行[base_line.ipynb](notebooks/base_line.ipynb)：近期固定5场，历史交锋使用此前全部记录，比较M0、M1、M2、M3四个逻辑回归模型，并分析与赔率的关系。所有代码均在Notebook内。

2000—2020训练，2021—2022验证，2023—2024测试；2025暂不进入本版。验证仅选择正则化强度，不选择近期窗口。

结果保存在[reports/baseline/RESULTS.zh-CN.md](reports/baseline/RESULTS.zh-CN.md)，完整输入清单、验证分数、逐场测试概率、模型和检查记录位于reports/baseline/。

## 时间衰减与LSTM多任务扩展

按顺序运行：

1. [time_decay.ipynb](notebooks/time_decay.ipynb)：近期5场、交锋全部历史，分别按90/365天半衰期计算加权统计，训练逻辑回归。输出 `time_decay_features.csv` 和 `history_sequences.npz`。
2. [lstm_multitask.ipynb](notebooks/lstm_multitask.ipynb)：读取相同逐场历史，联合训练LSTM和预测层；比较单任务与硬参数共享多任务。主任务预测赛果，辅助任务拟合赔率概率，赔率不进入网络输入。验证集选择辅助权重和训练轮数，三个固定seed重训。

两者沿用baseline的训练/验证/测试划分与比赛样本。所有代码在Notebook内，旧数据和baseline保留。

结果分别在 [时间衰减报告](reports/time_decay/RESULTS.zh-CN.md) 和 [LSTM报告](reports/lstm_multitask/RESULTS.zh-CN.md)。各目录同时保存参数、输入说明、逐场预测、配对比较和模型。

LSTM使用PyTorch 2.11.0，当前环境安装CUDA 12.8版本，自动选择GPU或CPU。重新安装依赖可运行 `.venv/bin/python -m pip install -r requirements.txt`；无NVIDIA显卡的机器也可以按PyTorch官方说明改装CPU版本。

## 从 GitHub 克隆后运行

先按 [原始数据说明](data/raw/README.md) 下载两份固定版本的 CSV，再创建环境：

```bash
python3.12 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m ipykernel install --sys-prefix --name aml-football --display-name "AML Football (.venv)"
.venv/bin/jupyter lab
```

依次完整运行 `02_features.ipynb`、`base_line.ipynb`、`time_decay.ipynb`、`lstm_multitask.ipynb`。已执行的 Notebook 和汇总结果随代码保存；原始/处理后 CSV、历史缓存、逐场预测、模型权重和虚拟环境由 `.gitignore` 排除，在本地按此顺序重新生成。

## 模型解释

前四本Notebook完成后，运行[post_analysis.ipynb](notebooks/post_analysis.ipynb)，读取已保存模型：导出22个逻辑回归模型的完整系数，展示历史比赛的固定时间权重，并整理近期/交锋输入组的贡献及多任务损失权重。该步骤不重新训练模型。

解释见[reports/post_analysis/RESULTS.zh-CN.md](reports/post_analysis/RESULTS.zh-CN.md)。系数、时间权重和损失权重含义不同，均不能直接当作特征重要性百分比。
