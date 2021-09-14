---
title: Find the maximum width of binary tree
date: 2020-01-17T10:08:22.921Z
categories: [Coding Problems]
tags: [binary-tree, maximum-width]
cover: /images/find-the-maximum-width-of-binary-tree/featured.png
---

Given a Binary tree, how will you find the maximum width of Binary Tree?
<!-- more -->

![](/images/find-the-maximum-width-of-binary-tree/problem.png)

For example, for this Binary tree it has maximum width is `4`.

### Solution
* We can use level order traversal along with queue to find maximum width.
* Create an empty queue and start from root node.
* Loop while queue is not empty.
    * Update `maxWidth` if count of nodes in queue is greater.
    * Remove all current level nodes from queue.
    * Add all next level nodes into the queue. 


#### Algorithm - To find the maximum width of binary tree

```java
class Algorithm {

    /**
     * We can use level order traversal along with queue to find maximum width.
     *
     * @param tree binary tree
     * @return maximum width of binary tree
     */
    public int maxWidth(BinaryTree tree) {

        Queue<Node> nodes = new LinkedList<>();
        int maxWidth = 0;
        nodes.add(tree.getRoot());

        while (!nodes.isEmpty()) {

            int count = nodes.size();

            maxWidth = Math.max(maxWidth, count);

            while (count-- > 0) {
                Node node = nodes.remove();

                if (node.getLeft() != null) {
                    nodes.add(node.getLeft());
                }

                if (node.getRight() != null) {
                    nodes.add(node.getRight());
                }

            }
        }

        return maxWidth;
    }

}
```

#### Test Scenario

```java
public class AlgorithmTest {

    @Test
    public void canEvaluateMaxWidthOfBinaryTree1() {
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

        assertEquals(4, new Algorithm().maxWidth(tree));
    }

    @Test
    public void canEvaluateMaxWidthOfBinaryTree2() {
        BinaryTree tree = new BinaryTree(new Node(1));

        tree.insert(new Node(2));
        tree.insert(new Node(3));
        tree.insert(new Node(4));
        tree.insert(new Node(5));
        tree.insert(new Node(6));
        tree.insert(new Node(7));
        tree.insert(new Node(8));
        tree.insert(new Node(9));
        tree.insert(new Node(10));
        tree.insert(new Node(11));
        tree.insert(new Node(12));
        tree.insert(new Node(13));
        tree.insert(new Node(14));
        tree.insert(new Node(15));

        assertEquals(8, new Algorithm().maxWidth(tree));
    }

}
```
