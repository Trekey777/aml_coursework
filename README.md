# 足球比赛分析：特征工程

运行[preprocess.ipynb](notebooks/preprocess.ipynb)，从上到下执行即可生成最终的[cleaned.csv](data/processed/cleaned.csv)。全部处理代码都在Notebook内；本步骤不训练预测模型。

## 当前流程

1. 读取 `Matches.csv` 与 `EloRatings.csv`，计算各列缺失率。
2. 删除 `HomeShots` 到 `C_PHB` 以及 `MatchTime`。这些列缺失过多，本项目不使用。
3. 删除比分或赛果缺失、赛果与比分不一致、或主客队相同的比赛。
4. 已有 Elo 不覆盖。缺失的一侧按同一国家、同一球队，取严格早于比赛日的最近评分。补全后仍缺任一侧的比赛删除。
5. 用全部有效赛果重算赛前 Form。双方近 5 场必须完整，任一不足则删除该场。近 3 场不足时对应 Form3 留空。
6. `OddHome`、`OddDraw`、`OddAway` 必须是大于 1 的有限数，否则删除该场。
7. 写出 `data/processed/cleaned.csv`。

## 当前结果

原始 238,858 场。比分与赛果无效删除 4 场。在双方都有 Elo 的比赛里，近 5 场不完整删除 2,170 场，赔率缺失或不大于 1 删除 2,000 场。`cleaned.csv` 保留 141,733 场、21 列：身份与联赛、双方 Elo、Form3/Form5、全场与半场比分和赛果、三项普通赔率、国家。双方 Elo、Form5 和三项赔率完整。半场比分和 Form3 仍可为空。

原始两份 CSV 不变。补全前后的缺失对比在 `reports/feature_engineering/completion_missing_before_after.csv`。这 21 列是清洗后的比赛表，不是模型输入清单。

## 运行

使用 `.venv` 虚拟环境和 AML Football (.venv) 内核。从项目根目录启动。

Windows：

```powershell
.\.venv\Scripts\jupyter.exe lab
```

Linux 或 macOS：

```bash
.venv/bin/jupyter lab
```

依赖在requirements.txt，原始数据在data/raw/，输出在data/processed/。

## 第一版 baseline

运行[baseline.ipynb](notebooks/baseline.ipynb)：近期固定5场，历史交锋使用此前全部记录，比较M0、M1、M2、M3四个逻辑回归模型，并分析与赔率的关系。所有代码均在Notebook内。

2000—2020训练，2021—2022验证，2023—2024测试；2025暂不进入本版。验证仅选择正则化强度，不选择近期窗口。

结果分析在[reports/baseline/baseline.md](reports/baseline/baseline.md)。测试分数、配对区间、Spearman 相关和比较图位于 reports/baseline/；后续笔记本读取的特征、逐场概率和模型也在同一目录。

## 时间衰减与LSTM多任务扩展

按顺序运行：

1. [time_decay.ipynb](notebooks/time_decay.ipynb)：近期5场、交锋全部历史，分别按90/365天半衰期计算加权统计，训练逻辑回归。输出 `time_decay_features.csv` 和 `history_sequences.npz`。
2. [lstm_multitask.ipynb](notebooks/lstm_multitask.ipynb)：读取相同逐场历史，联合训练LSTM和预测层；比较单任务与硬参数共享多任务。主任务预测赛果，辅助任务拟合赔率概率，赔率不进入网络输入。验证集选择辅助权重和训练轮数，三个固定seed重训。

两者沿用baseline的训练/验证/测试划分与比赛样本。所有代码在Notebook内，旧数据和baseline保留。

结果分别在 [时间衰减报告](reports/time_decay/RESULTS.zh-CN.md) 和 [LSTM报告](reports/lstm_multitask/RESULTS.zh-CN.md)。各目录同时保存参数、输入说明、逐场预测、配对比较和模型。

LSTM使用PyTorch 2.11.0，当前环境安装CUDA 12.8版本，自动选择GPU或CPU。重新安装依赖可运行 `.venv/bin/python -m pip install -r requirements.txt`（Windows 为 `.\.venv\Scripts\python.exe -m pip install -r requirements.txt`）。无 NVIDIA 显卡的机器可以按 PyTorch 官方说明改装 CPU 版本。

## 从 GitHub 克隆后运行

先按 [原始数据说明](data/raw/README.md) 下载两份固定版本的 CSV，再创建环境。

Windows：

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m ipykernel install --sys-prefix --name aml-football --display-name "AML Football (.venv)"
.\.venv\Scripts\jupyter.exe lab
```

Linux 或 macOS：

```bash
python3.12 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m ipykernel install --sys-prefix --name aml-football --display-name "AML Football (.venv)"
.venv/bin/jupyter lab
```

依次完整运行 `preprocess.ipynb`、`baseline.ipynb`、`time_decay.ipynb`、`lstm_multitask.ipynb`。已执行的 Notebook 和汇总结果随代码保存；原始/处理后 CSV、历史缓存、逐场预测、模型权重和虚拟环境由 `.gitignore` 排除，在本地按此顺序重新生成。

## 模型解释

前四本Notebook完成后，运行[post_analysis.ipynb](notebooks/post_analysis.ipynb)，读取已保存模型：导出22个逻辑回归模型的完整系数，展示历史比赛的固定时间权重，并整理近期/交锋输入组的贡献及多任务损失权重。该步骤不重新训练模型。

解释见[reports/post_analysis/RESULTS.zh-CN.md](reports/post_analysis/RESULTS.zh-CN.md)。系数、时间权重和损失权重含义不同，均不能直接当作特征重要性百分比。
