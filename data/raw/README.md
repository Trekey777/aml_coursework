# 原始数据

本项目只使用指定数据仓库中的两份 CSV。为保证结果可复现，固定使用提交 `4f4d8dd15f0e7290808e864b2f0b9f7457ee84fa` 中的数据。

下载后放在当前目录，保持文件名不变：

- [Matches.csv](https://raw.githubusercontent.com/xgabora/Club-Football-Match-Data/4f4d8dd15f0e7290808e864b2f0b9f7457ee84fa/data/Matches.csv)
- [EloRatings.csv](https://raw.githubusercontent.com/xgabora/Club-Football-Match-Data/4f4d8dd15f0e7290808e864b2f0b9f7457ee84fa/data/EloRatings.csv)

文件大小和参考 SHA-256 值记录在 [manifest.json](manifest.json)，用于记录数据版本。Notebook直接读取CSV，不进行哈希检查。

原始 CSV、生成的数据表、逐场预测和模型权重不提交到代码仓库，可以按根目录 README 的顺序运行 Notebook 重新生成。代码仓库保留已执行的 Notebook、数据说明、汇总分数和图表。
