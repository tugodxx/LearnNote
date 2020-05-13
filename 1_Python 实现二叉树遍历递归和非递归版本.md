# Python 实现二叉树的前序、中序、后序、层次遍历（递归和非递归版本

## 1. 构建树   

我们先构建一棵简单的树：

```python
class TreeNode:
     def __init__(self, x):
         self.val = x
         self.left = None
         self.right = None
 
a = TreeNode(1)
b = TreeNode(2)
c = TreeNode(3)
d = TreeNode(4)
e = TreeNode(5)
f = TreeNode(6)
g = TreeNode(7)
 
a.left = b
a.right = c
b.left = d
b.right = e
c.left = f
c.right = g
```

这是一棵这样的树：

![img](https://img-blog.csdn.net/20180918205155837?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3MzI0NzQw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 2. 前序遍历
根节点->左子树->右子树
# 先序打印二叉树（递归）
```python
# 先序打印二叉树（递归）
def preOrderTraverse(node):
    if not node:
        return None
    print(node.val)
    preOrderTraverse(node.left)
    preOrderTraverse(node.right)
```

```python
# 先序打印二叉树（非递归）
def preOrderTravese(node):
    stack = [node]
    while len(stack) > 0:
        print(node.val)
        if node.right:
            stack.append(node.right)
        if node.left:
            stack.append(node.left)
        node = stack.pop()
```

输出：1245367

## 3. 中序遍历
左子树->根节点->右子树

```python
# 中序打印二叉树（递归）
def inOrderTraverse(node):
    if node is None:
        return None
    inOrderTraverse(node.left)
    print(node.val)
    inOrderTraverse(node.right)
```

```python
# 中序打印二叉树（非递归）
def inOrderTraverse(node):
    stack = []
    pos = node
    while pos or len(stack) > 0:
        if pos:
            stack.append(pos)
            pos = pos.left
        else:
            pos = stack.pop()
            print(pos.val)
            pos = pos.right
```

输出：4251637

# 4. 后序遍历

左子树->右子树->根节点

```python
# 后序打印二叉树（递归）
def postOrderTraverse(node):
    if node is None:
        return None
    postOrderTraverse(node.left)
    postOrderTraverse(node.right)
    print(node.val)
```

```python
# 后序打印二叉树（非递归）
# 使用两个栈结构
# 第一个栈进栈顺序：左节点->右节点->跟节点
# 第一个栈弹出顺序： 跟节点->右节点->左节点(先序遍历栈弹出顺序：跟->左->右)
# 第二个栈存储为第一个栈的每个弹出依次进栈
# 最后第二个栈依次出栈
def postOrderTraverse(node):
    stack = [node]
    stack2 = []
    while len(stack) > 0:
        node = stack.pop()
        stack2.append(node)
        if node.left is not None:
            stack.append(node.left)
        if node.right is not None:
            stack.append(node.right)
    while len(stack2) > 0:
        print(stack2.pop().val)
```

输出：4526731

# 5. 层次遍历

逐层遍历

```python
def layerTraverse(node):
    
    if not node:
        return None
 
    queue = []  
    queue.append(node)
    while len(queue) > 0:
        tmp = queue.pop(0)
        print(tmp.val)
        if node.left:
            queue.append(tmp.left)
        if node.right:
            queue.append(tmp.right)
```

输出：1234567

# 6. 二叉树的层次遍历

二叉树的层次遍历即从上往下、从左至右依次打印树的节点。
其思路就是将二叉树的节点加入队列，出队的同时将其非空左右孩子依次入队，出队到队列为空即完成遍历。

```python
# -*- coding:utf-8 -*-
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None
class Solution:
    # 返回从上到下每个节点值列表，例：[1,2,3]
    def PrintFromTopToBottom(self, root):
        # write code here
        outList=[]
        queue=[root]
        while queue!=[] and root:
            outList.append(queue[0].val)
            if queue[0].left!=None:
                queue.append(queue[0].left)
            if queue[0].right!=None:
                queue.append(queue[0].right)
            queue.pop(0)
        return outList
```

在python中，队列是用列表来模拟的，其中pop(0)的操作复杂度是O(n)，下面不用队列作为改进。

```python
    def PrintFromTopToBottom(self, root):
        if not root:
            return []
        currentStack = [root]
        outList= []
        while currentStack:
            nextStack = []
            for point in currentStack:
                if point.left: 
                    nextStack.append(point.left)
                if point.right: 
                    nextStack.append(point.right)
                outList.append(point.val)
            currentStack = nextStack
        return outList
```

# 7. 二叉树的按层输出

二叉树的按层输出即从上往下、从左至右依次打印树的节点。每一层输出一行。

这一题和上一题的区别主要在于要区别开每一层，即不仅按层遍历还要按层输出。难点在于有的层会缺项，即不是完全二叉树。但根据层次遍历的特点，节点会按层依次的进入队列，利用这一特点来区分每一层。

上面问题改进版中即是按层存储在每一个列表中的，也适合解决该问题


```python
# -*- coding:utf-8 -*-
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None
class Solution:
    # 返回二维列表[[0],[1,2],[4,5]]
    def Print(self, pRoot):
        # write code here
        if not pRoot:
            return []
        queue=[pRoot]
        outList=[]
        while queue:
            res=[]
            nextQueue=[]
            for point in queue:     #这里再遍历每一层
                res.append(point.val)
                if point.left:
                    nextQueue.append(point.left)
                if point.right:
                    nextQueue.append(point.right)
            outList.append(res)
            queue=nextQueue     #这一步很巧妙，用当前层覆盖上一层
        return outList
```

当然，完全用队列来做也是没有问题的，不过，其关键在于确认每一层的分割点，因此可以用一个标志位来记录这个分割点的位置

```python
    def Print(self, pRoot):
        # write code here
        if not pRoot:
            return []
        queue=[pRoot]
        outList=[]
        while queue:
            res=[]
            i=0
            numberFlag=len(queue)   #这一步记录当前层中节点的个数
            while i <numberFlag:    #这里再遍历每一层
                point=queue.pop(0)
                res.append(point.val)
                if point.left:
                    queue.append(point.left)
                if point.right:
                    queue.append(point.right)
                i+=1
            outList.append(res)
        return outList
```

