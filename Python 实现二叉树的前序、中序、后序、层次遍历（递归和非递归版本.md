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