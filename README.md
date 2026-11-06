# 足球比赛分析

比较近期五场与全部历史交锋对胜平负预测的帮助，以及它们与博彩市场概率的关系。

## 运行方式

所有分析代码在三本 Jupyter Notebook 中。先运行预处理，再运行所需模型。两条模型路线不需要互相执行。

| Notebook | 输入 | 做什么 | 输出 |
|---|---|---|---|
| [preprocess.ipynb](notebooks/preprocess.ipynb) | 两份原始 CSV | 清洗、补 Elo、重算 Form、建立候选指标并筛选 | `data/processed/` |
| [baseline.ipynb](notebooks/baseline.ipynb) | `candidate_features.csv` 与 `selected_features.csv` | 等权统计＋逻辑回归；赛果、市场拟合、赔率后的历史增量 | `data/results/baseline/` |
| [nonlinear.ipynb](notebooks/nonlinear.ipynb) | `candidate_features.csv` 与 `selected_features.csv` | 同一输入组上的梯度提升树 | `data/results/nonlinear/` |

默认 `RETRAIN=False`：已有且输入一致的模型直接预测，不重新训练。缺少模型或筛选后的列发生变化时，运行验证选参和最终训练；`RETRAIN=True` 可以强制重训。

两条模型路线各自完成实验。为了得到完整总报告，先运行预处理，再运行 baseline 和非线性模型。每本从上到下运行。

## 安装

```bash
python3.12 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m ipykernel install --sys-prefix --name aml-football --display-name "AML Football (.venv)"
.venv/bin/jupyter lab
```

Windows 使用 `.venv\Scripts\python.exe` 和 `.venv\Scripts\jupyter.exe`。从项目根目录启动 Jupyter，选择 AML Football 内核。

## 数据和处理规则

两份输入是 `data/raw/Matches.csv` 和 `data/raw/EloRatings.csv`，来自[课程数据集](https://github.com/xgabora/Club-Football-Match-Data)。使用项目中的数据版本，直接按记录日期处理，不与其他来源对照，也不做 hash 校验。

- 身份、日期、全场赛果缺失或无效的记录删除。
- 保留已有 Elo；缺失时按国家和球队查找严格早于比赛日的最近评分，不限制间隔。仍缺任一侧 Elo 的比赛不进入预测样本。
- 近期固定最近五场，主客场都算；交锋使用此前全部直接比赛。历史池包含全部有效赛果，不因缺 Elo 或赔率而删除历史。
- 同一天先生成全部赛前特征，再更新当日赛果。
- 预测样本要求双方五场完整，三项普通赔率均为有限数且大于 1。
- 比赛表在预处理中使用，不单独保存。半场字段不进入模型；Form3/Form5 统一重算。技术统计、额外盘口和 C_* 列不进入本版。
- `data/processed/candidate_features.csv` 保存全部候选指标。`selected_features.csv` 只保存筛选后进入模型的列。

2020 年及以前训练，2021—2022 年验证选参数，最终拟合截至 2022 年，评价 2023—2024 年。2025 年及以后保留在公共表中，但不参与当前拟合和评分。标准化仅在当前拟合集学习。

## 结果在哪里

阅读一个[总报告](reports/reports.md)，其中包含公共数据、baseline 和非线性模型。每本 Notebook 只更新自己的报告章节。baseline 生成赛果/赔率对比图，非线性模型生成梯度提升对比图。

逐场预测、模型、分数、配对区间和必要解释表集中在 `data/results/<route>/`。系数写在 baseline，梯度提升的比较写在非线性模型。

原始数据、候选特征、筛选结果、模型和逐场预测都在 `data/` 中，`.gitignore` 不排除它们，可以随仓库上传。最大的单文件约 44 MB。克隆后可以直接运行模型笔记本；只有原始表或筛选规则改变时才需要重跑预处理。

当前结论的范围与限制见总报告。近期窗口固定五场，尚未比较其他 lookback 窗口。2023—2024 年此前已被查看，当前比较不是全新的盲测。
