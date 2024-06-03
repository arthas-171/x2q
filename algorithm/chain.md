# 链表翻转
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)   

## 使用循环实现链表翻转

## 定义一个链表
```java
public class ListNode {

    int val;
    ListNode next;
    ListNode(int x) {
        val = x;
        next = null;
    }
}


```
## 直接使用while循环翻转
```java
  public ListNode fz1(ListNode node){
       //  把三个节点都定义出来 注意最后一个tmp是用于临时存放 当前节点的下一个节点的临时节点
        ListNode pre=null; // 这个其实是新的要往回返的链表
        ListNode curr=node;
        ListNode tmp=null;

        while (curr!=null){
            tmp=curr.next;//把当前节点的下一个节点写入临时节点存储
            curr.next=pre;// 当前节点的下一个节点的值换成前一个节点的值
            pre=curr;// 前一个换成当前
            curr=tmp;// 当前换成下一个
        }
        //返回的时候应该吧 头返回回去
        return pre;
    }

```

#### 联系邮箱 xxx_xxx@aliyun.com