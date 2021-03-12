---
title: leetcode-二叉树-遍历-01
date: 2020-12-03 17:28:04
tags:
categories:
description: "leetcode 二叉树刷题遍历"
---



```
void traverse(TreeNode root) {
    // 前序遍历
    traverse(root.left)
    // 中序遍历
    traverse(root.right)
    // 后序遍历
}
```

```

/**
    * 先序遍历
    *144
    *
    作者：LeetCode-Solution
    链接：https://leetcode-cn.com/problems/binary-tree-preorder-traversal
    来源：力扣（LeetCode）
    著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
    * @param root
    * @return
    */
public static String traverseb(TreeNode root) {
    // 对于空节点，可以用一个特殊字符表示
    if (root == null) {
        return "#";
    }
    // 将左右子树序列化成字符串
    String left = traverseb(root.left);
    String right = traverseb(root.right);
    /* 先序遍历代码位置 */
    // 左右子树加上自己，就是以自己为根的二叉树序列化结果
    String subTree = root.val + "," + left + "," + right;
    return subTree;
}


```


参考:[遍历二叉树](https://github.com/muxiaobai/java-demo/blob/master/test-java-demo/src/main/java/io/github/muxiaobai/labuladong/hhh.java)