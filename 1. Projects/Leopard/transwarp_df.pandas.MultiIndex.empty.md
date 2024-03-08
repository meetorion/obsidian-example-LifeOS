## 删除的内容
```
=====  transwarp_df.pandas.MultiIndex.empty

如果当前对象为空，返回true。否则，返回false。

====== 例如

[source,python]
----
import transwarp_df.pandas as ps
ps.range(10).id.empty
TODO:未实现
----
[blue]#运行结果：#
[source,python]
----
False
----
[source,python]
----
import transwarp_df.pandas as ps
ps.range(0).id.empty
----
[blue]#运行结果：#
[source,python]
----
True
----
[source,python]
----
import transwarp_df.pandas as ps
ps.DataFrame({}, index=list('abc')).index.empty
----
[blue]#运行结果：#
[source,python]
----
False
----


```