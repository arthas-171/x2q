# 链表翻转
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)   

### 单链表 插入一个元素
```java
 public static void main(String[] args) {
        ListNode head = new ListNode(1);
        head.next = new ListNode(2);
        head.next.next = new ListNode(3);
        head.next.next.next = new ListNode(4);
        head.next.next.next.next = new ListNode(5);


   //    ListNode aa = fenzu(head, 2);

        //  在单链表中间插入一个元素
        // 例如在第二和第三个元素中间插入一个 66
        ListNode start=head;
        ListNode end=head;
        int ct=0;
        while (ct<1){
            end=end.next;
            ct++;
        }
        ListNode newNode=new ListNode(66);
        newNode.next=end.next;
        end.next=newNode;
        ListNode aa=start;

        while (aa!=null){
            System.out.println(aa.val);
            aa=aa.next;
        }


    }
```


## 删除元素
```java
   public static void main(String[] args) {
        ListNode head = new ListNode(1);
        head.next = new ListNode(2);
        head.next.next = new ListNode(3);
        head.next.next.next = new ListNode(4);
        head.next.next.next.next = new ListNode(5);
        

        // 删除第三个元素
        ListNode start=head;
        ListNode end=head;
        int ct=0;
        while (ct<1){
            end=end.next;
            ct++;
        }
        end.next=end.next.next;
        ListNode aa=start;

        while (aa!=null){
            System.out.println(aa.val);
            aa=aa.next;
        }
    }
```

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

## 进阶
### 翻转某个区间内的链表 
例如 翻转链表前1~3元素, 后面保持不变
```java
    public static void main(String[] args) {
        ListNode head = new ListNode(1);
        head.next = new ListNode(2);
        head.next.next = new ListNode(3);
        head.next.next.next = new ListNode(4);
        head.next.next.next.next = new ListNode(5);
        // 核心是要定义一个  翻转链表某个区间的函数,之后在找到这个区间的开始和截止
        // 翻转链表前3个元素  
        // 获取前三个元素
        int flag=0;
        ListNode start=head;
        ListNode end=head;
        while (flag<3){
            end=end.next;
            flag++;
        }
        ListNode aa = fz13(start, end);
        aa.next.next.next=end;
        while (aa!=null){
            System.out.println(aa.val);
            aa=aa.next;
        }


    }

//  翻转链表某个区间
public static  ListNode fz13(ListNode start,ListNode end){
    ListNode pre=null;
    ListNode curr=start;
    ListNode tmp;
    while (curr!=end){
        tmp=curr.next;
        curr.next=pre;
        pre=curr;
        curr=tmp;
    }
    return pre;
}

```

### n个一组 翻转链表
```java
   public static void main(String[] args) {
    ListNode head = new ListNode(1);
    head.next = new ListNode(2);
    head.next.next = new ListNode(3);
    head.next.next.next = new ListNode(4);
    head.next.next.next.next = new ListNode(5);
    ListNode aa = fenzu(head, 2);
    while (aa!=null){
        System.out.println(aa.val);
        aa=aa.next;
    }
}

//  思路 还是要先定义好 翻转区间的函数,
public static  ListNode fenzu(ListNode head,int k){
    ListNode start=head;
    ListNode end=head;

    // 设置递归跳出的条件
    // 找到第一次的时候end的位置
    int ct=0;
    while (ct<k){
        ct++;
        if(end.next!=null){
            end=end.next;
        }else {
            // 这个是递归跳出的条件, 已经不够k个元素了
            return  start;
        }

    }
    ListNode newhead = fanzhuan(start, end); //2->1
    // 递归调用分组翻转方法
    // 注意递归是要先压栈 在执行, 所以思考🤔的时候从最后一次开始思考
    start.next= fenzu(end,k);
    return  newhead;
}

private static ListNode fanzhuan(ListNode start, ListNode end) {
    ListNode pre=null;
    ListNode curr=start;
    ListNode tmp=null;
    while (curr!=end){
        tmp=curr.next;
        curr.next=pre;
        pre=curr;
        curr=tmp;
    }
    return  pre;
}
```
## 参考链接
https://labuladong.online/algo/data-structure/reverse-nodes-in-k-group/#%E4%BA%8C%E3%80%81%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0
#### 联系邮箱 xxx_xxx@aliyun.com