# 字符串中最长子字符串
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)   

## 题目
+ 力扣 https://leetcode.cn/problems/wtcaE1/description/
+ 给定一个字符串 s ，请你找出其中不含有重复字符的 最长连续子字符串 的长度。


```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
char[] arr =s.toCharArray();

        String  str="";
        ArrayList<Integer> l2 = new ArrayList<Integer>();
        int flag=0;
        while (flag<arr.length-1){
            if(str.indexOf(arr[flag])<0){
                    str=str.concat(String.valueOf(arr[flag]) );
                    l2.add(flag,str.length());
            }else{
                str=String.valueOf(arr[flag]);
                l2.add(flag,str.length());
            }
            flag++;
        }
        return Collections.max(l2);
    }
}
```

#### 联系邮箱 xxx_xxx@aliyun.com