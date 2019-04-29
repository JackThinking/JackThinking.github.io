---
title: LeetCode递归总结
date: 2019-04-01 22:44:34
categories: 
    - 算法
tags:
    - leetcode
    - 递归
---

# 257. Binary Tree Paths
```java
public class Solution {
    public List<String> binaryTreePaths(TreeNode root) {
        List<String> res = new ArrayList<>();
        helper(root, res, "");
        return res;
    }

    private void helper(TreeNode root, List<String> res, String string) {
        if (root == null) {
            return;
        }
        if (root.left == null && root.right == null) {
            res.add(string + root.val);
            return;
        }
        helper(root.left, res, string + root.val + "->");
        helper(root.right, res, string + root.val + "->");
    }
}
```
* 递归截止条件：`root == null`（不对结果产生影响）；`root.left == null && root.right == null`（真正的截止条件，到达树的叶子节点）
* 递归方程：左右子树遍历的方式
* 递归范围：从根节点到叶子节点
* 中间变量：用初始值为“”的String进行遍历的存储
* 结果变量：可以放在全局变量里，也可以放在递归函数里面
* 注意点：不是回溯，递归就行；string的生成过程还是没想清楚

# 129. Sum Root to Leaf Numbers
## 低效版(by myself)
```java
class Solution {
    public int sumNumbers(TreeNode root) {
        int res = 0;
        List<String> list = new ArrayList<>();
        helper(root, list, "");
        for (int i = 0; i < list.size(); i++) {
            res += Integer.parseInt(list.get(i));
        }
        return res;
    }

    private void helper(TreeNode root, List<String> list, String num) {
        if (root == null) {
            return;
        }
        if (root.left == null && root.right == null) {
            list.add(num + root.val);
            return;
        }
        helper(root.left, list, num + root.val);
        helper(root.right, list, num + root.val);
    }
}
```
* 递归截止条件：`root == null`（不对结果产生影响）；`root.left == null && root.right == null`（真正的截止条件，到达树的叶子节点）
* 递归方程：左右子树遍历的方式
* 递归范围：从根节点到叶子节点
* 中间变量：用初始值为“”的String进行遍历的存储
* 结果变量：放在递归函数里面保存，然后循环语句里面提取出答案
* 注意点：运行效率不高，存在信息的冗余，信息的保存用string很浪费，同时最后的sum值也可以简化在递归函数里面
## 高效版(by others)
```java
class Solution {
    public int sumNumbers(TreeNode root) {
        return helper(root, 0);
    }

    private int helper(TreeNode root, int sum) {
        if (root == null) {
            return 0;
        }
        sum = 10 * sum + root.val;
        if (root.left == null && root.right == null) {
            return sum;
        }
        return helper(root.left, sum) + helper(root.right, sum);
    }
}
```
* 递归截止条件：`root == null`（不对结果产生影响）；`root.left == null && root.right == null`（真正的截止条件，到达树的叶子节点）
* 递归方程：左右子树遍历的方式，同时将sum求和的方程融合进去（注意此时需要递归函数不是void）
* 递归范围：从根节点到叶子节点
* 中间变量：用sum保存
* 结果变量：用sum保存，关键点在于return的时候讲左右子树遍历相加
* 注意点：运行效率高，变量使用少，注意递归的单条路线遍历累加方式和多条线路合并成答案的方式（此处分别是`sum = 10 * sum + root.val`和`helper(root.left, sum) + helper(root.right, sum)`）

# 437. Path Sum III
```java
class Solution {
    public int pathSum(TreeNode root, int sum) {
        if (root == null) {
            return 0;
        }
        int res = helper(root, sum);
        res += pathSum(root.left, sum);
        res += pathSum(root.right, sum);
        return res;
    }

    private int helper(TreeNode root, int sum) {
        if (root == null) {
            return 0;
        }
        int res = 0;
        if (root.val == sum) {
            res++;//此处不能return，因为可能出现负数的情况，之后的路径中还能出现不同的路径
        }
        res += helper(root.left, sum - root.val);
        res += helper(root.right, sum - root.val);
        return res;
    }
}
```
* 递归截止条件：`root == null`（不对结果产生影响）；`root.val == sum`（真正的截止条件，找到了一个答案路径，但是考虑到有负数情况的存在，即节点之后的路径上可能产生新的答案项，所以此处不能return）
* 递归方程：左右子树遍历的方式，同时将res累加的方程融合进去（注意此时需要递归函数不是void）
* 递归范围：中间的任意路径（此时需要将原函数改为递归函数，考虑三种情况，一是本节点在路径中，另外二个是本节点不在路径中，而是在左右子树节点中）
* 中间变量：用sum保存路径
* 结果变量：用res保存答案信息，不是通过全局变量，也不是通过递归函数参数，而是使用递归的int来保存（新的思想）
* 注意点：高效，使用变量不冗余