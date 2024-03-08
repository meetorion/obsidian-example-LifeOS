## 删除的内容
```
=====  transwarp_df.pandas.CategoricalIndex.reorder_categories

按照 new_categories 中的指定重新排列类别，new_categories 需包括所有旧类别，而不包括新类别项。

====== 参数

* new_categories: Index-like；按新顺序排列的类别

* ordered: bool, 选填；是否将分类信息视为有序分类信息，如果未给出，则不更改有序信息

* inplace: bool, 默认值为 False；是否就地对类别重新排序，或返回一份经过重新排序的类别副本

====== 返回

* catCategorical: Index 或 None；已删除类别的分类，如果 `inplace=True` 则为 "None"

====== 加注

ValueError: 如果新类别不包含所有旧类别项目或任何新类别项目

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
idx.reorder_categories(['c', 'b', 'a'])  
TODO:不支持的数据类型
----

[blue]#运行结果：#

[source,python]
----
CategoricalIndex(['a', 'b', 'b', 'c', 'c', 'c'],
                 categories=['c', 'b', 'a'], ordered=False, dtype='category')
----


```