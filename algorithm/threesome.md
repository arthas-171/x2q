#  求数组中特定组合
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)   
## 在数组中求特定组合
### 数组找到两个加一起等于特定值的数组,数组是非递减数组
```java
    public static void main(String[] args) {
    int[] nums = {-4,-3,-1,-1,0,1,1,2,2};
    //0   1  2  3 4 5 6 7 8
//        Arrays.sort(nums);
//        for(int l=0;l<nums.length;l++){
//            System.out.println(nums[l]);
//        }

    //  注意  可以利用 数组是有顺序的特性
    int target=3;
    int start=0;
    int end=nums.length-1;
    while (start<end){
        int rs=nums[start]+nums[end];
        if(rs==target){
            System.out.println(start);
            System.out.println(end);
            System.out.println("--------------");
            break;
        }else if(rs<target){
            // 如果结果比目标值小, 因为数组是递增的 所以 左边加一个(换一个更大的数试试)
            start++;
        }else{
            // 结果比目标值大 就换一个更小的数据试试   右边往左挪一个
            end--;
        }
    }
}
```


### 在数组中 求三个元素加一起等于0的组合, 不要求顺序, 
给你一个整数数组 nums ，判断是否存在三元组 [nums[i], nums[j], nums[k]] 满足 i != j、i != k 且 j != k ，同时还满足 nums[i] + nums[j] + nums[k] == 0 。

### 思路
+ 我们可以先把数组排序,直接调用Arrays.sort() 进行排序
+ 之后参考两个元素的情况, 充分利用数组是有序的特性 ,
```java

public class N3 {
    public static void main(String[] args) {
        int[] nums = {-4, -3, -1, -1, 0, 1, 1, 2, 2};

        HashSet<ArrayList<Integer>> set = new HashSet<ArrayList<Integer>>();

        for(int i=0;i<nums.length;i++){
            int start=i+1;
            int end=nums.length-1;
            while (start<end){
                int jg=nums[i]+nums[start]+nums[end];
                if(jg==0){
                    ArrayList<Integer> in = new ArrayList<Integer>();
                    in.add(nums[i]);
                    in.add(nums[start]);
                    in.add(nums[end]);
                    set.add(in);
                    // 如果有符合条件的了 左右各自往中间靠一下
                    start++;
                    end--;
                }else if(jg<0){
                    // 证明左边的小了, start往右移动一位
                    start++;
                }else {
                    end--;
                }
            }
        }
        System.out.println(set.toString());
        List<List<Integer>> list = new ArrayList<>();
        list.addAll(set);
        System.out.println(list);
    }
}

```



#### 联系邮箱 xxx_xxx@aliyun.com