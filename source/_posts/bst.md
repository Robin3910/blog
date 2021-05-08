---
title: 二叉搜索树TS实现
date: 2021-05-09 21:37:08
tags: 二叉搜索树、TS实现、BST
---
数组的搜索比较方便，可以直接使用下标，但删除或者插入就比较麻烦了，而链表与之相反，删除和插入都比较简单， 但是查找很慢，这自然也与这两种数据结构的存储方式有关，数组是取一段相连的空间，而链表是每创建一个节点便取一个节点所需的空间，
只是使用指针进行连接，空间上并不是连续的。而二叉树就既有链表的好处，又有数组的优点。 

## 应用

二叉搜索树作为一种经典的数据结构，它既有链表的快速插入与删除操作的特点，又有数组快速查找的优势；所以应用十分广泛，例如在文件系统和数据库系统一般会采用这种数据结构进行高效率的排序与检索操作。

## 特点 

在二叉搜索树中：

- 若任意结点的左子树不空，则左子树上所有结点的值均不大于它的根结点的值。 
- 若任意结点的右子树不空，则右子树上所有结点的值均不小于它的根结点的值。 
- 任意结点的左、右子树也分别为二叉搜索树。

## TS实现

```ts
/***
 * 二叉搜索树接口
 */
interface BinarySearchTreeImpl {
    insert: (value: number) => void;
    remove: (target: number) => void;
    search: (target: number) => TreeNode | null;
    getRoot: () => TreeNode;
    getMin: () => TreeNode;
    getMax: () => TreeNode;
}


/***
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

/***
 * 二叉搜索树
 */
class BinarySearchTree implements BinarySearchTreeImpl {
    
    private readonly root: TreeNode | null;
    
    constructor(val) {
        if (!this.root) {
            this.root = new TreeNode(val);
        }
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
        console.log(node.value);
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
     * 获取根节点
     * */
    public getRoot() {
        return this.root;
    }
    
    /**
     * 插入节点
     * */
    public insert(value: number) {
        const node = new TreeNode(value);
        let curNode = this.root;
        let preNode;
        // 先找到插入的位置
        while (curNode) {
            if (curNode.value === node.value) {
                return;
            } else if (curNode.value > node.value) {
                preNode = curNode;
                curNode = curNode.left;
            } else if (curNode.value < node.value) {
                preNode = curNode;
                curNode = curNode.right;
            }
        }
        // 插入节点
        if (preNode.value > node.value) {
            preNode.left = node;
        } else if (preNode.value < node.value) {
            preNode.right = node;
        }
    }
    
    /**
     * 删除节点
     * */
    public remove(target: number) {
        if (!this.root) return;
        // 当前遍历的节点
        let curNode = this.root;
        // 要删除的目标节点是其父节点的左子节点还是右子节点
        let isLeft = true;
        // 要删除的目标节点
        let targetNode;
        // 要删除的目标节点的父节点
        let preNode = curNode;
        // 找到对应的节点
        while (curNode) {
            if (curNode.value === target) {
                targetNode = curNode;
                break;
            } else if (curNode.value > target) {
                preNode = curNode;
                isLeft = true;
                curNode = curNode.left;
            } else if (curNode.value < target) {
                preNode = curNode;
                isLeft = false;
                curNode = curNode.right;
            }
        }
        
        // 目标节点是叶子节点，无左右子节点，直接删除
        if (!curNode.left && !curNode.right) {
            if (isLeft) {
                preNode.left = null;
            } else {
                preNode.right = null;
            }
        }
        
        // 目标节点有左节点，无右节点
        if (curNode.left && !curNode.right) {
            if (isLeft) {
                preNode.left.value = curNode.left.value;
            } else {
                preNode.right.value = curNode.left.value;
            }
            curNode.left = null;
        }
        
        // 目标节点有右节点，无左节点
        if (!curNode.left && curNode.right) {
            if (isLeft) {
                preNode.left.value = curNode.right.value;
            } else {
                preNode.right.value = curNode.right.value;
            }
            curNode.right = null;
        }
        
        /**
         * 目标节点处于中间位置，存在左子节点也存在右子节点
         * 可以有以下两种处理方式：
         * 1、找到当前要删除节点左子树中最大的节点与要删除的节点进行替换
         * 2、找到当前要删除节点右子树中最小的节点与要删除的节点进行替换
         *
         * 以第一种方式为例，用即将删除节点的左子树中最大的节点与将要删除的节点替换，存在以下两种情况：
         * a.当前左子树根节点无右子树或者右节点：直接将左子树的根节点替换到删除的节点，并且将删除节点的右子树挂到当前左子树的根节点上
         * b.当前左子树根节点存在右子树或者右节点：找到左子树中的最大节点，用该最大节点去替换删除节点，并将删除节点的右子树挂到该最大节点上
         */
        if (curNode.left && curNode.right) {
            let tempCurNode = curNode.left;
            let tempPreNode = curNode;
            // 找到左子树中当前的最大值
            while (tempCurNode.right) {
                tempPreNode = tempCurNode;
                tempCurNode = tempCurNode.right;
            }
            if (tempPreNode === curNode) {
                // 当前左子树根节点无右子树或者右节点：直接将左子树的根节点替换到删除的节点，并且将删除节点的右子树挂到当前左子树的根节点上
                tempCurNode.right = curNode.right;
                if (isLeft) {
                    preNode.left = tempCurNode;
                } else {
                    preNode.right = tempCurNode;
                }
            } else {
                // 当前左子树根节点存在右子树或者右节点：找到左子树中的最大节点，用该最大节点去替换删除节点，并将删除节点的右子树挂到该最大节点上
                curNode.value = tempCurNode.value;
                tempPreNode.right = null;
            }
        }
    }
    
    /**
     * 搜索目标节点
     */
    public search(target: number): TreeNode | null {
        let node = this.root;
        while (node) {
            if (node.value === target) {
                return node;
            } else if (node.value > target) {
                node = node.left;
            } else if (node.value < target) {
                node = node.right;
            }
        }
        return null;
    }
    
    /**
     * 获取最小节点
     */
    public getMin(): TreeNode {
        let node = this.root;
        while (node.left) {
            node = node.left;
        }
        return node;
    }
    
    /**
     * 获取最大节点
     */
    public getMax(): TreeNode {
        let node = this.root;
        while (node.right) {
            node = node.right;
        }
        return node;
    }
}


/**
 * test
 */
const bst = new BinarySearchTree(10);
bst.insert(5);
bst.insert(15);
bst.insert(3);
bst.insert(8);
bst.insert(1);
bst.insert(4);
bst.insert(6);
bst.insert(9);
bst.insert(13);
bst.insert(20);
bst.insert(11);
bst.insert(25);

bst.remove(8);

BinarySearchTree.IN_ORDER_TRAVERSE(bst.getRoot());

```
