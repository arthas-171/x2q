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
        ListNode tmp;

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

## 使用递归的方式翻转链表

```java
public ListNode dg(ListNode head) {
    // 首先要判断跳出递归的条件, head 或者 head的下一个节点是null
    if (head != null && !head.next()==null){
        return  head;
    }
    // 然后递归调用方法本身, 把当前节点的下一个节点传入, 
    // 注意这里 可以理解为, 我们会一直调用这个方法, 指导传入的是最后一个节点, 因为最后一个节点没有next 就会被直接返回,
    // 从最后一个节点被返回开始,才会逐步执行翻转的逻辑, 
    ListNode newhead=dg(head.next());
    //  我们相当于跳过了 当前节点的下一个节点, 直接将当前节点付给他的下下一个节点,想象一下 在最后一个节点
      head.next().next()=head;
      // 然后这里斩断 当前节点和下一个节点的链接
      head.next()=null;
        return newhead;
}

```
### 画个图吧 展示一下
思考这个过程的时候不要硬性的去尝试在脑子里面压栈, 先理解掉 递归在执行head.next.next之前,其实指针已经移动到了 最后一位,然后再去理解下面的,
并且请明白链表的最后一个元素的下一个元素是null,null的下一个还是null,不要认为下面已经"结束了".
![图片](/static/img/img.png)
#### 联系邮箱 xxx_xxx@aliyun.com