## 原内容
```
=====  transwarp_df.pandas.Index.factorize

将对象编码为枚举类型或分类变量。当重要的是识别不同的值时，此方法对于获取数组的数字表示很有用。

====== 参数

* sort: bool, 默认值为 True

* na_sentinel: int 或 None, 默认值为 -1；标记“not found”的值，如果没有，将不会从值的唯一性中删除 NaN

====== 返回

* codes: Series 或 Index；一个系列或索引，它是 uniques 的索引器， uniques.take(codes) 将具有与 values 相同的值

* uniques：pd.Index；唯一的有效值

====== 例如

[source,python]
----
import transwarp_df.pandas as ps
psser = ps.Series(['b', None, 'a', 'c', 'b'])
codes, uniques = psser.factorize()
TODO:不支持的数据类型 
codes

uniques
----

[blue]#运行结果：#

[source,python]
----
0    1
1   -1
2    0
3    2
4    1
dtype: int32

Index(['a', 'b', 'c'], dtype='object')
----
[source,python]
----
codes, uniques = psser.factorize(na_sentinel=None)
TODO:不支持的数据类型 
codes

uniques
----

[blue]#运行结果：#

[source,python]
----
0    1
1    3
2    0
3    2
4    1
dtype: int32

Index(['a', 'b', 'c', None], dtype='object')
----
[source,python]
----
codes, uniques = psser.factorize(na_sentinel=-2)
TODO:不支持的数据类型
codes

uniques
----

[blue]#运行结果：#

[source,python]
----
0    1
1   -2
2    0
3    2
4    1
dtype: int32

Index(['a', 'b', 'c'], dtype='object')
----

对于索引，

[source,python]
----
import transwarp_df.pandas as ps
psidx = ps.Index(['b', None, 'a', 'c', 'b'])
codes, uniques = psidx.factorize()
TODO:不支持的数据类型
codes 

uniques
----

[blue]#运行结果：#

[source,python]
----
Int64Index([1, -1, 0, 2, 1], dtype='int64')

Index(['a', 'b', 'c'], dtype='object')
----


```