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


### 进阶题, 接雨水
如下图, 求接到的雨水数量, 
![图片](/static/img/img_1.png)
#### 思路1 
我tm直接算!!!,
+ 把这个图转换成数组输入是这样的 [0,1,0,2,1,0,1,3,2,1,2,1], 总的长度是12
+ 其实我们要计算的就是这12个位置,每个上面能存到多少"雨水"
+ 我们可以看出, 一个位置上面能不能存"雨水",其实取决于他左右的位置是不是比他高, 注意这个左右并不限于紧邻的左右,
+ 那具体能存多少"雨水",其实取决于左右里面最小的那个高度
+ so, 我们把每个位置左右的高度都算出来, 在分别进行比较 然后再累加就行了
```java

    // 42. 接雨水
    // 暴力
    // [0,1,0,2,1,0,1,3,2,1,2,1]
    public static int trap(int[] height) {
      int jg=0;
      if(height.length<=2){
          // 那是存不到水的
          return jg;
      }
      // 如果大于了 就一个一个的计算
        for(int i=0;i<height.length;i++){
            //  我们先把这个位置的左右最大的元素给定义出来, 当然这里这是假设 方便后面操作
            int l_max=height[0];
            int r_max=height[height.length-1];

            // 真正开始寻找这个节点左边最大的元素是多少
            for(int j=0;j<=i;j++){
                l_max=Math.max(l_max,height[j]);
            }
            // 寻找右边
            for(int k=i;k<height.length;k++){
                r_max=Math.max(r_max,height[k]);
            }
            // 至此以上, 已经找到了 这个元素左右最大的元素了
            // 之后我们只需要用左右里面比较小的那个 去减掉当前元素就是 能存到的水了
            int i_min = Math.min(l_max, r_max);
            jg=jg+(i_min-height[i]);
        }

      return jg;
    }
```
#### 使用动态规划的思路
其实就是弄个额外的数组去存储 分别从左到右  和 从右到左, 每个位置的最高值(如果当天位置比前一个高,则更新最高值为当前值, 如果没有则用前一个位置的值作为最高值)
```java
 /**
     *   动态规划
     * @param height
     * @return
     */
    public static int trap1(int[] height) {
        int len = height.length;
        if(len<=1) return 0;
        int res = 0;
        //  两个空数组 分别记录从左边 和从右边开始  每个位置及以后最高的高度是多少, 如果后面的高度小于前面的不会被更新
        //  int[] nums = {0,1,0,2,1,0,1,3,2,1,2,1};
        // 这个位置的高度=max(这个位置高度,历史最高值)
        int[] l_max = new int[len];
        int[] r_max = new int[len];
        l_max[0] = height[0];
        r_max[len-1] = height[len-1];
        for (int i = 1; i < len; i++) {
            l_max[i] = Math.max(height[i],l_max[i-1]);
        }
        for(int i = len-2; i>=0;i--){
            r_max[i] = Math.max(height[i],r_max[i+1]);
        }
        for (int i = 0; i < len; i++) {
            res += Math.min(l_max[i],r_max[i])-height[i];
        }
        return res;
    }
```

#### 联系邮箱 xxx_xxx@aliyun.com