# 字符串中最长子字符串
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)   

## 题目
+ 力扣 https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/
+ 给定一个数组 prices ，它的第 i 个元素 prices[i] 表示一支给定股票第 i 天的价格。
你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。


```java
class Solution {
    public int maxProfit(int[] prices) {
        int max=0;
        for(int i=0;i<prices.length;i++){
            for(int j=i+1;j<prices.length-1;j++){
                max=Math.max(max,prices[j]-prices[i]);
            }
        }
        return max;

    }
}
```

#### 联系邮箱 xxx_xxx@aliyun.com