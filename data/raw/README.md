# 原始数据

本项目只使用指定数据仓库中的两份 CSV。为保证结果可复现，固定使用提交 `4f4d8dd15f0e7290808e864b2f0b9f7457ee84fa` 中的数据。

下载后放在当前目录，保持文件名不变：

- [Matches.csv](https://raw.githubusercontent.com/xgabora/Club-Football-Match-Data/4f4d8dd15f0e7290808e864b2f0b9f7457ee84fa/data/Matches.csv)
- [EloRatings.csv](https://raw.githubusercontent.com/xgabora/Club-Football-Match-Data/4f4d8dd15f0e7290808e864b2f0b9f7457ee84fa/data/EloRatings.csv)

文件大小和参考 SHA-256 值记录在 [manifest.json](manifest.json)。Notebook 会记录当前原始文件的实际哈希，跳过与 manifest 中参考哈希的比对。因此，CSV 的字节内容与参考版本不同，也可以继续处理。

原始 CSV、生成的数据表、逐场预测和模型权重不提交到代码仓库，可以按根目录 README 的顺序运行 Notebook 重新生成。代码仓库保留已执行的 Notebook、数据说明、汇总分数、图表和核查记录。
