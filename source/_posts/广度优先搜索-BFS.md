---
title: 广度优先搜索(BFS)
top: false
cover: false
toc: true
mathjax: true
date: 2022-12-07 17:24:58
password:
summary: 学习广度优先搜索
tags:
- java
- 树
categories:
- 算法
---
## 广度优先搜索

![图片](https://img-blog.csdnimg.cn/1136649fc8134815bdb0aaf2ce227ce6.png)

#### 思想
```text
广度优先遍历树，需要用到队列（Queue）来存储节点对象，队列的特点就是先进先出。先往队列中插入左节点，再插入右节点，队列出队就是先出左节点，再出右节点。
 
首先将A节点插入队列中，队列中有元素（A）;
 
将A节点弹出，同时将A节点的左节点B、右节点C依次插入队列，B在队首，C在队尾，（B，C），此时得到A节点；
 
将队首元素B弹出，并将B的左节点D、右节点E插入队列，C在队首，E在队尾（C，D，E），此时得到B节点；
 
再将队首元素C弹出，并将C节点的左节点F，右节点G依次插入队列，（D，E，F，G），此时得到C节点；
 
将D弹出，此时D没有子节点，队列中元素为（E，F，G），得到D节点；
 。。。以此类推。最终的遍历结果为：A，B，C，D，E，F，G，H，I。
```

#### 代码

```java
package algorithm.list;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.Queue;

/**
 * 广度深度优先搜索
 * 可以理解为 从上到下  从左到右 一层一层遍历
 * @author dfq 2022/9/15 16:37
 * @implNote
 */
public class BFSSort {

     class TreeNode {
        int val = 0;
        TreeNode left = null;
        TreeNode right = null;
        public TreeNode(int val) {
            this.val = val;
        }
    }

    public ArrayList<Integer> BfsTree(TreeNode root) {
        ArrayList<Integer> lists = new ArrayList<>();
        if(root == null) {
            return lists;
        }
        Queue<TreeNode> queue = new LinkedList<>();
        //插入队列
        queue.offer(root);
        while(!queue.isEmpty()){
            //取出队列头信息并删除
            TreeNode tree = queue.poll();
            if(tree.left != null) {
                queue.offer(tree.left);
            }
            if(tree.right != null) {
                queue.offer(tree.right);
            }
            lists.add(tree.val);
        }
        return lists;
    }

}
```