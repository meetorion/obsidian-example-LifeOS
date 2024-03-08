# 使用Kryo序列化
## SparkConf中添加配置
```python
conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
```
但connect模式无法进行该设置。
## 在spark.env配置文件中指定
![[Pasted image 20240305145033.png]]
## 怎么判断Kryo序列化有没有生效？
在Spark web ui页面中查看Envrionment中确实存在对应key和value.

## 什么时候有用？

