# 执行计划之Exchange
## 为什么会Exchange
Exchange指任务中的部分数据需要被发送或接收到其他分区进行处理，涉及到数据分片之间的网络通信。
需要Exchange的场景：
- groupByKey：groupByKey需要将属于同一个key的所有元素被发送到同一个节点

## 如何优化Exchange
Exchange描述的是Spark的Shuffle过程，包括shuffle write和shuffle read两个阶段。
- shuffle write: Spark任务会产生多个输出文件，然后将这些文件写入磁盘
- shuffle read：后续任务从已排序的输出文件读取属于自己的文件

## 相关链接
- [[为什么要进行AQEShuffleRead]]
