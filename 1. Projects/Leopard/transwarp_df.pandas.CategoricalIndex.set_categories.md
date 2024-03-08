## 删除的内容
```
=====  transwarp_df.pandas.CategoricalIndex.set_categories

将类别设置为指定的 new_categories。new_categories 可以包含新类别（将导致未使用的类别）或删除旧类别（将导致值设置为 NaN）。如果 rename==True，类别将被简单重命名。

====== 参数

* new_categories: Index-like；按新顺序排列的类别

* ordered：bool, 默认值为 False；是否将分类信息视为有序分类信息，如果未给出，则不更改有序信息

* rename：bool, 默认值为 False；是否应将 new_categories视为旧类别的重命名或重新排序的类别

* inplace: bool, 默认值为 False；是否就地对类别重新排序，或返回一份经过重新排序的类别副本

====== 返回

* 类别重新排序的序列，如果存在则为None

====== 加注

* ValueError: 如果 new_categories 未验证为类别

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
idx.set_categories(['b', 'c'])  
----

[blue]#运行结果：#

[source,python]
----
CategoricalIndex([nan, 'b', 'b', 'c', 'c', 'c'],
                 categories=['b', 'c'], ordered=False, dtype='category')
----

[source,python]
----
idx.set_categories([1, 2, 3], rename=True)
TODO:不支持
----

[blue]#运行结果：#

[source,python]
----
CategoricalIndex([1, 2, 2, 3, 3, 3], categories=[1, 2, 3], ordered=False, dtype='category')
----


```