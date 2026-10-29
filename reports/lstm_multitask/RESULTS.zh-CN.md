# LSTM单任务与硬参数共享多任务：实际运行结果

## 做了什么

近期仍是双方各5场；交锋使用全部此前比赛。近期主客共用一个LSTM，交锋用另一个LSTM；两任务共用这两个编码器及32单元中间层，最后各自输出H/D/A三项概率。只有B的组是MLP。

赛果损失CE，加λ倍赔率KL。赔率仅作辅助训练目标，推断不需要本场赔率；它仍是一种训练期市场信息监督，不能叫“完全不利用赔率”。任务可能互助，也可能互相干扰。没有强迫多任务胜出。

## 输入和输出

| model | base_inputs | recent_sequence | h2h_sequence | training_targets | outputs | lambda |
| --- | --- | --- | --- | --- | --- | --- |
| ST_B | Division one-hot, elo_diff, h2h_count, h2h_missing | none | none | FTResult | outcome probabilities | 0.000000 |
| MT_B | Division one-hot, elo_diff, h2h_count, h2h_missing | none | none | FTResult + q_H/q_D/q_A | outcome probabilities + odds probabilities | 0.300000 |
| ST_F | Division one-hot, elo_diff, h2h_count, h2h_missing | home+away each 5 x [points,GD,draw,age] | none | FTResult | outcome probabilities | 0.000000 |
| MT_F | Division one-hot, elo_diff, h2h_count, h2h_missing | home+away each 5 x [points,GD,draw,age] | none | FTResult + q_H/q_D/q_A | outcome probabilities + odds probabilities | 0.300000 |
| ST_H | Division one-hot, elo_diff, h2h_count, h2h_missing | none | all prior x [points difference,GD,draw,age] | FTResult | outcome probabilities | 0.000000 |
| MT_H | Division one-hot, elo_diff, h2h_count, h2h_missing | none | all prior x [points difference,GD,draw,age] | FTResult + q_H/q_D/q_A | outcome probabilities + odds probabilities | 1.000000 |
| ST_FH | Division one-hot, elo_diff, h2h_count, h2h_missing | home+away each 5 x [points,GD,draw,age] | all prior x [points difference,GD,draw,age] | FTResult | outcome probabilities | 0.000000 |
| MT_FH | Division one-hot, elo_diff, h2h_count, h2h_missing | home+away each 5 x [points,GD,draw,age] | all prior x [points difference,GD,draw,age] | FTResult + q_H/q_D/q_A | outcome probabilities + odds probabilities | 0.300000 |

每条近期记录为该队积分、净胜球、平局标记、距本场天数；每条交锋为本场主队减客队的积分差、主队净胜球、平局标记、距本场天数。积分/3、净胜球/5、年龄log(1+天数)/log366，不用测试集估计比例。每个编码器8维输出，近期取主客表示之差和均值；交锋取主队视角表示。隐藏序列按90/365天半衰期加权，空交锋全0且B有缺失标记。8维表示没有预先定义的“优势”单位。

## 如何选择与运行

训练106288场（≤2020），验证12598场（2021—22），测试11772场（2023—24）；2025不参与。4组×4个λ在验证集选轮数，主目标均为赛果Log Loss；λ为0保留单任务，正λ中选一个多任务。各选择以3个seed在截至2022年的118886场重训。主表为3个seed概率平均，各seed见seed_scores.csv。测试不用于挑seed、λ或轮数。

| group | single_epochs | multi_lambda | multi_epochs | single_val_loss | multi_val_loss | validation_preferred |
| --- | --- | --- | --- | --- | --- | --- |
| B | 8 | 0.300000 | 8 | 1.016896 | 1.016648 | multi |
| F | 8 | 0.300000 | 8 | 1.015865 | 1.015615 | multi |
| H | 8 | 1.000000 | 8 | 1.016145 | 1.015622 | multi |
| FH | 8 | 0.300000 | 8 | 1.015099 | 1.014759 | multi |

## 真实赛果预测

| model | log_loss | brier | accuracy |
| --- | --- | --- | --- |
| M0 | 1.003156 | 0.599690 | 0.502718 |
| M1 | 1.002596 | 0.599286 | 0.501189 |
| M2 | 1.003159 | 0.599688 | 0.503313 |
| M3 | 1.002583 | 0.599273 | 0.501954 |
| TD_M1 | 1.002421 | 0.599169 | 0.501104 |
| TD_M2 | 1.003141 | 0.599664 | 0.504077 |
| TD_M3 | 1.002407 | 0.599144 | 0.502379 |
| ST_B | 1.002217 | 0.599221 | 0.503058 |
| ST_F | 1.001342 | 0.598606 | 0.503313 |
| ST_H | 1.002177 | 0.599181 | 0.502548 |
| ST_FH | 1.001317 | 0.598578 | 0.503483 |
| MT_B | 1.002026 | 0.599096 | 0.503313 |
| MT_F | 1.001121 | 0.598463 | 0.503738 |
| MT_H | 1.001935 | 0.598996 | 0.505776 |
| MT_FH | 1.001042 | 0.598387 | 0.503908 |
| Market | 0.987725 | 0.589294 | 0.519878 |
| Qcal | 0.986260 | 0.588522 | 0.519368 |

## 多任务是否优于同结构单任务

| left | right | metric | difference | ci_low | ci_high | ci_includes_zero |
| --- | --- | --- | --- | --- | --- | --- |
| MT_B | ST_B | log_loss | -0.000190 | -0.000294 | -0.000075 | False |
| MT_F | ST_F | log_loss | -0.000221 | -0.000355 | -0.000075 | False |
| MT_H | ST_H | log_loss | -0.000242 | -0.000493 | 0.000039 | True |
| MT_FH | ST_FH | log_loss | -0.000275 | -0.000439 | -0.000115 | False |

负差值表示左边更好；区间跨0不能断言稳定改善。区间是测试期28天块、按年分层bootstrap的描述性95%区间（1000次），不包含训练或选参不确定性，也未校正多重比较。

## 赔率拟合

| model | kl_to_odds | odds_probability_mae |
| --- | --- | --- |
| MT_B_odds | 0.012582 | 0.037877 |
| MT_F_odds | 0.011813 | 0.036870 |
| MT_H_odds | 0.012386 | 0.037559 |
| MT_FH_odds | 0.011758 | 0.036783 |
| O0 | 0.013099 | 0.038584 |
| O1 | 0.012378 | 0.037852 |
| O2 | 0.013028 | 0.038467 |
| O3 | 0.012289 | 0.037713 |
| TD_O1 | 0.012317 | 0.037789 |
| TD_O2 | 0.013041 | 0.038506 |
| TD_O3 | 0.012245 | 0.037697 |

赔率表用多任务的独立赔率头；赛果表用赛果头。单任务赔率头没有受训，不报告。λ按赛果而非赔率选择，所以多任务赔率头不一定达到最优赔率拟合。能拟合市场不等于能赚钱，也不能证明博彩公司用了同样特征。

## 如何客观解读

- B：多任务−单任务Log Loss=-0.000190，95%区间[-0.000294, -0.000075]。多任务的测试损失较低。
- F：多任务−单任务Log Loss=-0.000221，95%区间[-0.000355, -0.000075]。多任务的测试损失较低。
- H：多任务−单任务Log Loss=-0.000242，95%区间[-0.000493, 0.000039]。区间跨0，差异不确定。
- FH：多任务−单任务Log Loss=-0.000275，95%区间[-0.000439, -0.000115]。多任务的测试损失较低。

这是小型网络的第一版实证，不代表LSTM的性能上限。与逻辑回归的差别包含序列信息保留、非线性预测层和（MT）训练期赔率监督；不能把全部差异归功于LSTM。比较ST和MT才较直接检验这个辅助任务。训练seed有限，λ只用一个seed选择；还没有独立的非线性统计特征控制实验。年龄进入网络，信息量也与统计摘要不同。

此次LSTM未实现“本场赔率作为输入”的增量模型，所以不由此声称在给定赔率后历史仍有增益；该问题当前只有time_decay和旧baseline中的Q系列证据。无需把每个方向一次堆入第一版。

2023—24此前已查看，是后续分析。结论限于Elo完整、近期5场完整、赔率有效样本及当前参数；原始日期和现有Elo按项目约定直接使用。
