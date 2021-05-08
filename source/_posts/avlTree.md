---
title: AVL树TS实现
date: 2021-05-10 20:37:08
tags: 二叉平衡搜索树、TS实现、AVL

---

## 定义

AVL树也称为平衡二叉树，AVL 树得名于它的发明者 G. M. Adelson-Velsky 和 Evgenii Landis，他们在1962年的论文《An algorithm for the organization of information》中公开了这一数据结构。

AVL是一种特殊的二叉搜索树，具备二叉搜索树的所有特点，且任何一个结点的左子树与右子树都是平衡二叉树，并且高度之差的绝对值不超过 1（此高度差也称为平衡因子balance factor）。

## 为何需要有平衡二叉搜索树

二叉搜索树一定程度上可以提高搜索效率，但是当原序列有序时，例如序列 A = {1，2，3，4，5，6}，

构造二叉搜索树如下图时 ，依据此序列构造的二叉搜索树为右斜树，同时二叉树退化成单链表，搜索效率降低为 O(n)。

<img src="/images/avl/1.jpeg" width = "500" alt="1" align=center />

在上述的二叉搜索树中查找元素 6 需要查找 6 次。

二叉搜索树的查找效率取决于树的高度，因此保持树的高度最小，即可保证树的查找效率。

同样的序列 A，将其改为下图的方式存储，查找元素 6 时只需比较 3 次，查找效率提升一倍。

<img src="/images/avl/2.jpeg" width = "500" alt="1" align=center />



## 具体接口

- 往树中插入(insert)一个结点；
- 删除(remove)树中已经存在的某个结点；
- 搜索(search)树中某个结点；

## 辅助函数

- 获取节点高度方法：treeHeight；
- 获取最大/最小节点方法：getMax/getMin；
- 判断某个节点是否平衡方法：getBalanceFactor；

## 平衡操作

相较于BST，AVL多了一步平衡操作，即在插入或者删除节点时，对树中不平衡的地方进行重新平衡，这是AVL树的精髓，但也是实现的难点所在。

对节点进行平衡之前，需要判断当前节点是否平衡，通过计算节点中左右子树的高度差来判断是否平衡，如果高度差绝对值超过1，则当前节点失衡，需要对其进行平衡操作。

平衡操作分为左旋和右旋：

<img src="/images/avl/3.png" width = "500" alt="1" align=center />

如上图所示，左图为对y节点进行右旋操作，有图为对x节点进行左旋操作，借助这张图片，可以写出左旋右旋的代码：

```js
	/**
     *  对节点y进行向右旋转操作，返回旋转后新的根节点x
     *        y                              x
     *       / \                           /   \
     *      x   T4     向右旋转 (y)        z     y
     *     / \       - - - - - - - ->    / \   / \
     *    z   T3                       T1  T2 T3 T4
     *   / \
     * T1   T2
     * */
	function rightRotate(node) {
        if (!node) return null;
        const leftNode = node.left;
        node.left = leftNode.right;
        leftNode.right = node;
        return leftNode;
    }

    /**
     * 对节点y进行向左旋转操作，返回旋转后新的根节点x
     *    y                             x
     *  /  \                          /   \
     * T1   x      向左旋转 (y)       y     z
     *     / \   - - - - - - - ->   / \   / \
     *   T2  z                     T1 T2 T3 T4
     *      / \
     *     T3 T4
     * */
    function leftRotate(node) {
        if (!node) return null;
        const rightNode = node.right;
        node.right = rightNode.left;
        rightNode.left = node;
        return rightNode;
    }
```

上述，是原理性的操作。在真实的情况下，会遇到四种可能出现的情况：

**case1：LL情况（左左情况）**

<img src="/images/avl/4.png" width = "500" alt="1" align=center />

上述情况，只需对z节点进行右旋操作即可

**case2：LR情况（左右情况）**

<img src="/images/avl/5.png" width = "500" alt="1" align=center />


上述情况，需要对Y节点进行左旋，再对Z节点进行右旋

**case3：RL情况（右左情况）**

<img src="/images/avl/6.png" width = "500" alt="1" align=center />


上述情况，需要对Y节点进行右旋，再对Z节点进行左旋

**case4：RR情况（右右情况）**

<img src="/images/avl/7.png" width = "500" alt="1" align=center />


上述情况，只需对z节点进行左旋操作即可。



TS具体实现

```ts
/***
 * AVL树接口
 */
interface AVLTreeImpl {
    insert: (value: number) => void;
    remove: (target: number) => void;
    search: (target: number) => TreeNode | null;
    getRoot: () => TreeNode;
    getMin: (node: TreeNode) => TreeNode;
    getMax: (node: TreeNode) => TreeNode;
}

/**
 * 树节点
 */
class TreeNode {
    public value: number;
    public left: TreeNode | null;
    public right: TreeNode | null;

    constructor(value) {
        this.value = value;
        this.left = null;
        this.right = null;
    }
}

/**
 * AVL树
 */
class AVLTree implements AVLTreeImpl {

    private root: TreeNode | null;

    constructor(value: number) {
        this.root = new TreeNode(value);
    }

    /**
     * 前序遍历
     * */
    public static PRE_ORDER_TRAVERSE(node: TreeNode): void {
        if (!node) return;
        console.log(node.value);
        this.PRE_ORDER_TRAVERSE(node.left);
        this.PRE_ORDER_TRAVERSE(node.right);
    }

    /**
     * 中序遍历
     * */
    public static IN_ORDER_TRAVERSE(node: TreeNode): void {
        if (!node) return;
        this.IN_ORDER_TRAVERSE(node.left);
        console.log(node);
        this.IN_ORDER_TRAVERSE(node.right);
    }

    /**
     * 后序遍历
     * */
    public static POST_ORDER_TRAVERSE(node: TreeNode): void {
        if (!node) return;
        this.POST_ORDER_TRAVERSE(node.left);
        this.POST_ORDER_TRAVERSE(node.right);
        console.log(node.value);
    }

    /**
     * 获取节点高度
     * 递归实现
     * */
    public treeHeight(node: TreeNode): number {
        if (!node) return 0;
        return Math.max(this.treeHeight(node.left), this.treeHeight(node.right)) + 1;
    }

    /**
     * 获取节点平衡因子
     * */
    public getBalanceFactor(node: TreeNode): number {
        if (!node) return 0;
        return this.treeHeight(node.left) - this.treeHeight(node.right);
    }

    /**
     *  对节点y进行向右旋转操作，返回旋转后新的根节点x
     *        y                              x
     *       / \                           /   \
     *      x   T4     向右旋转 (y)        z     y
     *     / \       - - - - - - - ->    / \   / \
     *    z   T3                       T1  T2 T3 T4
     *   / \
     * T1   T2
     * */
    public rightRotate(node: TreeNode): TreeNode {
        if (!node) return null;
        const leftNode = node.left;
        node.left = leftNode.right;
        leftNode.right = node;
        return leftNode;
    }

    /**
     * 对节点y进行向左旋转操作，返回旋转后新的根节点x
     *    y                             x
     *  /  \                          /   \
     * T1   x      向左旋转 (y)       y     z
     *     / \   - - - - - - - ->   / \   / \
     *   T2  z                     T1 T2 T3 T4
     *      / \
     *     T3 T4
     * */
    public leftRotate(node: TreeNode): TreeNode {
        if (!node) return null;
        const rightNode = node.right;
        node.right = rightNode.left;
        rightNode.left = node;
        return rightNode;
    }

    /**
     * 平衡节点
     * */
    public reBalance(node: TreeNode): TreeNode {
        if (!node) return node;
        if (this.getBalanceFactor(node) > 1 && this.getBalanceFactor(node.left) > 0) {
            /**
             * case1：LL情况，直接右旋
             * 对节点y进行向右旋转操作，返回旋转后新的根节点x
             *        y                              x
             *       / \                           /   \
             *      x   T4     向右旋转 (y)        z     y
             *     / \       - - - - - - - ->    / \   / \
             *    z   T3                       T1  T2 T3 T4
             *   / \
             * T1   T2
             * */
            return this.rightRotate(node);
        } else if (this.getBalanceFactor(node) > 1 && this.getBalanceFactor(node.left) <= 0) {
            /**
             * case2：LR情况，先对左子树节点进行左旋，再对节点进行右旋
             * 对节点x进行向左旋转操作，再对y节点进行右旋操作
             *        y                              y                             T3
             *       / \                           /  \                           /   \
             *      x   T4     向左旋转 (x)        T3   T4       向右旋转 (y)       x     y
             *     / \       - - - - - - - ->    / \         - - - - - - - ->   / \   / \
             *    z   T3                        x  T2                          z  T1 T2  T4
             *       / \                       / \
             *     T1   T2                    z  T1
             * */
            node.left = this.leftRotate(node.left);
            return this.rightRotate(node);
        } else if (this.getBalanceFactor(node) < -1 && this.getBalanceFactor(node.right) > 0) {
            /**
             * case3：RL情况，先对右子树节点进行右旋，再对节点进行左旋
             * 对节点T4进行向右旋转操作，再对y节点进行左旋操作
             *        y                                   y                                     z
             *       / \                                /  \                                  /   \
             *      x   T4          向右旋转 (T4)       x    z            向左旋转 (y)          y     T4
             *         / \       - - - - - - - ->         / \         - - - - - - - ->      / \   / \
             *       z   T3                              T1  T4                            x  T1 T2  T3
             *      / \                                     / \
             *    T1   T2                                 T2  T3
             * */
            node.right = this.rightRotate(node.right);
            return this.leftRotate(node);
        } else if (this.getBalanceFactor(node) < -1 && this.getBalanceFactor(node.right) <= 0) {
            /**
             * case4：RR情况，直接左旋
             * 对节点y进行向左旋转操作，返回旋转后新的根节点x
             *    y                             x
             *  /  \                          /   \
             * T1   x      向左旋转 (y)       y     z
             *     / \   - - - - - - - ->   / \   / \
             *   T2  z                     T1 T2 T3 T4
             *      / \
             *     T3 T4
             * */
            return this.leftRotate(node);
        }
        return node;
    }

    /**
     * 插入节点
     * */
    public insert(value) {
        this.root = this.insertNode(this.root, value);
    }

    /**
     * 插入节点辅助函数，利用递归实现
     * */
    private insertNode(root: TreeNode, value: number): TreeNode {
        if (root.value < value) {
            if (!root.right) {
                root.right = new TreeNode(value);
            } else {
                // 这步递归是关键，非常巧妙，insertNode方法会将沿途的节点重新平衡
                // 平衡后，小范围子树的根节点可能就会发生变化，会失去与父节点的连接关系
                // 而重新将其赋值给root.right父节点之后，又能重新建立联系
                root.right = this.insertNode(root.right, value);
            }
        } else if (root.value > value) {
            if (!root.left) {
                root.left = new TreeNode(value);
            } else {
                root.left = this.insertNode(root.left, value);
            }
        } else {
            return;
        }
        // 在叶子节点中插入新的节点后，会沿经过路径重新平衡整棵树
        return this.reBalance(root);
    }

    /**
     * 查找节点
     * */
    public search(target: number): TreeNode {
        let node = this.root;
        while (node) {
            if (target < node.value) {
                node = node.left;
            } else if (target > node.value) {
                node = node.right;
            } else {
                return node;
            }
        }
        return null;
    }

    /**
     * 获取根节点
     * */
    public getRoot() {
        return this.root;
    }

    /**
     * 删除节点
     * */
    public remove(value: number) {
        this.root = this.removeNode(this.root, value);
    }
    /**
     * 删除节点辅助函数
     * */
    private removeNode(node: TreeNode, target: number): TreeNode {
        if (target > node.value && node.right) {
            node.right = this.removeNode(node.right, target);
        } else if (target < node.value && node.left) {
            node.left = this.removeNode(node.left, target);
        } else if(target === node.value) {
            let curNode = node;

            // 目标节点是叶子节点，无左右子节点，直接删除
            if (!curNode.left && !curNode.right) {
                curNode = null;
                return curNode;
            }

            // 目标节点有左节点，无右节点
            if (curNode.left && !curNode.right) {
                curNode.value = curNode.left.value;
                curNode.left = null;
                return curNode;
            }

            // 目标节点有右节点，无左节点
            if (!curNode.left && curNode.right) {
                curNode.value = curNode.right.value;
                curNode.right = null;
                return curNode;
            }

            /**
             * 目标节点处于中间位置，存在左子节点也存在右子节点
             * 可以有以下两种处理方式：
             * 1、找到当前要删除节点左子树中最大的节点与要删除的节点进行替换
             * 2、找到当前要删除节点右子树中最小的节点与要删除的节点进行替换
             *
             * 以第一种方式为例:
             * 找到左子树中最大的节点s，将s的值替换要删除的节点的值并删除节点s
             */
            if (curNode.left && curNode.right) {
                let maxNodeOfLeft = this.getMax(curNode.left);
                curNode.value = maxNodeOfLeft.value;
                curNode.left = this.removeNode(curNode.left, maxNodeOfLeft.value);
            }
        }
        return this.reBalance(node);
    }

    /**
     * 获取最小节点
     */
    public getMin(node: TreeNode): TreeNode {
        while (node.left) {
            node = node.left;
        }
        return node;
    }

    /**
     * 获取最大节点
     */
    public getMax(node: TreeNode): TreeNode {
        while (node.right) {
            node = node.right;
        }
        return node;
    }
}

const avl = new AVLTree(1);
avl.insert(2);
avl.insert(3);
avl.insert(4);
avl.insert(5);
avl.insert(6);
avl.insert(7);
avl.insert(8);
avl.insert(9);

avl.remove(6);

AVLTree.IN_ORDER_TRAVERSE(avl.getRoot());



```
