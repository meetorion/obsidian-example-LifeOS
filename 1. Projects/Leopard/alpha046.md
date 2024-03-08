# alpha046

## 代码
```python
def alpha046(df: ps.DataFrame):
    # compile 6.787904262542725 s, execute 750.4142465591431 s (12min)
    df = df[
        ["code", "close_price"]
    ]  # 存储scan了三列，多了一个ts列，是否可以不scan ts列？
    # df.spark.cache()
    delay_20 = df.groupby("code").shift(20)["close_price"]  # scan
    delay_10 = df.groupby("code").shift(10)["close_price"]  # scan
    cond = ((delay_20 - delay_10) / 10 - (delay_10 - df["close_price"]) / 10).rename(
        "cond"
    )
    alpha046 = (-1 * df.groupby("code").diff(1)["close_price"]).rename(
        "alpha046"
    )  # scan
    tmp = cond.to_frame().join(alpha046)
    tmp.loc[tmp["cond"] < 0, "alpha046"] = 1
    tmp.loc[tmp["cond"] > 0.25, "alpha046"] = -1
    ans = tmp["alpha046"].replace([-np.inf, np.inf], 0).fillna(value=0)
    # ans.to_frame().spark.explain()
    return ans
```
## Stage划分是否合理(有时会划分成Scan3次)
1. BatchScan
2. AttachDistributedSequence
3. BatchScan
4. AttachDistributedSequence

可以该Spark执行计划，使得只Scan一次。#leopard/执行计划 [[如何修改Spark的执行计划]]


## Job划分是否合理
划分成了11个Job
1. Scan + AttachDistributedSequence 得到需要的列
2. Scan + AttachDistributedSequence + Exchange 进行Groupby操作
3. Scan + AttachDistributedSequence 得到需要的列
4. Scan + AttachDistributedSequence + Exchange 进行Groupby操作
5. AQEShuffleRead + Window + Exchange 进行shift操作
6. AQEShuffleRead + Window + Exchage 进行shift操作
7. AQEShuffleRead + Window + Exchage 进行shift操作
8. Exchange 
9. Exchange
10. Exchange
11. Exchange + CollectLimit + mapPartitionsInternal
## Stage是否合理
### BatchScan:合理
触发代码是：
```pythob
df = df[
        ["code", "close_price"]
    ]
```
Spark执行计划扫描了code列和close_price，存储扫描了code、close_price和ts列。


### AttachDistributedSequence
PySpark的机制，赋予分布式id，并将分布式id作为索引
## 调优动作(资源参数)
- 啥也不调：compile 3.2542755603790283 s, execute 41.02533984184265 s
- [[使用Kryo序列化]]:compile 7.137723207473755 s, execute 44.39489507675171 s

- 设置并行度为128（4个excutor，每个excutor有16个cpu核心）:compile 7.069753885269165 s, execute 44.75358176231384 s

- 设置“存储”内存占比为0.1：compile 7.273091077804565 s, execute 47.623881816864014 s
- spark.shuffle.memoryFraction=0.8：compile 6.822169303894043 s, execute 44.814592123031616 s

## 原理
- [] [[如何划分stage]]
- [] 为什么会shuffle，如何对shuffle进行优化



```
spark.memory.storageFraction=0.1,spark.default.parallelism=128,spark.serializer=org.apache.spark.serializer.KryoSerializer
```

## 执行计划
```
== Physical Plan ==
AdaptiveSparkPlan (145)
+- == Final Plan ==
   CollectLimit (75)
   +- * Project (74)
      +- * SortMergeJoin LeftOuter (73)
         :- * LocalLimit (35)
         :  +- * Project (34)
         :     +- * SortMergeJoin LeftOuter (33)
         :        :- * LocalLimit (22)
         :        :  +- * Project (21)
         :        :     +- * SortMergeJoin LeftOuter (20)
         :        :        :- * Sort (10)
         :        :        :  +- AQEShuffleRead (9)
         :        :        :     +- ShuffleQueryStage (8), Statistics(sizeInBytes=250.3 KiB, rowCount=1.60E+4)
         :        :        :        +- Exchange (7)
         :        :        :           +- * LocalLimit (6)
         :        :        :              +- * Project (5)
         :        :        :                 +- AttachDistributedSequence (4)
         :        :        :                    +- * Project (3)
         :        :        :                       +- * ColumnarToRow (2)
         :        :        :                          +- BatchScan finance.sel2_order_20231218163228276 (1)
         :        :        +- * Sort (19)
         :        :           +- AQEShuffleRead (18)
         :        :              +- ShuffleQueryStage (17), Statistics(sizeInBytes=690.1 MiB, rowCount=4.52E+7)
         :        :                 +- Exchange (16)
         :        :                    +- * Project (15)
         :        :                       +- AttachDistributedSequence (14)
         :        :                          +- * Project (13)
         :        :                             +- * ColumnarToRow (12)
         :        :                                +- BatchScan finance.sel2_order_20231218163228276 (11)
         :        +- * Project (32)
         :           +- * SortMergeJoin LeftOuter (31)
         :              :- * Sort (26)
         :              :  +- AQEShuffleRead (25)
         :              :     +- ShuffleQueryStage (24), Statistics(sizeInBytes=690.1 MiB, rowCount=4.52E+7)
         :              :        +- ReusedExchange (23)
         :              +- * Sort (30)
         :                 +- AQEShuffleRead (29)
         :                    +- ShuffleQueryStage (28), Statistics(sizeInBytes=690.1 MiB, rowCount=4.52E+7)
         :                       +- ReusedExchange (27)
         +- * Sort (72)
            +- AQEShuffleRead (71)
               +- ShuffleQueryStage (70), Statistics(sizeInBytes=1035.2 MiB, rowCount=4.52E+7)
                  +- Exchange (69)
                     +- * Project (68)
                        +- Window (67)
                           +- * Sort (66)
                              +- AQEShuffleRead (65)
                                 +- ShuffleQueryStage (64), Statistics(sizeInBytes=2.4 GiB, rowCount=4.52E+7)
                                    +- Exchange (63)
                                       +- * Project (62)
                                          +- * Project (61)
                                             +- * SortMergeJoin LeftOuter (60)
                                                :- * Project (49)
                                                :  +- * SortMergeJoin LeftOuter (48)
                                                :     :- * Sort (43)
                                                :     :  +- AQEShuffleRead (42)
                                                :     :     +- ShuffleQueryStage (41), Statistics(sizeInBytes=2.0 GiB, rowCount=4.52E+7)
                                                :     :        +- Exchange (40)
                                                :     :           +- * Project (39)
                                                :     :              +- AttachDistributedSequence (38)
                                                :     :                 +- * ColumnarToRow (37)
                                                :     :                    +- BatchScan finance.sel2_order_20231218163228276 (36)
                                                :     +- * Sort (47)
                                                :        +- AQEShuffleRead (46)
                                                :           +- ShuffleQueryStage (45), Statistics(sizeInBytes=690.1 MiB, rowCount=4.52E+7)
                                                :              +- ReusedExchange (44)
                                                +- * Project (59)
                                                   +- * SortMergeJoin LeftOuter (58)
                                                      :- * Sort (53)
                                                      :  +- AQEShuffleRead (52)
                                                      :     +- ShuffleQueryStage (51), Statistics(sizeInBytes=690.1 MiB, rowCount=4.52E+7)
                                                      :        +- ReusedExchange (50)
                                                      +- * Sort (57)
                                                         +- AQEShuffleRead (56)
                                                            +- ShuffleQueryStage (55), Statistics(sizeInBytes=690.1 MiB, rowCount=4.52E+7)
                                                               +- ReusedExchange (54)
+- == Initial Plan ==
   CollectLimit (144)
   +- Project (143)
      +- SortMergeJoin LeftOuter (142)
         :- LocalLimit (106)
         :  +- Project (105)
         :     +- SortMergeJoin LeftOuter (104)
         :        :- LocalLimit (89)
         :        :  +- Project (88)
         :        :     +- SortMergeJoin LeftOuter (87)
         :        :        :- Sort (81)
         :        :        :  +- Exchange (80)
         :        :        :     +- LocalLimit (79)
         :        :        :        +- Project (78)
         :        :        :           +- AttachDistributedSequence (77)
         :        :        :              +- Project (76)
         :        :        :                 +- BatchScan finance.sel2_order_20231218163228276 (1)
         :        :        +- Sort (86)
         :        :           +- Exchange (85)
         :        :              +- Project (84)
         :        :                 +- AttachDistributedSequence (83)
         :        :                    +- Project (82)
         :        :                       +- BatchScan finance.sel2_order_20231218163228276 (11)
         :        +- Project (103)
         :           +- SortMergeJoin LeftOuter (102)
         :              :- Sort (95)
         :              :  +- Exchange (94)
         :              :     +- Project (93)
         :              :        +- AttachDistributedSequence (92)
         :              :           +- Project (91)
         :              :              +- BatchScan finance.sel2_order_20231218163228276 (90)
         :              +- Sort (101)
         :                 +- Exchange (100)
         :                    +- Project (99)
         :                       +- AttachDistributedSequence (98)
         :                          +- Project (97)
         :                             +- BatchScan finance.sel2_order_20231218163228276 (96)
         +- Sort (141)
            +- Exchange (140)
               +- Project (139)
                  +- Window (138)
                     +- Sort (137)
                        +- Exchange (136)
                           +- Project (135)
                              +- Project (134)
                                 +- SortMergeJoin LeftOuter (133)
                                    :- Project (118)
                                    :  +- SortMergeJoin LeftOuter (117)
                                    :     :- Sort (110)
                                    :     :  +- Exchange (109)
                                    :     :     +- Project (108)
                                    :     :        +- AttachDistributedSequence (107)
                                    :     :           +- BatchScan finance.sel2_order_20231218163228276 (36)
                                    :     +- Sort (116)
                                    :        +- Exchange (115)
                                    :           +- Project (114)
                                    :              +- AttachDistributedSequence (113)
                                    :                 +- Project (112)
                                    :                    +- BatchScan finance.sel2_order_20231218163228276 (111)
                                    +- Project (132)
                                       +- SortMergeJoin LeftOuter (131)
                                          :- Sort (124)
                                          :  +- Exchange (123)
                                          :     +- Project (122)
                                          :        +- AttachDistributedSequence (121)
                                          :           +- Project (120)
                                          :              +- BatchScan finance.sel2_order_20231218163228276 (119)
                                          +- Sort (130)
                                             +- Exchange (129)
                                                +- Project (128)
                                                   +- AttachDistributedSequence (127)
                                                      +- Project (126)
                                                         +- BatchScan finance.sel2_order_20231218163228276 (125)


(1) BatchScan finance.sel2_order_20231218163228276
Output [1]: [symbol#4742236]
Format: timelyre
PushedLimit: 0
ReadSchema: struct<symbol:string>
ShivaMasterAddress: 172.18.128.49:8530,172.18.128.50:8530,172.18.128.51:8530
TableName: finance.sel2_order_20231218163228276

(2) ColumnarToRow [codegen id : 1]
Input [1]: [symbol#4742236]

(3) Project [codegen id : 1]
Output: []
Input [1]: [symbol#4742236]

(4) AttachDistributedSequence
Input: []
Arguments: distributed_sequence_id#4742252: bigint

(5) Project [codegen id : 2]
Output [1]: [distributed_sequence_id#4742252L AS __this___index_level_0__#4742290L]
Input [1]: [distributed_sequence_id#4742252L]

(6) LocalLimit [codegen id : 2]
Input [1]: [__this___index_level_0__#4742290L]
Arguments: 1001

(7) Exchange
Input [1]: [__this___index_level_0__#4742290L]
Arguments: hashpartitioning(__this___index_level_0__#4742290L, 200), ENSURE_REQUIREMENTS, [plan_id=1101091]

(8) ShuffleQueryStage
Output [1]: [__this___index_level_0__#4742290L]
Arguments: 0

(9) AQEShuffleRead
Input [1]: [__this___index_level_0__#4742290L]
Arguments: coalesced

(10) Sort [codegen id : 26]
Input [1]: [__this___index_level_0__#4742290L]
Arguments: [__this___index_level_0__#4742290L ASC NULLS FIRST], false, 0

(11) BatchScan finance.sel2_order_20231218163228276
Output [1]: [symbol#4742308]
Format: timelyre
PushedLimit: 0
ReadSchema: struct<symbol:string>
ShivaMasterAddress: 172.18.128.49:8530,172.18.128.50:8530,172.18.128.51:8530
TableName: finance.sel2_order_20231218163228276

(12) ColumnarToRow [codegen id : 3]
Input [1]: [symbol#4742308]

(13) Project [codegen id : 3]
Output: []
Input [1]: [symbol#4742308]

(14) AttachDistributedSequence
Input: []
Arguments: distributed_sequence_id#4742324: bigint

(15) Project [codegen id : 4]
Output [1]: [distributed_sequence_id#4742324L AS __that___index_level_0__#4742364L]
Input [1]: [distributed_sequence_id#4742324L]

(16) Exchange
Input [1]: [__that___index_level_0__#4742364L]
Arguments: hashpartitioning(__that___index_level_0__#4742364L, 200), ENSURE_REQUIREMENTS, [plan_id=1101126]

(17) ShuffleQueryStage
Output [1]: [__that___index_level_0__#4742364L]
Arguments: 1

(18) AQEShuffleRead
Input [1]: [__that___index_level_0__#4742364L]
Arguments: coalesced

(19) Sort [codegen id : 27]
Input [1]: [__that___index_level_0__#4742364L]
Arguments: [__that___index_level_0__#4742364L ASC NULLS FIRST], false, 0

(20) SortMergeJoin [codegen id : 28]
Left keys [1]: [__this___index_level_0__#4742290L]
Right keys [1]: [__that___index_level_0__#4742364L]
Join type: LeftOuter
Join condition: None

(21) Project [codegen id : 28]
Output [1]: [__this___index_level_0__#4742290L]
Input [2]: [__this___index_level_0__#4742290L, __that___index_level_0__#4742364L]

(22) LocalLimit [codegen id : 28]
Input [1]: [__this___index_level_0__#4742290L]
Arguments: 1001

(23) ReusedExchange [Reuses operator id: 16]
Output [1]: [__this___index_level_0__#4742499L]

(24) ShuffleQueryStage
Output [1]: [__this___index_level_0__#4742499L]
Arguments: 3

(25) AQEShuffleRead
Input [1]: [__this___index_level_0__#4742499L]
Arguments: coalesced

(26) Sort [codegen id : 29]
Input [1]: [__this___index_level_0__#4742499L]
Arguments: [__this___index_level_0__#4742499L ASC NULLS FIRST], false, 0

(27) ReusedExchange [Reuses operator id: 16]
Output [1]: [__that___index_level_0__#4742573L]

(28) ShuffleQueryStage
Output [1]: [__that___index_level_0__#4742573L]
Arguments: 5

(29) AQEShuffleRead
Input [1]: [__that___index_level_0__#4742573L]
Arguments: coalesced

(30) Sort [codegen id : 30]
Input [1]: [__that___index_level_0__#4742573L]
Arguments: [__that___index_level_0__#4742573L ASC NULLS FIRST], false, 0

(31) SortMergeJoin [codegen id : 31]
Left keys [1]: [__this___index_level_0__#4742499L]
Right keys [1]: [__that___index_level_0__#4742573L]
Join type: LeftOuter
Join condition: None

(32) Project [codegen id : 31]
Output [1]: [__this___index_level_0__#4742499L AS __that___index_level_0__#4742626L]
Input [2]: [__this___index_level_0__#4742499L, __that___index_level_0__#4742573L]

(33) SortMergeJoin [codegen id : 32]
Left keys [1]: [__this___index_level_0__#4742290L]
Right keys [1]: [__that___index_level_0__#4742626L]
Join type: LeftOuter
Join condition: None

(34) Project [codegen id : 32]
Output [1]: [__this___index_level_0__#4742290L]
Input [2]: [__this___index_level_0__#4742290L, __that___index_level_0__#4742626L]

(35) LocalLimit [codegen id : 32]
Input [1]: [__this___index_level_0__#4742290L]
Arguments: 1001

(36) BatchScan finance.sel2_order_20231218163228276
Output [2]: [orderprice#4742774, code#4742783]
Format: timelyre
PushedLimit: 0
ReadSchema: struct<orderprice:double,code:string>
ShivaMasterAddress: 172.18.128.49:8530,172.18.128.50:8530,172.18.128.51:8530
TableName: finance.sel2_order_20231218163228276

(37) ColumnarToRow [codegen id : 9]
Input [2]: [orderprice#4742774, code#4742783]

(38) AttachDistributedSequence
Input [2]: [orderprice#4742774, code#4742783]
Arguments: distributed_sequence_id#4742785: bigint

(39) Project [codegen id : 10]
Output [3]: [distributed_sequence_id#4742785L AS __this___index_level_0__#4742823L, orderprice#4742774 AS __this_orderprice#4742829, code#4742783 AS __this_code#4742838]
Input [3]: [distributed_sequence_id#4742785L, orderprice#4742774, code#4742783]

(40) Exchange
Input [3]: [__this___index_level_0__#4742823L, __this_orderprice#4742829, __this_code#4742838]
Arguments: hashpartitioning(__this___index_level_0__#4742823L, 200), ENSURE_REQUIREMENTS, [plan_id=1101290]

(41) ShuffleQueryStage
Output [3]: [__this___index_level_0__#4742823L, __this_orderprice#4742829, __this_code#4742838]
Arguments: 6

(42) AQEShuffleRead
Input [3]: [__this___index_level_0__#4742823L, __this_orderprice#4742829, __this_code#4742838]
Arguments: coalesced

(43) Sort [codegen id : 17]
Input [3]: [__this___index_level_0__#4742823L, __this_orderprice#4742829, __this_code#4742838]
Arguments: [__this___index_level_0__#4742823L ASC NULLS FIRST], false, 0

(44) ReusedExchange [Reuses operator id: 16]
Output [1]: [__that___index_level_0__#4742897L]

(45) ShuffleQueryStage
Output [1]: [__that___index_level_0__#4742897L]
Arguments: 8

(46) AQEShuffleRead
Input [1]: [__that___index_level_0__#4742897L]
Arguments: coalesced

(47) Sort [codegen id : 18]
Input [1]: [__that___index_level_0__#4742897L]
Arguments: [__that___index_level_0__#4742897L ASC NULLS FIRST], false, 0

(48) SortMergeJoin [codegen id : 19]
Left keys [1]: [__this___index_level_0__#4742823L]
Right keys [1]: [__that___index_level_0__#4742897L]
Join type: LeftOuter
Join condition: None

(49) Project [codegen id : 19]
Output [3]: [__this___index_level_0__#4742823L, __this_orderprice#4742829, __this_code#4742838]
Input [4]: [__this___index_level_0__#4742823L, __this_orderprice#4742829, __this_code#4742838, __that___index_level_0__#4742897L]

(50) ReusedExchange [Reuses operator id: 16]
Output [1]: [__this___index_level_0__#4743032L]

(51) ShuffleQueryStage
Output [1]: [__this___index_level_0__#4743032L]
Arguments: 10

(52) AQEShuffleRead
Input [1]: [__this___index_level_0__#4743032L]
Arguments: coalesced

(53) Sort [codegen id : 20]
Input [1]: [__this___index_level_0__#4743032L]
Arguments: [__this___index_level_0__#4743032L ASC NULLS FIRST], false, 0

(54) ReusedExchange [Reuses operator id: 16]
Output [1]: [__that___index_level_0__#4743106L]

(55) ShuffleQueryStage
Output [1]: [__that___index_level_0__#4743106L]
Arguments: 12

(56) AQEShuffleRead
Input [1]: [__that___index_level_0__#4743106L]
Arguments: coalesced

(57) Sort [codegen id : 21]
Input [1]: [__that___index_level_0__#4743106L]
Arguments: [__that___index_level_0__#4743106L ASC NULLS FIRST], false, 0

(58) SortMergeJoin [codegen id : 22]
Left keys [1]: [__this___index_level_0__#4743032L]
Right keys [1]: [__that___index_level_0__#4743106L]
Join type: LeftOuter
Join condition: None

(59) Project [codegen id : 22]
Output [1]: [__this___index_level_0__#4743032L AS __that___index_level_0__#4743159L]
Input [2]: [__this___index_level_0__#4743032L, __that___index_level_0__#4743106L]

(60) SortMergeJoin [codegen id : 23]
Left keys [1]: [__this___index_level_0__#4742823L]
Right keys [1]: [__that___index_level_0__#4743159L]
Join type: LeftOuter
Join condition: None

(61) Project [codegen id : 23]
Output [4]: [__this___index_level_0__#4742823L AS __index_level_0__#4743161L, __this_orderprice#4742829, __this_code#4742838, monotonically_increasing_id() AS __natural_order__#4743185L]
Input [4]: [__this___index_level_0__#4742823L, __this_orderprice#4742829, __this_code#4742838, __that___index_level_0__#4743159L]

(62) Project [codegen id : 23]
Output [4]: [__index_level_0__#4743161L, __natural_order__#4743185L, __this_orderprice#4742829, __this_code#4742838]
Input [4]: [__index_level_0__#4743161L, __this_orderprice#4742829, __this_code#4742838, __natural_order__#4743185L]

(63) Exchange
Input [4]: [__index_level_0__#4743161L, __natural_order__#4743185L, __this_orderprice#4742829, __this_code#4742838]
Arguments: hashpartitioning(__this_code#4742838, 200), ENSURE_REQUIREMENTS, [plan_id=1101827]

(64) ShuffleQueryStage
Output [4]: [__index_level_0__#4743161L, __natural_order__#4743185L, __this_orderprice#4742829, __this_code#4742838]
Arguments: 13

(65) AQEShuffleRead
Input [4]: [__index_level_0__#4743161L, __natural_order__#4743185L, __this_orderprice#4742829, __this_code#4742838]
Arguments: coalesced

(66) Sort [codegen id : 24]
Input [4]: [__index_level_0__#4743161L, __natural_order__#4743185L, __this_orderprice#4742829, __this_code#4742838]
Arguments: [__this_code#4742838 ASC NULLS FIRST, __natural_order__#4743185L ASC NULLS FIRST], false, 0

(67) Window
Input [4]: [__index_level_0__#4743161L, __natural_order__#4743185L, __this_orderprice#4742829, __this_code#4742838]
Arguments: [lag(__this_orderprice#4742829, -1, null) windowspecdefinition(__this_code#4742838, __natural_order__#4743185L ASC NULLS FIRST, specifiedwindowframe(RowFrame, -1, -1)) AS _we0#4743216], [__this_code#4742838], [__natural_order__#4743185L ASC NULLS FIRST]

(68) Project [codegen id : 25]
Output [2]: [__index_level_0__#4743161L AS __that___index_level_0__#4743213L, (-1.0 * (__this_orderprice#4742829 - _we0#4743216)) AS __that_orderprice#4743214]
Input [5]: [__index_level_0__#4743161L, __natural_order__#4743185L, __this_orderprice#4742829, __this_code#4742838, _we0#4743216]

(69) Exchange
Input [2]: [__that___index_level_0__#4743213L, __that_orderprice#4743214]
Arguments: hashpartitioning(__that___index_level_0__#4743213L, 200), ENSURE_REQUIREMENTS, [plan_id=1101974]

(70) ShuffleQueryStage
Output [2]: [__that___index_level_0__#4743213L, __that_orderprice#4743214]
Arguments: 14

(71) AQEShuffleRead
Input [2]: [__that___index_level_0__#4743213L, __that_orderprice#4743214]
Arguments: coalesced

(72) Sort [codegen id : 33]
Input [2]: [__that___index_level_0__#4743213L, __that_orderprice#4743214]
Arguments: [__that___index_level_0__#4743213L ASC NULLS FIRST], false, 0

(73) SortMergeJoin [codegen id : 34]
Left keys [1]: [__this___index_level_0__#4742290L]
Right keys [1]: [__that___index_level_0__#4743213L]
Join type: LeftOuter
Join condition: None

(74) Project [codegen id : 34]
Output [2]: [__this___index_level_0__#4742290L AS __index_level_0__#4743215L, CASE WHEN (CASE WHEN __that_orderprice#4743214 IN (-Infinity,Infinity) THEN false ELSE isnull(__that_orderprice#4743214) END OR isnan(CASE WHEN __that_orderprice#4743214 IN (-Infinity,Infinity) THEN 0.0 ELSE __that_orderprice#4743214 END)) THEN 0.0 ELSE CASE WHEN __that_orderprice#4743214 IN (-Infinity,Infinity) THEN 0.0 ELSE __that_orderprice#4743214 END END AS alpha046#4743272]
Input [3]: [__this___index_level_0__#4742290L, __that___index_level_0__#4743213L, __that_orderprice#4743214]

(75) CollectLimit
Input [2]: [__index_level_0__#4743215L, alpha046#4743272]
Arguments: 1001

(76) Project
Output: []
Input [1]: [symbol#4742236]

(77) AttachDistributedSequence
Input: []
Arguments: distributed_sequence_id#4742252: bigint

(78) Project
Output [1]: [distributed_sequence_id#4742252L AS __this___index_level_0__#4742290L]
Input [1]: [distributed_sequence_id#4742252L]

(79) LocalLimit
Input [1]: [__this___index_level_0__#4742290L]
Arguments: 1001

(80) Exchange
Input [1]: [__this___index_level_0__#4742290L]
Arguments: hashpartitioning(__this___index_level_0__#4742290L, 200), ENSURE_REQUIREMENTS, [plan_id=1101020]

(81) Sort
Input [1]: [__this___index_level_0__#4742290L]
Arguments: [__this___index_level_0__#4742290L ASC NULLS FIRST], false, 0

(82) Project
Output: []
Input [1]: [symbol#4742308]

(83) AttachDistributedSequence
Input: []
Arguments: distributed_sequence_id#4742324: bigint

(84) Project
Output [1]: [distributed_sequence_id#4742324L AS __that___index_level_0__#4742364L]
Input [1]: [distributed_sequence_id#4742324L]

(85) Exchange
Input [1]: [__that___index_level_0__#4742364L]
Arguments: hashpartitioning(__that___index_level_0__#4742364L, 200), ENSURE_REQUIREMENTS, [plan_id=1101021]

(86) Sort
Input [1]: [__that___index_level_0__#4742364L]
Arguments: [__that___index_level_0__#4742364L ASC NULLS FIRST], false, 0

(87) SortMergeJoin
Left keys [1]: [__this___index_level_0__#4742290L]
Right keys [1]: [__that___index_level_0__#4742364L]
Join type: LeftOuter
Join condition: None

(88) Project
Output [1]: [__this___index_level_0__#4742290L]
Input [2]: [__this___index_level_0__#4742290L, __that___index_level_0__#4742364L]

(89) LocalLimit
Input [1]: [__this___index_level_0__#4742290L]
Arguments: 1001

(90) BatchScan finance.sel2_order_20231218163228276
Output [1]: [symbol#4742445]
Format: timelyre
PushedLimit: 0
ReadSchema: struct<symbol:string>
ShivaMasterAddress: 172.18.128.49:8530,172.18.128.50:8530,172.18.128.51:8530
TableName: finance.sel2_order_20231218163228276

(91) Project
Output: []
Input [1]: [symbol#4742445]

(92) AttachDistributedSequence
Input: []
Arguments: distributed_sequence_id#4742461: bigint

(93) Project
Output [1]: [distributed_sequence_id#4742461L AS __this___index_level_0__#4742499L]
Input [1]: [distributed_sequence_id#4742461L]

(94) Exchange
Input [1]: [__this___index_level_0__#4742499L]
Arguments: hashpartitioning(__this___index_level_0__#4742499L, 200), ENSURE_REQUIREMENTS, [plan_id=1101028]

(95) Sort
Input [1]: [__this___index_level_0__#4742499L]
Arguments: [__this___index_level_0__#4742499L ASC NULLS FIRST], false, 0

(96) BatchScan finance.sel2_order_20231218163228276
Output [1]: [symbol#4742517]
Format: timelyre
PushedLimit: 0
ReadSchema: struct<symbol:string>
ShivaMasterAddress: 172.18.128.49:8530,172.18.128.50:8530,172.18.128.51:8530
TableName: finance.sel2_order_20231218163228276

(97) Project
Output: []
Input [1]: [symbol#4742517]

(98) AttachDistributedSequence
Input: []
Arguments: distributed_sequence_id#4742533: bigint

(99) Project
Output [1]: [distributed_sequence_id#4742533L AS __that___index_level_0__#4742573L]
Input [1]: [distributed_sequence_id#4742533L]

(100) Exchange
Input [1]: [__that___index_level_0__#4742573L]
Arguments: hashpartitioning(__that___index_level_0__#4742573L, 200), ENSURE_REQUIREMENTS, [plan_id=1101029]

(101) Sort
Input [1]: [__that___index_level_0__#4742573L]
Arguments: [__that___index_level_0__#4742573L ASC NULLS FIRST], false, 0

(102) SortMergeJoin
Left keys [1]: [__this___index_level_0__#4742499L]
Right keys [1]: [__that___index_level_0__#4742573L]
Join type: LeftOuter
Join condition: None

(103) Project
Output [1]: [__this___index_level_0__#4742499L AS __that___index_level_0__#4742626L]
Input [2]: [__this___index_level_0__#4742499L, __that___index_level_0__#4742573L]

(104) SortMergeJoin
Left keys [1]: [__this___index_level_0__#4742290L]
Right keys [1]: [__that___index_level_0__#4742626L]
Join type: LeftOuter
Join condition: None

(105) Project
Output [1]: [__this___index_level_0__#4742290L]
Input [2]: [__this___index_level_0__#4742290L, __that___index_level_0__#4742626L]

(106) LocalLimit
Input [1]: [__this___index_level_0__#4742290L]
Arguments: 1001

(107) AttachDistributedSequence
Input [2]: [orderprice#4742774, code#4742783]
Arguments: distributed_sequence_id#4742785: bigint

(108) Project
Output [3]: [distributed_sequence_id#4742785L AS __this___index_level_0__#4742823L, orderprice#4742774 AS __this_orderprice#4742829, code#4742783 AS __this_code#4742838]
Input [3]: [distributed_sequence_id#4742785L, orderprice#4742774, code#4742783]

(109) Exchange
Input [3]: [__this___index_level_0__#4742823L, __this_orderprice#4742829, __this_code#4742838]
Arguments: hashpartitioning(__this___index_level_0__#4742823L, 200), ENSURE_REQUIREMENTS, [plan_id=1101039]

(110) Sort
Input [3]: [__this___index_level_0__#4742823L, __this_orderprice#4742829, __this_code#4742838]
Arguments: [__this___index_level_0__#4742823L ASC NULLS FIRST], false, 0

(111) BatchScan finance.sel2_order_20231218163228276
Output [1]: [symbol#4742841]
Format: timelyre
PushedLimit: 0
ReadSchema: struct<symbol:string>
ShivaMasterAddress: 172.18.128.49:8530,172.18.128.50:8530,172.18.128.51:8530
TableName: finance.sel2_order_20231218163228276

(112) Project
Output: []
Input [1]: [symbol#4742841]

(113) AttachDistributedSequence
Input: []
Arguments: distributed_sequence_id#4742857: bigint

(114) Project
Output [1]: [distributed_sequence_id#4742857L AS __that___index_level_0__#4742897L]
Input [1]: [distributed_sequence_id#4742857L]

(115) Exchange
Input [1]: [__that___index_level_0__#4742897L]
Arguments: hashpartitioning(__that___index_level_0__#4742897L, 200), ENSURE_REQUIREMENTS, [plan_id=1101040]

(116) Sort
Input [1]: [__that___index_level_0__#4742897L]
Arguments: [__that___index_level_0__#4742897L ASC NULLS FIRST], false, 0

(117) SortMergeJoin
Left keys [1]: [__this___index_level_0__#4742823L]
Right keys [1]: [__that___index_level_0__#4742897L]
Join type: LeftOuter
Join condition: None

(118) Project
Output [3]: [__this___index_level_0__#4742823L, __this_orderprice#4742829, __this_code#4742838]
Input [4]: [__this___index_level_0__#4742823L, __this_orderprice#4742829, __this_code#4742838, __that___index_level_0__#4742897L]

(119) BatchScan finance.sel2_order_20231218163228276
Output [1]: [symbol#4742978]
Format: timelyre
PushedLimit: 0
ReadSchema: struct<symbol:string>
ShivaMasterAddress: 172.18.128.49:8530,172.18.128.50:8530,172.18.128.51:8530
TableName: finance.sel2_order_20231218163228276

(120) Project
Output: []
Input [1]: [symbol#4742978]

(121) AttachDistributedSequence
Input: []
Arguments: distributed_sequence_id#4742994: bigint

(122) Project
Output [1]: [distributed_sequence_id#4742994L AS __this___index_level_0__#4743032L]
Input [1]: [distributed_sequence_id#4742994L]

(123) Exchange
Input [1]: [__this___index_level_0__#4743032L]
Arguments: hashpartitioning(__this___index_level_0__#4743032L, 200), ENSURE_REQUIREMENTS, [plan_id=1101046]

(124) Sort
Input [1]: [__this___index_level_0__#4743032L]
Arguments: [__this___index_level_0__#4743032L ASC NULLS FIRST], false, 0

(125) BatchScan finance.sel2_order_20231218163228276
Output [1]: [symbol#4743050]
Format: timelyre
PushedLimit: 0
ReadSchema: struct<symbol:string>
ShivaMasterAddress: 172.18.128.49:8530,172.18.128.50:8530,172.18.128.51:8530
TableName: finance.sel2_order_20231218163228276

(126) Project
Output: []
Input [1]: [symbol#4743050]

(127) AttachDistributedSequence
Input: []
Arguments: distributed_sequence_id#4743066: bigint

(128) Project
Output [1]: [distributed_sequence_id#4743066L AS __that___index_level_0__#4743106L]
Input [1]: [distributed_sequence_id#4743066L]

(129) Exchange
Input [1]: [__that___index_level_0__#4743106L]
Arguments: hashpartitioning(__that___index_level_0__#4743106L, 200), ENSURE_REQUIREMENTS, [plan_id=1101047]

(130) Sort
Input [1]: [__that___index_level_0__#4743106L]
Arguments: [__that___index_level_0__#4743106L ASC NULLS FIRST], false, 0

(131) SortMergeJoin
Left keys [1]: [__this___index_level_0__#4743032L]
Right keys [1]: [__that___index_level_0__#4743106L]
Join type: LeftOuter
Join condition: None

(132) Project
Output [1]: [__this___index_level_0__#4743032L AS __that___index_level_0__#4743159L]
Input [2]: [__this___index_level_0__#4743032L, __that___index_level_0__#4743106L]

(133) SortMergeJoin
Left keys [1]: [__this___index_level_0__#4742823L]
Right keys [1]: [__that___index_level_0__#4743159L]
Join type: LeftOuter
Join condition: None

(134) Project
Output [4]: [__this___index_level_0__#4742823L AS __index_level_0__#4743161L, __this_orderprice#4742829, __this_code#4742838, monotonically_increasing_id() AS __natural_order__#4743185L]
Input [4]: [__this___index_level_0__#4742823L, __this_orderprice#4742829, __this_code#4742838, __that___index_level_0__#4743159L]

(135) Project
Output [4]: [__index_level_0__#4743161L, __natural_order__#4743185L, __this_orderprice#4742829, __this_code#4742838]
Input [4]: [__index_level_0__#4743161L, __this_orderprice#4742829, __this_code#4742838, __natural_order__#4743185L]

(136) Exchange
Input [4]: [__index_level_0__#4743161L, __natural_order__#4743185L, __this_orderprice#4742829, __this_code#4742838]
Arguments: hashpartitioning(__this_code#4742838, 200), ENSURE_REQUIREMENTS, [plan_id=1101057]

(137) Sort
Input [4]: [__index_level_0__#4743161L, __natural_order__#4743185L, __this_orderprice#4742829, __this_code#4742838]
Arguments: [__this_code#4742838 ASC NULLS FIRST, __natural_order__#4743185L ASC NULLS FIRST], false, 0

(138) Window
Input [4]: [__index_level_0__#4743161L, __natural_order__#4743185L, __this_orderprice#4742829, __this_code#4742838]
Arguments: [lag(__this_orderprice#4742829, -1, null) windowspecdefinition(__this_code#4742838, __natural_order__#4743185L ASC NULLS FIRST, specifiedwindowframe(RowFrame, -1, -1)) AS _we0#4743216], [__this_code#4742838], [__natural_order__#4743185L ASC NULLS FIRST]

(139) Project
Output [2]: [__index_level_0__#4743161L AS __that___index_level_0__#4743213L, (-1.0 * (__this_orderprice#4742829 - _we0#4743216)) AS __that_orderprice#4743214]
Input [5]: [__index_level_0__#4743161L, __natural_order__#4743185L, __this_orderprice#4742829, __this_code#4742838, _we0#4743216]

(140) Exchange
Input [2]: [__that___index_level_0__#4743213L, __that_orderprice#4743214]
Arguments: hashpartitioning(__that___index_level_0__#4743213L, 200), ENSURE_REQUIREMENTS, [plan_id=1101063]

(141) Sort
Input [2]: [__that___index_level_0__#4743213L, __that_orderprice#4743214]
Arguments: [__that___index_level_0__#4743213L ASC NULLS FIRST], false, 0

(142) SortMergeJoin
Left keys [1]: [__this___index_level_0__#4742290L]
Right keys [1]: [__that___index_level_0__#4743213L]
Join type: LeftOuter
Join condition: None

(143) Project
Output [2]: [__this___index_level_0__#4742290L AS __index_level_0__#4743215L, CASE WHEN (CASE WHEN __that_orderprice#4743214 IN (-Infinity,Infinity) THEN false ELSE isnull(__that_orderprice#4743214) END OR isnan(CASE WHEN __that_orderprice#4743214 IN (-Infinity,Infinity) THEN 0.0 ELSE __that_orderprice#4743214 END)) THEN 0.0 ELSE CASE WHEN __that_orderprice#4743214 IN (-Infinity,Infinity) THEN 0.0 ELSE __that_orderprice#4743214 END END AS alpha046#4743272]
Input [3]: [__this___index_level_0__#4742290L, __that___index_level_0__#4743213L, __that_orderprice#4743214]

(144) CollectLimit
Input [2]: [__index_level_0__#4743215L, alpha046#4743272]
Arguments: 1001

(145) AdaptiveSparkPlan
Output [2]: [__index_level_0__#4743215L, alpha046#4743272]
Arguments: isFinalPlan=true
```
## 相关概念
- [[执行计划之Exchange]]
- [[为什么要进行AQEShuffleRead]]
- [[为什么要进行SortMergeJoin]]

## 优化思路
影响一个Spark作业性能的因素，主要还是
- 代码开发 #leopard/性能调优 [[代码开发如何调优]]
- 资源参数 #leopard/性能调优  [[官方如何推荐对资源参数进行调优]]
- 数据倾斜 #leopard/性能调优 [[如何解决数据倾斜问题]]
参考文章：
- https://tech.meituan.com/2016/05/12/spark-tuning-pro.html
- https://tech.meituan.com/2016/04/29/spark-tuning-basic.html
### 对series进行shift
目前只支持对dataframe进行shift，然后select需要的列，是否可以只对需要的列进行shift？
只挑选了两列，对dataframe shift和对其中一列shift差别不大。
### groupby支持列裁剪
df为code列和orderprice列，groupby使用到的列已经是最少的，不需要再支持groupby列裁剪。
在alpha046中，我们只需要操作
```python
df = df[["code", "orderprice"]]
df.groupby("code")["orderprice"]
```
这在执行计划中会
1. 先BatchScan获取groupby后的所有数据
2. 然后执行Project挑选其中的orderprice列。

如果直接BatchScan只读取需要的orderprice列，就减少了其他列的读取时间，而读取时间是和列数成比例的。

### persist/cache 中间结果
没有需要重复计算的中间结果。

由于多次对相同的groupby结果进行不同的操作，是否可以考虑将groupby后的结果缓存到内存中，避免重复加载？
- [[为什么要使用persist或cache]] #leopard/优化
本身就是按照code分区进行存储的，还需要groupby操作吗？
### 资源参数调优
- [[spark任务gc耗时多，如何设置资源参数]]
- [ ] 对于I/O密集型，[[使用Kryo序列化]]来替代Java序列化 #leopard/性能调优 
- 设置tasks数量是(excutor-nums * cpu-core-nums) 的2倍或3倍
## 实验统计
### 48s
http://crux-52:4280/SQL/execution/?id=16421
```python
def alpha046(df: ps.DataFrame):
    df = df[["code", "orderprice"]]
    delay_20 = df.groupby("code").shift(20)["orderprice"]
    delay_10 = df.groupby("code").shift(10)["orderprice"]
    df["cond"] = (delay_20 - delay_10) / 10 - (delay_10 - df["orderprice"]) / 10
    df["alpha046"] = -1 * df.groupby("code").diff(1)["orderprice"]
    df.loc[df["cond"] < 0, "alpha046"] = 1
    df.loc[df["cond"] > 0.25, "alpha046"] = -1
    return df["alpha046"].replace([-np.inf, np.inf], 0).fillna(value=0)
```
### 35s
http://crux-52:4280/SQL/execution/?id=16442
- 不开编译优化的话，编译时间需要27s。
- compile 8.933296203613281 s, execute 35.63214182853699 s
```python
def alpha046(df: ps.DataFrame):
    df = df[
        ["code", "orderprice"]
    ]  # 存储scan了三列，多了一个ts列，是否可以不scan ts列？
    delay_20 = df.groupby("code").shift(20)["orderprice"]  # scan
    delay_10 = df.groupby("code").shift(10)["orderprice"]  # scan
    cond = ((delay_20 - delay_10) / 10 - (delay_10 - df["orderprice"]) / 10).rename(
        "cond"
    )
    alpha046 = (-1 * df.groupby("code").diff(1)["orderprice"]).rename(
        "alpha046"
    )  # scan
    tmp = cond.to_frame().join(alpha046)
    tmp.loc[tmp["cond"] < 0, "alpha046"] = 1
    tmp.loc[tmp["cond"] > 0.25, "alpha046"] = -1
    return tmp["alpha046"].replace([-np.inf, np.inf], 0).fillna(value=0)
```

- 第一行只挑选code和orderprice是否下推到存储？
   - 查看scanFields关键字对应的存储日志
   - 发现确实下推到了存储
- [ ] [[select code和orderprice列，存储为什么会scan code、orderprice、ts三列]] #leopard/存储
