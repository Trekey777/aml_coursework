# 足球比赛分析

比较近期五场与全部历史交锋对胜平负预测的帮助，以及它们与博彩市场概率的关系。

## 运行方式

所有分析代码在四本 Jupyter Notebook 中。先运行预处理，再运行所需模型路线；三条路线不需要互相执行。

| Notebook | 输入 | 做什么 | 输出 |
|---|---|---|---|
| [preprocess.ipynb](notebooks/preprocess.ipynb) | 两份原始 CSV | 清洗、补 Elo、重算 Form、共同特征、历史序列与时间划分 | `data/processed/` |
| [baseline.ipynb](notebooks/baseline.ipynb) | 公共特征 | 等权统计＋逻辑回归；赛果、市场拟合、赔率后的历史增量 | `data/results/baseline/` |
| [time_decay.ipynb](notebooks/time_decay.ipynb) | 公共特征和历史序列 | 90/365 天半衰期加权统计＋逻辑回归 | `data/results/time_decay/` |
| [lstm_multitask.ipynb](notebooks/lstm_multitask.ipynb) | 公共特征和历史序列 | 单任务与硬参数共享多任务网络 | `data/results/lstm_multitask/` |

默认 `RETRAIN=False`：逻辑回归加载已有模型预测，LSTM 加载已有权重推断，不重新训练。缺少模型时，运行原有的验证选参和最终训练；改为 `RETRAIN=True` 可以强制重训。更换训练数据、输入定义或模型配置后应重新训练，不应直接复用旧模型。

模型路线独立完成自己的实验；其他路线已有预测时，额外生成跨路线对照。为了得到完整总报告，建议依次运行预处理、baseline、time_decay、LSTM。每本从上到下运行。

## 安装

```bash
python3.12 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m ipykernel install --sys-prefix --name aml-football --display-name "AML Football (.venv)"
.venv/bin/jupyter lab
```

Windows 使用 `.venv\Scripts\python.exe` 和 `.venv\Scripts\jupyter.exe`。从项目根目录启动 Jupyter，选择 AML Football 内核。PyTorch 自动选择 GPU 或 CPU；依赖当前指定 CUDA 12.8 wheel，没有 NVIDIA 显卡时也可安装 CPU 版。

## 数据和处理规则

两份输入是 `data/raw/Matches.csv` 和 `data/raw/EloRatings.csv`，来自[课程数据集](https://github.com/xgabora/Club-Football-Match-Data)。使用项目中的数据版本，直接按记录日期处理，不与其他来源对照，也不做 hash 校验。

- 身份、日期、全场赛果缺失或无效的记录删除。
- 保留已有 Elo；缺失时按国家和球队查找严格早于比赛日的最近评分，不限制间隔。仍缺任一侧 Elo 的比赛不进入预测样本。
- 近期固定最近五场，主客场都算；交锋使用此前全部直接比赛。历史池包含全部有效赛果，不因缺 Elo 或赔率而删除历史。
- 同一天先生成全部赛前特征，再更新当日赛果。
- 预测样本要求双方五场完整，三项普通赔率均为有限数且大于 1。
- cleaned 保存 21 列。半场字段不进入模型；Form3/Form5 统一重算。技术统计、额外盘口和 C_* 列不进入本版。

2020 年及以前训练，2021—2022 年验证选参数，最终拟合截至 2022 年，评价 2023—2024 年。2025 年及以后保留在公共表中，但不参与当前拟合和评分。标准化仅在当前拟合集学习。

## 结果在哪里

阅读一个[总报告](reports/reports.md)，其中包含公共数据、baseline、time_decay 和 LSTM。每本 Notebook 只更新自己的报告章节。`reports/figures/` 的四张 PNG 都由当前 Notebook 自动生成：baseline 生成赛果/赔率对比图，time_decay 生成衰减对比图和系数图，LSTM 生成预测与输入组贡献图。旧版原图继续归档。

逐场预测、模型、分数、配对区间和必要解释表集中在 `data/results/<route>/`；这些是复用和计算所需的数据，不再生成大量独立说明文档。之前的实验文件保存在 `data/results/archive/` 和原路线结果目录中；无需运行独立的 post_analysis，系数、时间权重和网络贡献已经并入对应模型路线。

原始数据、处理缓存、模型、逐场预测和本地归档由 `.gitignore` 排除；仓库保留四本已执行 Notebook、一个总报告及图片。克隆后把两份 CSV 放到 `data/raw/`，运行预处理和所需路线即可重建。数据、模型和预测若已在 Git 中被跟踪，仅修改 `.gitignore` 不会自动取消跟踪，提交时仍应检查文件清单。

当前结论的范围与限制见总报告。近期窗口固定五场，尚未比较其他 lookback 窗口。2023—2024 年此前已被查看，当前比较不是全新的盲测。

## 本地 AI 交付包

完整压缩包仅保留在本地 `deliverables/`，不通过 GitHub 发布。它包含代码、数据、模型、预测、结果和归档，解压后先阅读 `AI_HANDOFF.md`。内容说明见[交付说明](deliverables/README.md)。
