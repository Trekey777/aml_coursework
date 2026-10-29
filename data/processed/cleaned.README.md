# cleaned 数据集

文件：[cleaned.csv](cleaned.csv)，UTF-8 BOM编码。共133,512场、46列。

从补全缺失值.csv中筛选双方完整最近5场、且三项普通赔率均为有限数并大于1的比赛，再删除指定23列。保留行的值和先后顺序不变。双方Elo和四个Form均完整，三项赔率完整且有效；半场、技术统计及Elo评分日期仍可能缺失。

筛选前137,603场。先排除最近5场不足的2,101场，再从剩余比赛中排除赔率缺失或无效的1,990场，共排除4,091场。赔率无空值但数值不合法的3场也已排除。

删除明细在reports/feature_engineering/cleaned_excluded_matches.csv。这些筛选只作用于最终预测样本，历史统计仍基于原始文件中此前所有有效赛果。

## 删除列

- MatchTime
- MaxHome
- MaxDraw
- MaxAway
- Over25
- Under25
- MaxOver25
- MaxUnder25
- HandiSize
- HandiHome
- HandiAway
- C_LTH
- C_LTA
- C_VHD
- C_VAD
- C_HTB
- C_PHB
- HomeEloOriginallyMissing
- AwayEloOriginallyMissing
- HomeEloSource
- AwayEloSource
- HomeEloFillStatus
- AwayEloFillStatus

## 保留列

- SourceRow
- Division
- MatchDate
- HomeTeam
- AwayTeam
- HomeElo
- AwayElo
- Form3Home
- Form5Home
- Form3Away
- Form5Away
- FTHome
- FTAway
- FTResult
- HTHome
- HTAway
- HTResult
- HomeShots
- AwayShots
- HomeTarget
- AwayTarget
- HomeFouls
- AwayFouls
- HomeCorners
- AwayCorners
- HomeYellow
- AwayYellow
- HomeRed
- AwayRed
- OddHome
- OddDraw
- OddAway
- Country
- HomeEloRatingDate
- AwayEloRatingDate
- Form3HomeCount
- Form3HomeMissing
- Form5HomeCount
- Form5HomeMissing
- Form3AwayCount
- Form3AwayMissing
- Form5AwayCount
- Form5AwayMissing
- HomePriorMatchCount
- AwayPriorMatchCount
- OddsAvailable

全部代码在[02_features.ipynb](../../notebooks/02_features.ipynb)的第9部分。填补来源等信息可按SourceRow在reports/feature_engineering/completion_elo_lineage.csv查找。半场和技术统计仍保留，当前比赛的赛后统计不能作为其赛前预测输入。

这是整理后的数据表，46列不代表全部46列都应该输入模型。

SHA-256：`5f3d64f2b68199c805674921056462d68f50ccb5f5203be9671053ab2fd16d35`。
