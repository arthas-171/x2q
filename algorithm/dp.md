# 动态规划
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)   
## 最长递增子序列
求最长递增子序列的长度, 例如nums=[10,9,2,5,3,7,101,18] 最长递增子序列是[2,3,7,101] 长度是4,注意子序列不要求连续, 
并且最长的不一定是唯一的,可能有并列,但是只要求长度即可 不用打印明细

### 核心思想
弄个等长的数组(dp)里面存储,以原来数组(nums)某一个元素结尾的递增子序列的长度是多少, 然后在计算这个dp里面每个长度值是多少,
注意因为子序列是递增的, 那么最后一个元素一定是序列里面最大的, 因此我们只需要比较 子序列里面最后一个元素与当前元素的大小即可,
如果大于,就在这个元素对应的长度上面+1,无论有几个大于的 通通+1,之后再取出最大的就行了;

### 举个例子
+ 数组nums是 -------------------[1,4,3,4,2]
+ 用来存放递增子序列长度的数组db就是 [1,2,2,3,2]
+ nums[3]对应的值是4;db[3]对应的值是3,他的含义是 在原来数组中以下标是3对应值是4结尾的子序列的长度是3                
+ 根据dp这种存储的数据可以得知, dp里面最大的数就是最长的递增子序列的长度
  + 求这个的话代码如下
```java
int res = 0;
for (int i = 0; i < dp.length; i++) {
res = Math.max(res, dp[i]);
}
return res;
```
+ 那么最核心的问题是 怎么计算每个dp[i]的值是多少,其实我们直接就拿每个i下标对应的nums[i]的值去比较就行了, 如果大于nums[i]的话对应的dp[i]就+1,
+ 因为可能有很多个大于的就从这些里面取出最大的就行
```java

for (int i = 0; i < nums.length; i++) {
       // 拿到nums里面的每个元素和他前面的元素都比一遍, 两层for循环 时间复杂的n*n
        for (int j = 0; j < i; j++) {
        // 寻找 nums[0..j-1] 中比 nums[i] 小的元素
        if (nums[i] > nums[j]) {
            // 把 nums[i] 接在后面，即可形成长度为 dp[j] + 1，
            // 且以 nums[i] 为结尾的递增子序列
            dp[i] = Math.max(dp[i], dp[j] + 1);
             }
        }
}

```
  
### 完整代码
```java

import java.util.Arrays;

/**
 * @Author：huowang
 * @Date：2024/5/28
 */
public class Dp {
        public static void main(String[] args) {
            System.out.println("Hello world!");
            int[] nums = {0,1,0,3,2,3,8,11};
            int aa = lengthOfLIS(nums);
            System.out.println(aa);
        }

        public static int lengthOfLIS(int[] nums) {
            if (nums == null || nums.length == 0) {
                return 0;
            }

            // dp[i] 表示以 nums[i] 结尾的最长递增子序列的长度
            int[] dp = new int[nums.length];
            int max = 1;

            // 初始化 dp 数组，每个元素至少可以构成长度为 1 的递增子序列
            Arrays.fill(dp, 1);

            // 把数组里面每个元素都和他前面的比一遍, 如果大于了就在对应的dp里面+1
            for(int i=0;i<nums.length;i++){
                for (int j=0;j<i;j++){
                    if(nums[i]>nums[j]){
                        dp[i]=Math.max(dp[i],dp[j]+1);
                    }
                }
            }
            int maxLength=0;
            // 求dp里面的最大值
            for(int i=0;i<dp.length;i++){
                maxLength=Math.max(maxLength,dp[i]);
            }

            return maxLength;
        }

}


```

### 参考链接 写的非常好
+ https://github.com/labuladong/fucking-algorithm/blob/master/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E7%B3%BB%E5%88%97/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E8%AE%BE%E8%AE%A1%EF%BC%9A%E6%9C%80%E9%95%BF%E9%80%92%E5%A2%9E%E5%AD%90%E5%BA%8F%E5%88%97.md

#### 联系邮箱 xxx_xxx@aliyun.com