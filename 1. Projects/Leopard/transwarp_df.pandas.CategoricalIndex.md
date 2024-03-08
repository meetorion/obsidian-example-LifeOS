## 原内容(删除的内容)
```
对于系列，
[source,python]
----
import transwarp_df.pandas as ps
s = ps.Series(["a", "b", "c", "a", "b", "c"], index=[10, 20, 30, 40, 50, 60])
ps.CategoricalIndex(s)
TODO:不支持
----

[blue]#运行结果：#

[source,python]
----
CategoricalIndex(['a', 'b', 'c', 'a', 'b', 'c'],
                 categories=['a', 'b', 'c'], ordered=False, dtype='category')
----

对于索引，
[source,python]
----
import transwarp_df.pandas as ps
idx = ps.Index(["a", "b", "c", "a", "b", "c"])
ps.CategoricalIndex(idx)  
TODO:不支持
----

[blue]#运行结果：#

[source,python]
----
CategoricalIndex(['a', 'b', 'c', 'a', 'b', 'c'],
                 categories=['a', 'b', 'c'], ordered=False, dtype='category')
----


```