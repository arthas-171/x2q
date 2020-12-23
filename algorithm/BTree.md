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
                                           
                                           
```java
public void depthOrderTraverse(TreeNode root) {
		if (root == null) {
			return;
		}
       // 使用linkedList 作为一个栈 栈是先进后出的我们把叶子节点压入最前面，根节点在后面压入
		LinkedList<TreeNode> stack = new LinkedList<>();
		stack.push(root);
		while (!stack.isEmpty()) {
			TreeNode node = stack.pop();
			System.out.print(node.val+"  ");
			if (node.right != null) {
				stack.push(node.right);
			}
			if (node.left != null) {
				stack.push(node.left);
			}
		}
	}
```                                                                                                                                                       
#### 联系邮箱 xxx_xxx@aliyun.com