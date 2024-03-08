# 如何划分Stage
Spark的DAGScheduler根据RDD之间是否具有宽依赖来划分Stage，有宽依赖则划分新的Stage。
Spark的每个Job根据RDD的依赖关系（父RDD -> 子RDD）来划分Stage，从最后一个RDD开始，将其作为第一个Stage。
如果父RDD和子RDD是窄依赖关系，那么就将父RDD加入子RDD所在的Stage；
如果父RDD和子RDD是宽依赖关系，那么就将父RDD作为新的Stage。

## DAGScheduler
RDD之间的依赖关系是由DAGScheduler来分析的，分析完后将Job划分为多个Stage。

## TaskScheduler
每个Stage会生成一个TaskSet并提交给TaskScheduler，由TaskScheduler负责分发task到worker中执行。

## 窄依赖
父RDD的分区最多被子RDD的一个分区使用，则说明这两个RDD是窄依赖。

## 宽依赖
父RDD的分区会被子RDD的多个分区使用，则说明这两个RDD之间是宽依赖。

## Stage的task数量
每个Stage拥有一个TaskSet，其中的tasks是并行执行的;
每个Stage里面的tasks数量由最后一个加入该Stage的RDD的分区数决定。
