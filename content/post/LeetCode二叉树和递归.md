---
title: LeetCode二叉树和递归
date: 2019-04-05 20:39:14
categories: 
    - 算法
tags:
    - leetcode
    - 二叉搜索树
    - 递归
---
# 104. Maximum Depth of Binary Tree

```java
public class Solution {
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
    }
}
```
* 递归截止条件：`root == null`
* 递归方程：左右子树遍历
* 递归范围：从root节点到叶子节点
* 中间变量：不需要
* 结果变量：不需要，包含在int里面了
* 注意点：直接在return里面写累加的过程

# 111. Minimum Depth of Binary Tree
```java
class Solution {
    public int minDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        if (root.left == null) {
            return 1 + minDepth(root.right);
        } else if (root.right == null) {
            return 1 + minDepth(root.left);
        } else {
            return 1 + Math.min(minDepth(root.left), minDepth(root.right));
        }
    }
}
```
* 递归截止条件：`root == null`
* 递归方程：左右子树遍历
* 递归范围：从root节点到叶子节点
* 中间变量：不需要
* 结果变量：不需要，包含在int里面了
* 注意点：和最大深度不一样的是，如果子树为空，则结果会被污染

# 226. Invert Binary Tree

```java
public class Solution {
    public TreeNode invertTree(TreeNode root) {
        if (root == null) {
            return root;
        }
        TreeNode temp = root.left;
        root.left = invertTree(root.right);
        root.right = invertTree(temp);
        return root;
    }
}
```
* 递归截止条件：`root == null`
* 递归方程：左右子树遍历
* 递归范围：从root节点到叶子节点
* 中间变量：不需要
* 结果变量：不需要，包含在TreeNode里面了
* 注意点：swap的基础思想

# 100. Same Tree
```java
class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
        if (p == null && q == null) {
            return true;
        }
        if (p == null || q == null) {
            return false;
        }
        if (p.val == q.val) {
            return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
        }
        return false;
    }
}
```
* 递归截止条件：`p == null && q == null`
* 递归方程：左右子树遍历
* 递归范围：从root节点到叶子节点
* 中间变量：不需要
* 结果变量：不需要，包含在boolean里面了
* 注意点：三个if可以考虑到所有情况


# 101. Symmetric Tree
```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        return root == null || helper(root, root);
    }

    private boolean helper(TreeNode left, TreeNode right) {
        if (left == null || right == null) {
            return left == right;
        }
        if (left.val != right.val) {
            return false;
        }
        return helper(left.left, right.right) && helper(left.right, right.left);
    }
}
```
* 递归截止条件：`left == null || right == null`**(由于后面有left.left等，故只要left与right一者为null就需要停止判断)**
* 递归方程：左的左对比右的右，左的右对比右的左，&&连接
* 递归范围：从root节点到叶子节点
* 中间变量：不需要
* 结果变量：不需要，包含在boolean里面了
* 注意点：要将helper函数包含两个TreeNode参数

# 222. Count Complete Tree Nodes
```java
class Solution {
    public int countNodes(TreeNode root) {
        if (root == null) {
            return 0;
        }
        return 1 + countNodes(root.left) + countNodes(root.right);
    }
}
```
* 递归截止条件：`root == null`
* 递归方程：左右遍历子树，+1累计
* 递归范围：从root节点到叶子节点
* 中间变量：不需要
* 结果变量：不需要，包含在int里面了
* 注意点：这种方法是通用树的解决方式，其实平衡二叉树其性质是除了最后一层都是满的，所以可以根据层数和底层叶子数来求解

# 110. Balanced Binary Tree
```java
class Solution {
    public boolean isBalanced(TreeNode root) {
        if (root == null) {
            return true;
        }
        if (Math.abs(depth(root.left) - depth(root.right)) > 1) {
            return false;
        }
        return isBalanced(root.left) && isBalanced(root.right);
    }

    private int depth(TreeNode node) {
        if (node == null) {
            return 0;
        }
        return 1 + Math.max(depth(node.left), depth(node.right));
    }
}
```
* 递归截止条件：`root == null`
* 递归方程：左右遍历子树，&&连接
* 递归范围：从root节点到叶子节点
* 中间变量：使用了另外一个求深度的递归函数
* 结果变量：不需要，包含在boolean里面了
* 注意点：一个递归调用另外一个递归的题目，要注意

# 112. Path Sum
```java
class Solution {
    public boolean hasPathSum(TreeNode root, int sum) {
        return helper(root, sum, 0);
    }

    private boolean helper(TreeNode root, int sum, int tempSum) {
        if (root == null) {
            return false;
        }
        tempSum += root.val;//中间变量的更新放在外面会更好
        if (root.left == null && root.right == null && tempSum == sum) {
            return true;
        }
        return helper(root.left, sum, tempSum) || helper(root.right, sum, tempSum);
    }
}
```
* 递归截止条件：`root == null`（无用）；`root.left == null && root.right == null && tempSum == sum`（有用）
* 递归方程：左右遍历子树，||连接，只要找到一条即可
* 递归范围：从root节点到叶子节点（题目明确说明）
* 中间变量：使用tempSum来保存中间的值，由于只要返回boolean变量，故不需要具体的List来保存
* 结果变量：不需要，包含在boolean里面了
* 注意点：中间变量的更新放在递归外面会更好

# 404. Sum of Left Leaves
```java
class Solution {
    public int sumOfLeftLeaves(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int res = 0;
        if (root.left != null) {
            if (root.left.left == null && root.left.right == null) {
                res += root.left.val;
            } else {
                res += sumOfLeftLeaves(root.left);
            }
        }
        res += sumOfLeftLeaves(root.right);
        return res;
    }
}
```
* 递归截止条件：`root == null`（无用）；`root.left == null && root.right == null && tempSum == sum && root是其父节点的左子树`（有用）
* 递归方程：左右遍历子树
* 递归范围：从root节点到叶子节点（根据题目理解）
* 中间变量：不需要
* 结果变量：使用res来保存中间的值
* 注意点：如果要判断是不是左子树，按照往常的判断是不行的（只要判断是否是叶子节点），那就返回到上一层去考虑，以叶子节点的父节点去为出发点，就可以解决了，这道题很好