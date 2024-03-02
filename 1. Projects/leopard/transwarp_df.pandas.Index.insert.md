## 原内容
```
=====  transwarp_df.pandas.Index.insert

使新索引在位置插入新项目。

====== 参数

* loc: int

* item: object

====== 返回

* new_index: Index

====== 例如
[source,python]
----
import transwarp_df.pandas as ps
psidx = ps.Index([1, 2, 3, 4, 5])
psidx.insert(3, 100) 
TODO:结果有bug
----

[blue]#运行结果：#

[source,python]
----
Int64Index([1, 2, 3, 100, 4, 5], dtype='int64')
----

对于负值，
[source,python]
----
import transwarp_df.pandas as ps
psidx = ps.Index([1, 2, 3, 4, 5])
psidx.insert(-3, 100)  
TODO:结果有bug
----

[blue]#运行结果：#

[source,python]
----
Int64Index([1, 2, 100, 3, 4, 5], dtype='int64')
----


```