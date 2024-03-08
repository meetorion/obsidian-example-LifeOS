# 是否可以去掉窗口函数的orderby操作
数据源本身是按照ts排序存储的，那么是否可以保证groupby操作shuffle write到磁盘文件的数据也是按照ts有序的呢？

如果保证了shuffle write到磁盘的数据是ts有序的，那么是否可以省掉Window里面的OrderBy。
目前窗口函数的window定义是按照自然序排序的，有序了还需要吗，或者还会去sort吗？
```python
window = (
    Window.partitionBy(*part_cols)
    .orderBy(NATURAL_ORDER_COLUMN_NAME)
    .rowsBetween(-periods, -periods)
)
```
