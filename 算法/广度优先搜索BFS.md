# 广度优先搜索BFS

## 1、广度优先搜索

一般是作用在树上的，当然作用在图上也可以。

最基础的，我们想按树的层级，逐层级打印该层级的所有节点。很明显，按层级打印节点，就不能使用深度优先搜索了。

对于广度优先搜索，需要维护一个队列，入队列的是被遍历到的节点，出队列的话则打印。

说起来太麻烦了，不如直接看题根。

```markdown
题目：从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。

例如:
给定二叉树: [3,9,20,null,null,15,7],

    3
   / \
  9  20
    /  \
   15   7

返回：[3,9,20,15,7]

```

代码：

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int[] levelOrder(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if (root == null) {
            return new int[0];
        }
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while (!queue.isEmpty()) {
            int size = queue.size();
            for (int i = 0; i < size; i++) {
                TreeNode temp = queue.poll();
                res.add(temp.val);
                if (temp.left != null) {
                    queue.offer(temp.left);
                }
                if (temp.right != null) {
                    queue.offer(temp.right);
                }
            }
        }
        int len = res.size();
        int[] realRes = new int[len];
        int count = 0;
        for (Integer i : res) {
            realRes[count] = i;
            count++;
        }
        return realRes;
    }
}
```



广度优先的题目都差不多，反正都是造：

```java
Queue<Object> queue = new LinkedList<>();
```

先添加一个root节点

然后反反复复 `offer()` `poll()`

直到队列中没有元素`while (!queue.isEmpty())`



但是今天写的时候，遇到一个连通图的题目，里面造的数据结构挺有趣的。

题目是这样的：

```markdown
例 2：「力扣」第 323 题：无向图中连通分量的数目（中等）
给定编号从 0 到 n-1 的 n 个节点和一个无向边列表（每条边都是一对节点），请编写一个函数来计算无向图中连通分量的数目。


示例 1：
输入: n = 5 和 edges = [[0, 1], [1, 2], [3, 4]]

     0          3
     |          |
     1 --- 2    4 

输出: 2


示例 2：
输入: n = 5 和 edges = [[0, 1], [1, 2], [2, 3], [3, 4]]

     0           4
     |           |
     1 --- 2 --- 3

输出:  1

```

它造了一个列表数组`List<Integer>[] adj = new ArrayList[n];`，数组长度为 n 的数值，即具体的节点。

比方说`adj[0]`就表示节点0。

而这个`adj[i]`则是`adj[i] = new ArrayList<>();`里面存放的都是与节点 i 相邻的节点的。

它是这样处理edges的：

```java
for (int[] edge : edges) {
  adj[edge[0]].add[edge[1]];
  adj[edge[1]].add[edge[0]];
}
```

这样所有连通关系都出来了，很美妙。

## 2、总结

在具体问题中，要辩证使用 dfs 和 bfs 。有的题，dfs 会比 bfs 慢很多，而有的题，bfs 则会 dfs 慢得多。

比如下题中，dfs 就比 bfs 快很多。

```markdown
在二叉树中，根节点位于深度 0 处，每个深度为 k 的节点的子节点位于深度 k+1 处。
如果二叉树的两个节点深度相同，但 父节点不同 ，则它们是一对堂兄弟节点。
我们给出了具有唯一值的二叉树的根节点 root ，以及树中两个不同节点的值 x 和 y 。
只有与值 x 和 y 对应的节点是堂兄弟节点时，才返回 true 。否则，返回 false。


示例 1：
输入：root = [1,2,3,4], x = 4, y = 3
输出：false


示例 2：
输入：root = [1,2,3,null,4,null,5], x = 5, y = 4
输出：true


示例 3：
输入：root = [1,2,3,null,4], x = 2, y = 3
输出：false
```

dfs 解法：（0ms）

```java
class Solution {
    int xpar, xdep, ypar, ydep;

    public boolean isCousins(TreeNode root, int x, int y) {
        dfs(root.left, 1, x, y, root.val);
        dfs(root.right, 1, x, y, root.val);
        return (xpar != ypar) && (xdep == ydep);
    }
    
    public void dfs(TreeNode node, int dep, int x, int y, int par) {
        if (node == null) {
            return;
        }
        if (node.val == x) {
            xpar = par;
            xdep = dep;
        } else if (node.val == y) {
            ypar = par;
            ydep = dep;
        } else {
            dfs(node.left, dep+1, x, y, node.val);
            dfs(node.right, dep+1, x, y, node.val);
        }
    }
}
```

bfs 解法：（1ms）

```java
class Solution {
    public boolean isCousins(TreeNode root, int x, int y) {
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        int total = x + y;
        boolean isXInThisLevel = false;
        boolean isYInThisLevel = false;

        while (!queue.isEmpty()) {
            int size = queue.size();
            for (int i = 0; i < size; i++) {
                TreeNode temp = queue.poll();

                if (temp.left != null && temp.right != null && (temp.left.val == x || temp.left.val == y) && (temp.right.val == total - temp.left.val)) {
                    return false;
                }

                
                int value = temp.val;
                if (value == x) {
                    isXInThisLevel = true;
                }
                if (value == y) {
                    isYInThisLevel = true;
                }
                if (temp.left != null) {
                    queue.offer(temp.left);
                }
                if (temp.right != null) {
                    queue.offer(temp.right);
                }
                if (isXInThisLevel && isYInThisLevel) {
                    return true;
                }
            }
            isXInThisLevel = false;
            isYInThisLevel = false;
        }
        return false;


    }
}
```

