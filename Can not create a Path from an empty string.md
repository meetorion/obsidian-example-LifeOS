## 复现
```python
import numpy as np
import pandas as pd
import transwarp_df.pandas as ps
from transwarp.timelyre.timelyre_public import DatabaseConn

DB = "default"
TIMECHORD_URL = "172.18.126.75:9998"
JDBC_URL = "jdbc:hive2://172.18.126.75:10000"

# guardianToken 方式建立连接：
conn = DatabaseConn(
    TIMECHORD_URL, JDBC_URL, DB, auth_type="guardian", token="vLaSK3FXMCqtLvk3sUqK-TDH"
)


local_tz = "Asia/Shanghai"
trades = pd.DataFrame(
    {
        "col_time": [
            pd.Timestamp("2023-01-01 09:30:00.000000001", tz=local_tz),
            pd.Timestamp("2023-01-01 09:40:00.000000002", tz=local_tz),
            pd.Timestamp("2023-01-01 09:50:00.000000003", tz=local_tz),
            pd.Timestamp("2023-01-01 09:30:00.000000001", tz=local_tz),
            pd.Timestamp("2023-01-01 09:50:00.000000003", tz=local_tz),
        ],
        "ticker": ["MSFT", "MSFT", "GOOG", "GOOG", "AAPL"],
        "price": [51.95, 51.95, 720.77, 720.92, 98.0],
        "quantity": [75, 155, 100, 100, 100],
    }
)
print(f"trades{trades}\n")
table_name = "trades_test"
df_schema = [
    ["col_time", "timestamp"],
    ["ticker", "string"],
    ["price", "double"],
    ["quantity", "int"],
]

conn.drop_table(table_name)
conn.create_table(
    table_name,
    tblType="timelyre",
    schema=df_schema,
    tags="ticker",
    timecol="col_time",
    shard_group_duration=31536000,
)
conn.upload_df(trades, df_schema, table_name=table_name)
df_load = conn.load_df(table_name)


####save_df_as_table csv
conn.drop_table(table_name + "_csv")
conn.create_table(table_name + "_csv", tblType="csv", schema=df_schema)
conn.save_df_as_table(
    df_load, table_name=table_name + "_csv", table_type="csv", mode="append"
)
df_copy_csv = conn.load_df(table_name + "_csv", table_type="csv")

####save_df_as_table 保存为 parquet
conn.drop_table(table_name + "_parquet")
conn.create_table(table_name + "_parquet", tblType="parquet", schema=df_schema)
conn.save_df_as_table(
    df_load, table_name=table_name + "_parquet", table_type="parquet", mode="append"
)
df_copy_parquet = conn.load_df(table_name + "_parquet", table_type="parquet")
print(f"df_copy_parquet{df_copy_parquet}\n")

```
## 报错信息
```
transwarp_df.errors.exceptions.connect.IllegalArgumentException: Can not create a Path from an empty string
```
## 根因追朔
path参数没有传导下去，是接口变动的影响。