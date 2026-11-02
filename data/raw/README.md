# 原始数据

将项目使用的 `Matches.csv` 与 `EloRatings.csv` 放在本目录。
数据来自[课程提供的 GitHub 数据集](https://github.com/xgabora/Club-Football-Match-Data)。

直接使用记录中的日期，不与其他数据源对照，不进行 hash 校验。
数据版本更新后应重新生成公共特征；如果训练期内容改变，应重新训练模型。
处理规则、运行顺序和结果入口见[根目录说明](../../README.md)。
