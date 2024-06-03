# 合并数组里面的连续的区间
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)   

## 实现一个函数，对于输入一个区间数组，合并其中有交集的区间。 输入的区间数组已经按区间起点排序
+ 输入 int[][] arr ={{1,2},{3,6},{5,9},{11,16},{15,18}};  输出 1-2,3-9,11-18
+ 输入 int[][] arr ={{1,3},{2,6},{5,9},{11,13},{15,18}};  输出 1-9,11-13,15-18
+ 输入 int[][] arr ={{1,2},{4,6},{5,9},{11,16},{15,18}};  输出 1-2,4-9,11-20

### 核心思想
我们借助一个map去实现这个功能, map里面的key存放的是区间的起点, value存放的是区间的终点,
循环数组用比较每个区间的起点是否在现有区间的终点以前,如果是则替换新区间的终点为区间终点,
注意对于已经合并过的区间不需要再去重复操作,直接跳到未合并的区间开始计算

```java


/**
 * @Author：huowang
 * @Date：2024/5/31
 */
public class IJoin {
    public static void main(String[] args) {
//        实现一个函数，对于输入一个区间数组，合并其中有交集的区间。
//        输入的区间数组已经按区间起点排序
      //  int[][] arr ={{1,2},{3,6},{5,9},{11,16},{15,18}};
        int[][] arr ={{1,2},{4,6},{5,9},{11,16},{15,18}};
       // int[][] arr ={{1,3},{2,6},{5,9},{11,13},{15,18}};
        int flag=0;
        HashMap<Integer, Integer> map = new HashMap<Integer, Integer>();
        while (flag< arr.length){
            int i=flag;
            map.put(arr[i][0],arr[i][1]);
            for(int j=i;j<arr.length;j++){
                if(map.get(arr[i][0])>arr[j][0]){
                    flag++;
                    map.put(arr[i][0],arr[j][1]);
                }
            }
        }
        System.out.printf(map.toString());
    }
}


```
#### 联系邮箱 xxx_xxx@aliyun.com