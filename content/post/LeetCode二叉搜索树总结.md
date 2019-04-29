---
title: LeetCode二叉搜索树总结
date: 2019-04-02 15:36:53
categories: 
    - 算法
tags:
    - leetcode
    - 二叉搜索树
---

# 235. Lowest Common Ancestor of a Binary Search Tree
```java
public class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null) {
            return root;
        }
        if (p.val < root.val && q.val < root.val) {
            return lowestCommonAncestor(root.left, p, q);
        }
        if (p.val > root.val && q.val > root.val) {
            return lowestCommonAncestor(root.right, p, q);
        }
        return root;
    }
}
```
* 递归截止条件：`root == null`（不对结果产生影响）；`(p.val <= root.val && q.val >= root.val`（真正的截止条件，由于二叉树的性质，两个节点在root的左右两边，root即为最低公共节点）
* 递归方程：根据不同的条件左右子树遍历的方式
* 递归范围：从root节点到叶子节点
* 中间变量：不需要
* 结果变量：不需要，找到节点`return root`即可
* 注意点：手撕算法的经典题，这道是最基础的版本，即根据二叉搜索树的性质进行判断，之后还会有根据是否有父指针的进阶版本

# 98. Validate Binary Search Tree
```java
public class Solution {
    public boolean isValidBST(TreeNode root) {
        return helper(root, Long.MAX_VALUE, Long.MIN_VALUE);
    }

    private boolean helper(TreeNode root, long minValue, long maxValue) {
        if (root == null) {
            return true;
        }
        if (root.val >= maxValue || root.val <= minValue) {
            return false;
        }
        return helper(root.left, minValue, root.val) && helper(root.right, root.val, maxValue);
    }
}
```
* 递归截止条件：`root == null`（不对结果产生影响)
* 递归方程：左右子树遍历的方式
* 递归范围：从root节点到叶子节点
* 中间变量：minValue代表最小值，maxValue代表最大值
* 结果变量：包含在递归的boolean类型里了
* 注意点：思路倒是不难，需要考虑到引入中间变量来帮助递归

# 450. Delete Node in a BST
```java
class Solution {
    public TreeNode deleteNode(TreeNode root, int key) {
        if (root == null) {
            return null;
        }
        if (root.val > key) {
            root.left = deleteNode(root.left, key);
        } else if (root.val < key) {
            root.right = deleteNode(root.right, key);
        } else {
            if (root.left == null) {
                return root.right;
            } else if (root.right == null) {
                return root.left;
            }
            TreeNode minNode = findMinNode(root.right);
            root.val = minNode.val;
            root.right = deleteNode(root.right, root.val);
        }
        return root;
    }

    private TreeNode findMinNode(TreeNode root) {
        while (root.left != null) {
            root = root.left;
        }
        return root;
    }
}
```
* 递归截止条件：`root == null`（不对结果产生影响)，`root.val == key`（找到要删除的值）
* 递归方程：左右子树**遍历加上编织**的方式
* 递归范围：从root节点到叶子节点
* 中间变量：minNode用来找到右子树中最小的节点
* 结果变量：包含在递归的TreeNode类型里了（这道题是返回root节点，需要在遍历的时候编织）
* 注意点：难点在于编织的方式遍历，同时要考虑如何删除二叉树的节点

# 108. Convert Sorted Array to Binary Search Tree
```java
public class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        //1.找中间数
        //2.左右递归
        //3.对于1个数怎么办--这个是难点
        //4.对于2个数怎么办--这个是难点
        if (nums.length == 0) {
            return null;
        }
        return helper(nums, 0, nums.length - 1);
    }

    private TreeNode helper(int[] nums, int low, int high) {
        if (low > high) {
            return null;
        }
        if (low == high) {
            return new TreeNode(nums[low]);
        }
        int mid = low + (high - low) / 2;
        TreeNode node = new TreeNode(nums[mid]);
        node.left = helper(nums, low, mid - 1);
        node.right = helper(nums, mid + 1, high);
        return node;
    }
}
```
* 递归截止条件：`low > high`（遍历越界，不对结果产生影响)，`low == high`（遍历结束，生成单节点）
* 递归方程：数组边界遍历，二叉树编织
* 递归范围：从`[0,nums.length - 1]`到`low == high`
* 中间变量：初始值为[0,nums.length - 1]左右边界
* 结果变量：包含在递归的TreeNode类型里了（这道题是返回root节点，需要在遍历的时候编织）
* 注意点：难点在于考虑到题目要求是高度平衡的，在编织的时候要选择中间节点为子树的根节点

# 230. Kth Smallest Element in a BST
```java
public class Solution {
    /*
     * 二叉树的思想，左子树的个数+1就是该节点的排序
     * */
    public int kthSmallest(TreeNode root, int k) {
        int count = count(root.left);
        if (k > count + 1) {
            return kthSmallest(root.right, k - count - 1);
        } else if (k < count + 1) {
            return kthSmallest(root.left, k);
        } else {
            return root.val;
        }
    }

    private int count(TreeNode node) {
        if (node == null) {
            return 0;
        }
        return 1 + count(node.left) + count(node.right);
    }
}
```
* 递归截止条件：`count == k + 1`（左子树的总节点个数等于k+1)
* 递归方程：左右子树遍历查找
* 递归范围：从根节点到叶子节点（找到就停）
* 中间变量：count的辅助递归函数
* 结果变量：包含在递归的int类型里了
* 注意点：难点在于要考虑到树高要尽量平衡，故递归的一般情况要选中间节点才行，还有就是k的边界条件又搞错了

# 236. Lowest Common Ancestor of a Binary Tree
```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) {
            return root;
        }
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);

        if (left != null && right != null) {
            return root;
        } else {
            return left != null ? left : right;
        }
    }
}
```
* 递归截止条件：`root == null || root == p || root == q`（找到了节点返回p或q，到达边界一路返回null)
* 递归方程：左右子树遍历查找
* 递归范围：从根节点到叶子节点（找到就停）
* 中间变量：无
* 结果变量：包含在递归的TreeNode类型里了
* 注意点：这道题自己理解是很难的，但是其解法却很巧妙，关键在于理解递归截止的函数，以及关键条件（节点的左子树包含一个，右子树包含一个；或者一个在另外一个的子树中）