# alpha012
## 代码开发
- pandas版本或初始版本如下：
```python
def alpha012(df : ps.DataFrame):
    df['temp1'] = df.groupby('code').diff(1)['volume']
    df.loc[df['temp1']>0,'temp1'] = 1
    df.loc[df['temp1']<0,'temp1'] = -1
    df['temp2'] = -1 * df.groupby('code').diff(1)['close']
    alpha012 = df['temp1'] * df['temp2']
    return alpha012.replace([-np.inf, np.inf], 0).fillna(value=0)
```
- 在没有优化代码的情况下，性能为：
