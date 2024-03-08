## 原内容
```
=====  transwarp_df.pandas.Index.empty

如果当前对象为空，则返回 true；否则返回 false。

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
TODO:未实现
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