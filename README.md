# 时间序列解析nginx日志


服务器上，nginx的日志很常见


网络服务日志分析，是常见的一种分析。可以根据之前的访问量，预估之后的访问量，然后根据访问量决定服务器是增加配置和带宽，还是减少。并且结合用户注册以及支付还可以分析用户转化率和支付率。


我们现在就用时间序列分析一下nginx日志


# 数据采集


## nginx配置


打开nginx配置，可以找到记录日志的部分


```conf
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```


总共我们统计了这些内容——

 key | value
----- | -----
remote_addr | 远程地址
remote_user | 用户
time_local | 服务器时间
request | 请求地址
status | 状态
body_bytes_sent | 页面尺寸
http_referer | 来源
http_user_agent | 设备
http_x_forwarded_for | 真实地址



```py
import re

pattern = re.compile(r'\[(\w+\/\w+/\w+):(\w+:\w+:\w+)\s(\+\w+)\]')
s = '[18/Mar/2018:20:05:08 +0800]'
res = pattern.match(s)
res.group(1) + ' ' + res.group(2) + res.group(3)
```


```py
import re

pattern = re.compile(r'\[(\w+\/\w+/\w+):(\w+:\w+:\w+)\s(\+\w+)\]')

def to_datetime(s):
    res = pattern.match(s)
    return pd.to_datetime(res.group(1) + ' ' + res.group(2) + res.group(3))

df['date'] = df['time_local'] + ' ' + df['time_zone']
df['date'] = df['date'].apply(to_datetime)
```


```py
from statsmodels.graphics import tsaplots
fig, ax = plt.subplots(figsize=(15, 6))
tsaplots.plot_acf(count, lags=120, ax=ax)
```

```py
from statsmodels.tsa.stattools import arma_order_select_ic
orders = arma_order_select_ic(diff, ic=['aic', 'bic'], trend='nc', max_ar=8, max_ma=6)
````


```py
from statsmodels.tsa import arima_model

arima516 = arima_model.ARIMA(count, (5, 1, 6)).fit()
arima413 = arima_model.ARIMA(count, (4, 1, 3)).fit()
```


AIC:赤池信息准则（AkaikeInformation Criterion，AIC）


在一般的情况下，AIC可以表示为： AIC=2k-2ln(L)


BIC:贝叶斯信息准则（Bayesian Information Criterion，BIC）


计算公式：BIC=-2 ln(L) +ln(n)*k
- L：似然函数
- n：样本大小
- K：参数数量
与AIC一样是对模型的拟合效果进行评价的一个指标，BIC值越小，则模型对数据的拟合越好。

