# cleaned 数据集

[cleaned.csv](cleaned.csv)：133,512场、46列。只用原始两份CSV直接生成，完整处理代码见[02_features.ipynb](../../notebooks/02_features.ipynb)。

已有Elo保留；缺失时查找同一国家和球队、严格早于比赛日的最近评分，没有年限上限。双方Elo仍有缺失、最近5场不足或普通赔率无效的比赛删除。Form为此前恰好3/5场的积分总和，历史使用全部核心赛果有效的比赛。

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

半场、技术统计及原有Elo的评分日期仍允许缺失。表中含身份、赛果和辅助记录，不代表所有列都进入模型；本场赛后统计不作为赛前预测输入。
