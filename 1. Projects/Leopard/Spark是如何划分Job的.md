# Spark是如何划分Job的
Spark有两种类型的动作：transformations和actions。
每当遇到一个Action时，就创建一个Job。

## transformations
transformations会创建新数据集，并应用于原始数据集。
这些操作是延迟评估的，等待调用action时候一并执行。

## actions
actions会对数据集进行计算并生成结果，如collect()、count()。
### 哪些操作是actions的
