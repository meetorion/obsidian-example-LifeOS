## 删除的内容
```
=====  transwarp_df.pandas.CategoricalIndex.remove_categories

删除指定的类别，removals 必须包含在旧类别中，已删除类别中的值将设置为 NaN。

====== 参数

* removals: category 或 list of categories；应该删除的类别

* inplace: bool, 默认值为 False；是就地删除类别，还是返回一个已删除类别的分类副本

====== 返回

* CategoricalIndex 或 None: 已删除类别的分类，如果 inplace=True 则为None

====== 加注

* ValueError: 如果删的内容除不包含在类别中

====== 例如
[source,python]
----
import transwarp_df.pandas as ps
idx = ps.CategoricalIndex(list("abbccc"))
idx    
----

[blue]#运行结果：#

[source,python]
----
CategoricalIndex(['a', 'b', 'b', 'c', 'c', 'c'],
                 categories=['a', 'b', 'c'], ordered=False, dtype='category')
----

[source,python]
----
idx.remove_categories('b')    
TODO:不支持的数据类型
----

[blue]#运行结果：#

[source,python]
----
CategoricalIndex(['a', nan, nan, 'c', 'c', 'c'],
                 categories=['a', 'c'], ordered=False, dtype='category')
----

=====  transwarp_df.pandas.CategoricalIndex.remove_unused_categories

删除不使用的类别。

====== 参数

* inplace: bool, 默认值为 False；是就地删除未使用的类别，还是在删除未使用的类别后返回该分类的副本

====== 返回

* cat: CategoricalIndex 或 None；删除未使用的类别，如果 inplace=True 则为None

====== 例如
[source,python]
----
import transwarp_df.pandas as ps
idx = ps.CategoricalIndex(list("abbccc"), categories=['a', 'b', 'c', 'd'])
idx  
----

[blue]#运行结果：#

[source,python]
----
CategoricalIndex(['a', 'b', 'b', 'c', 'c', 'c'],
                 categories=['a', 'b', 'c', 'd'], ordered=False, dtype='category')
----

[source,python]
----
idx.remove_unused_categories()  
TODO:不支持的数据类型
----

[blue]#运行结果：#

[source,python]
----
CategoricalIndex(['a', 'b', 'b', 'c', 'c', 'c'],
                 categories=['a', 'b', 'c'], ordered=False, dtype='category')
----


```