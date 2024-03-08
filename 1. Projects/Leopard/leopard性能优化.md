# leopard性能优化

## alpha101系列算法
- [ ] [[alpha046]]
- [ ] [[alpha012]]
- [ ] [[alpha041]]
- [ ] [[alpha049]]
- [ ] [[alpha051]]
- [ ] [[alpha053]]
- [ ] [[alpha054]]
- [ ] [[alpha101]]

## 问题
- [ ] [[为什么是16个task,15个不行吗]]
- [ ] [[为什么要进行AQEShuffleRead]]
- [x] [[Spark是如何划分Job的]]
- [ ] [[打jstack生成热点图，找出Spark运行较慢的函数]]

## 预期性能估算
整个过程可以分为读和算。
读依赖于storage的性能，算依赖于Spark的性能。
那么应该统计每个Stage的耗时，判断是否合理，即确定该Stage在干什么，干这种事情花费的时间是否合理。

## 通用优化
- [ ] [[是否可以去掉AttachDistributedSequence阶段，在数据源生成唯一id]]
- [ ] [[是否可以复用Groupby shuffle write的结果，使得只Scan一次]]
- [ ] [[是否可以去掉窗口函数的orderby操作]]
