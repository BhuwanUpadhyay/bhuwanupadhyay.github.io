---
title: How to create binary tree in Java
date: 2020-01-05T10:08:22.921Z
categories: [Coding Problems]
tags: [binary-tree]
cover: /images/how-to-create-binary-tree-in-java/featured.png
---

Given a Binary tree, how to implement this binary tree in java?
<!-- more -->

![](/images/how-to-create-binary-tree-in-java/problem.png)

For example, for this Binary tree it has `8` nodes.

#### Tree Node

```java
/**
 * Tree Node Class
 */
class Node {
    private Object value;
    private Node left;
    private Node right;

    public Node() {

    }

    public Node(Object value) {
        this.value = value;
    }

    public Object getValue() {
        return value;
    }

    public void setValue(Object value) {
        this.value = value;
    }

    public Node getLeft() {
        return left;
    }

    public void setLeft(Node left) {
        this.left = left;
    }

    public Node getRight() {
        return right;
    }

    public void setRight(Node right) {
        this.right = right;
    }
}

```

#### Binary Tree
```java
/**
 * Binary tree
 */
class BinaryTree {
    private Node root;

    /**
     * Create new binary tree with root node
     *
     * @param root Root node
     */
    public BinaryTree(Node root) {
        this.setRoot(root);
    }

    /**
     * Create new empty binary tree
     */
    public BinaryTree() {
        this.setRoot(null);
    }


    public Node getRoot() {
        return root;
    }

    public void setRoot(Node root) {
        this.root = root;
    }

    /**
     * Insert new node into the tree. <br>
     * This method will insert new node into last tree level, until last tree level is full, then add new level.
     *
     * @param newNode new tree node
     */
    public void insert(Node newNode) {
        if (this.root == null) {
            this.root = newNode;
        } else if (this.root.getLeft() == null) {
            this.root.setLeft(newNode);
        } else if (this.root.getRight() == null) {
            this.root.setRight(newNode);
        } else {
            List<Node> siblingNodes = new LinkedList<>();
            siblingNodes.add(this.root.getLeft());
            siblingNodes.add(this.root.getRight());
            insertSiblings(siblingNodes, newNode);
        }
    }

    /**
     * Check a level sibling nodes, find the node which dones't have left or right child, then insert the new node
     *
     * @param siblingNodes List of current level tree nodes
     * @param newNode      new tree node
     */
    private void insertSiblings(List<Node> siblingNodes, Node newNode) {
        List<Node> nextSiblingNodes = new LinkedList<>();
        for (Node currentNode : siblingNodes) {
            if (currentNode.getLeft() == null) {
                currentNode.setLeft(newNode);
                return;
            } else if (currentNode.getRight() == null) {
                currentNode.setRight(newNode);
                return;
            }
            nextSiblingNodes.add(currentNode.getLeft());
            nextSiblingNodes.add(currentNode.getRight());
        }
        insertSiblings(nextSiblingNodes, newNode);
    }

}
```

#### Test Scenario

```java
public class BinaryTreeTest {

    @Test
    public void canCreateBinaryTree1() {
        BinaryTree tree = new BinaryTree(new Node(5));

        tree.insert(new Node(4));
        tree.insert(new Node(9));
        tree.insert(new Node(3));
        tree.insert(new Node(7));
        tree.insert(new Node(8));
        tree.insert(new Node(6));
        tree.insert(new Node(5));

        assertNotNull(tree.getRoot());
        assertEquals(5, tree.getRoot().getValue());
        assertEquals(4, tree.getRoot().getLeft().getValue());
        assertEquals(9, tree.getRoot().getRight().getValue());
    }


    @Test
    public void canCreateBinaryTree2() {
        BinaryTree tree = new BinaryTree(new Node(5));

        tree.insert(new Node(4));
        tree.insert(new Node(9));
        tree.insert(new Node(3));
        tree.insert(new Node(7));

        Node node = new Node(8);
        node.setLeft(new Node(5));
        
        tree.insert(node);
        tree.insert(new Node(6));
        tree.insert(new Node(5));

        assertNotNull(tree.getRoot());

        assertEquals(1, tree.getRoot().getValue());
        assertEquals(2, tree.getRoot().getLeft().getValue());
        assertEquals(3, tree.getRoot().getRight().getValue());
    }
    
}
```
