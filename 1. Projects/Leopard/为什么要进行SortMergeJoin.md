# 为什么要进行SortMergeJoin

## 将两个DataFrame进行Join操作
当对某个dataframe新增一列时，可能导致SortMergeJoin操作。如：
```python
df["delay_20"] = df.groupby("code")["orderprice"].shift(20)
```
那么，如果可以不赋值就可以省掉SortMergeJoin过程。
