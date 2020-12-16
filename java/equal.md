# java equal compareTo 和 ==
### [go back](/java.md)      
### [go home](../README.md)     
## 概述
+ equals 比较的是两个对象值是否相等
+ == 比较的是内存地址
+ compareTo 是基于字符串的 Unicode值来比较字符串,如果相等返回 0 否则返回一个正数或者负数

## 额外关于声明字符串 String a="a" 和 String a= new String("a")
+ 前者 是会先去常量池寻找有没有"a",如果有直接指向,如果没有创建一个,并且以后如果有声明"a",还是指向该地址
+ 后者是直接创建一个对象在堆里面,
它们做比较会存在一下区别
                                                 
                                                 
```java
package com.example.demo.test;

import java.util.HashMap;

/**
 * @Auther: huowang
 * @Date: 14:56:03 2020/5/23
 * @DES: 字符串比较
 * @Modified By:
 */
public class SingleDog {

    public static void main(String[] args) {
        String a1="aa";
        String a2="aa";
        if(a1.equals(a2)){
            System.out.println("a1.equals(a2)");
        }else {
            System.out.println("!a1.equals(a2)");
        }

        if(a1==a2){
            System.out.println("a1==a2");
        }else {
            System.out.println("!a1==a2");
        }

        String b1=new String("bb");
        String b2=new String("bb");

        if(b1==b2){
            System.out.println("b1==b2");
        }else {
            System.out.println("!b1==b2");
        }
        // a1.equals(a2)
        // a1==a2
        // !b1==b2
    }
}

```                                                 



#### 联系邮箱 xxx_xxx@aliyun.com

