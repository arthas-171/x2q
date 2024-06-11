#  二叉树
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)       
## 定义
                                                                        
                                                                        
```java

public class BinaryTreeNode {
    
    private int data;  //数据
    private BinaryTreeNode leftChirld;  //左孩子
    private BinaryTreeNode rightChirld; //右孩子
    
    public int getData() {
        return data;
    }
    public void setData(int data) {
        this.data = data;
    }
    public BinaryTreeNode getLeftChirld() {
        return leftChirld;
    }
    public void setLeftChirld(BinaryTreeNode leftChirld) {
        this.leftChirld = leftChirld;
    }
    public BinaryTreeNode getRightChirld() {
        return rightChirld;
    }
    public void setRightChirld(BinaryTreeNode rightChirld) {
        this.rightChirld = rightChirld;
    }        
}


```
                                                                   
                                                                   
## 深度遍历，前序/中序/倒叙
+ 前序遍历：根结点 ---> 左子树 ---> 右子树
+ 中序遍历：左子树---> 根结点 ---> 右子树
+ 后序遍历：左子树 ---> 右子树 ---> 根结点
                                                  
                                                  
```java
//前序遍历
public void preOrderTraverse1(TreeNode root) {
		if (root != null) {
			System.out.print(root.val+"  ");
			preOrderTraverse1(root.left);
			preOrderTraverse1(root.right);
		}
	}

//中序遍历
public void inOrderTraverse1(TreeNode root) {
		if (root != null) {
			inOrderTraverse1(root.left);
			System.out.print(root.val+"  ");
			inOrderTraverse1(root.right);
		}
	}

//后序遍历
public void postOrderTraverse1(TreeNode root) {
		if (root != null) {
			postOrderTraverse1(root.left);
			postOrderTraverse1(root.right);
			System.out.print(root.val+"  ");
		}
	}

```         

## 广度遍历
             
###  按层遍历,比较简单, 借助一个队列完成

                                           
```java

package org.example.Tree;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

/**
 * @Author：xialy
 * @Date：2024/6/9
 */
public class Tree {
    int val;
    Tree left;
    Tree right;
    Tree(int v){
        this.val=v;
    }

    @Override
    public String toString() {
        return "Tree{" +
                "val=" + val +
                ", left=" + left +
                ", right=" + right +
                '}';
    }

    public static void main(String[] args) {
        Tree t1 = new Tree(3);
        Tree t2 = new Tree(9);
        Tree t3 = new Tree(20);
        Tree t4 = new Tree(15);
        Tree t5 = new Tree(7);
        // Tree t6 = new Tree(6);

        t1.left=t2;
        t1.right=t3;



        t3.left=t4;
        t3.right=t5;
        System.out.println(t1.toString());

//        LinkedList<Integer> queue = new LinkedList<Integer>();
//        // offer  从尾部追加一个元素
//        // 从头部获取一个元素并且从队列中删除
//        queue.offer(1);
//        queue.offer(2);
//        queue.offer(3);
//        System.out.println(queue.poll());
//        System.out.println(queue.toString());
////        System.out.println(t1.toString());
////
        // aa(t1);
        System.out.println(getGc(t1));
    }

///   逐层 遍历  借助一个队列 往里面添加每一层的元素, 再获取 在头部添加 在尾部获取
    public static List<List<Integer>> getc(Tree root) {
        List<List<Integer>> ret = new ArrayList<List<Integer>>();
        if (root == null) {
            return ret;
        }
        LinkedList<Tree> queue = new LinkedList<Tree>();
        queue.addLast(root);
        while (!queue.isEmpty()) {
            List<Integer> level = new ArrayList<Integer>();
            // 当前的队列大小 因为在for循环里面会往队列里面添加值
            int currentLevelSize = queue.size();
            for (int i = 1; i <= currentLevelSize; ++i) {
                Tree node;
                node = queue.removeFirst();
                if (node.left != null) {
                    queue.addLast(node.left);
                }
                if (node.right != null) {
                    queue.addLast(node.right);
                }
                level.add(node.val);
            }
            ret.add(level);
        }
        return ret;

    }


//   锯齿状遍历,  这个加一个标记位, 记录遍历到的层数, 然后计算层数分别进行 从头添加再结尾拿 还是从结尾添加在头拿的逻辑, 注意叶子节点的添加先后顺序也需要改
public static List<List<Integer>> getGc(Tree root) {
    List<List<Integer>> ret = new ArrayList<List<Integer>>();
    if (root == null) {
        return ret;
    }

    LinkedList<Tree> queue = new LinkedList<Tree>();
    queue.addLast(root);
    int level_flage=1;
    while (!queue.isEmpty()) {
        List<Integer> level = new ArrayList<Integer>();
        // 当前的队列大小 因为在for循环里面会往队列里面添加值
        int currentLevelSize = queue.size();
        for (int i = 1; i <= currentLevelSize; ++i) {
            Tree node;
            if(level_flage%2==0){
                node = queue.removeFirst();
                //  注意这里叶子节点的添加顺序也需要调整
                if (node.right != null) {
                    queue.addLast(node.right);
                }
                if (node.left != null) {
                    queue.addLast(node.left);
                }


            }else{

                node = queue.removeLast();
                if (node.left != null) {
                    queue.addFirst(node.left);
                }
                if (node.right != null) {
                    queue.addFirst(node.right);
                }
            }
            level.add(node.val);


        }
        level_flage++;
        ret.add(level);
    }

    return ret;

}


   

}



```                                                                                                                                                       
#### 联系邮箱 xxx_xxx@aliyun.com