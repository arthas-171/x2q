#  LRU算法
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)   
## 概述
实现一个k-v的数据结构, 可以制定存储容量, 然后如果超过了容量则删除最晚使用过的(获取过的)元素, 然后把新的放进去
## 实现思路
能快速根据key获取value肯定是map结构, 但是我们需要维护一个记录,记录下数据结构中数据的使用情况, 其实我们容易想到的就是通过时间戳去实现, 
如果超过了容量, 删除的时候就删除最早存进去的, 但是这样我们每次需要去遍历存储结构,获取到最小的时间戳,这样显然事件负责度比较高, 换个思路
就是用链表去存储元素的使用记录, 我们没使用过一个元素,就把他放到链表的开头, 然后如果超过容量了, 就从链表的结尾去移除,因为我们要快速的添加
移除, 因此选用双链表
## java 编码
+ 先定义一个节点 就是双链表里面存的具体的类型
+ 在定义一个 双向链表,实现链表的 movehead(将元素移动到头部), deletetail(删除尾部元素),两个方法,移动到头部方法需要addhead(在头部新家),deleteNode(删除元素) 
+ 定义最终的数据结构 LURcache, 里面有四个属性, map 用于快速获取, dlinkList 双链表用于记录, size 当前容器大小, total 总容量上限

```java
package org.example.lru;

import java.util.HashMap;
import java.util.Map;

/**
 * @Author：xialy
 * @Date：2024/6/9
 */

public class LRUCache {

    class DLinkedNode {
        int key;
        int value;
        DLinkedNode prev;
        DLinkedNode next;
        public DLinkedNode() {}
        public DLinkedNode(int _key, int _value) {key = _key; value = _value;}
    }

    // 定义一个map 来存key value 用于快速的查找值   注意值是 节点类型
    private Map<Integer, DLinkedNode> cache = new HashMap<Integer, DLinkedNode>();

    /// 表示当前大小
    private int size;
    // 容量, 总的容量
    private int capacity;
    // 双向链表的头和尾
    private DLinkedNode head, tail;

    //  这个数据结构的主体的构造方法, 入参容量
    public LRUCache(int capacity) {
        // 初始化的时候 大小是0
        this.size = 0;
        this.capacity = capacity;
        // 使用伪头部和伪尾部节点
        head = new DLinkedNode();
        tail = new DLinkedNode();
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        DLinkedNode node = cache.get(key);
        if (node == null) {
            return -1;
        }
        // 如果 key 存在，先通过哈希表定位，再移到头部
        // 注意要把获取的节点 移动到双链表的头部
        moveToHead(node);
        // 返回这key对应的值
        return node.value;
    }

    public void put(int key, int value) {
        DLinkedNode node = cache.get(key);
        if (node == null) {
            // 如果 key 不存在，创建一个新的节点
            DLinkedNode newNode = new DLinkedNode(key, value);
            // 添加进哈希表
            cache.put(key, newNode);
            // 添加至双向链表的头部
            addToHead(newNode);
            ++size;
            if (size > capacity) {
                // 如果超出容量，删除双向链表的尾部节点
                DLinkedNode tail = removeTail();
                // 删除哈希表中对应的项
                cache.remove(tail.key);
                --size;
            }
        }
        else {
            // 如果 key 存在，先通过哈希表定位，再修改 value，并移到头部
            node.value = value;
            moveToHead(node);
        }
    }

    private void addToHead(DLinkedNode node) {
        // 注意这里 head是虚拟的节点
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }

    private void removeNode(DLinkedNode node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void moveToHead(DLinkedNode node) {
        // 把元素移动到双链表的头部, 通过把他先移除 在添加到头部的方式实现
        removeNode(node);
        addToHead(node);
    }

    private DLinkedNode removeTail() {
        // 直接找到尾部的前一个节点 将他移除
        DLinkedNode res = tail.prev;
        removeNode(res);
        return res;
    }
}


```




#### 联系邮箱 xxx_xxx@aliyun.com