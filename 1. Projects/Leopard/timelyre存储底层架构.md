# timelyre存储底层架构

## 内存
内存中保持的主要是索引结构和部分数据。
进行compaction时也会导致内存占用变高。

## 索引的维护和更新
当写入不同的标签组合数据时，索引就会进行一次更新。
举个例子，当以"code"作为索引列时，插入一条"code=0"的数据时，索引会更新，记录"code=0"的被写入的tsm文件（磁盘文件）的位置。
如，插入3条记录：
```
code=0 ... 索引更新,新增
code=1 ... 索引更新，新增
code=1 ... 索引不更新
code=0 ... 索引不更新
code=2 ... 索引更新，新增
```
