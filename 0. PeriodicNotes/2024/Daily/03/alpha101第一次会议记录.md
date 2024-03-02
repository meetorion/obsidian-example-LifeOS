## apply
1. 迭代load_df code，不光是apply，先给个for循环迭代
2. 落临时表
3. apply
## 基本性能
几个case 性能指标，为什么是这个指标，每个阶段耗时，是否合理，后续如何优化。

## alpha101 
1. 跑起来
2. 对
3. 性能（对能跑起来的马上调性能）
## 数据
5s 高频 
分钟 基准 
resample转高频
一个月 全A ticket 3s  5s
一天四千万
4 * 25 = 10乙 
## 结论
1. 优化已有的能跑通且正确的算法case（统计每个阶段耗时，理解为什么这个阶段耗时这么多，尝试去优化） 
2. 修复groupby(as_index=False)失败的bug
3. 支持for循环迭代GroupbyDataFrame
