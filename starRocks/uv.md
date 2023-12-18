# 数仓理论SCD(缓慢变化维度)
### [go back](/x2q/starRocks/starRocks)      
### [go home](/x2q)    

## starRocks 求uv
## 使用 bitmap
https://docs.starrocks.io/zh/docs/3.2/using_starrocks/Using_bitmap/
## 使用 hll
https://docs.starrocks.io/zh/docs/3.2/using_starrocks/Using_HLL/
https://juejin.cn/post/6844903785744056333

## 补充
### 调和平均数
调和平均数比平均数的好处就是不容易受到大的数值的影响
以下例子
"""
求平均工资:
A的是1000/月，B的30000/月。采用平均数的方式就是： (1000 + 30000) / 2 = 15500
采用调和平均数的方式就是： 2/(1/1000 + 1/30000) ≈ 1935.484
"""

  

#### 联系邮箱 xxx_xxx@aliyun.com

