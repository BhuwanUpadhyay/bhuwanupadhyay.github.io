---
title: Find the vertical sum of binary tree
date: 2020-01-07T10:08:22.921Z
categories:
- CodingProblems
tags:
- binary-tree
cover: /images/find-the-vertical-sum-of-binary-tree/featured.png
---

Given a Binary tree, how will you find the Vertical Sum of Binary Tree?
<!-- more -->

![](/images/find-the-vertical-sum-of-binary-tree/problem.png)

For example, for this Binary tree it has `5` vertical lines.
For line `3` the sum will be `5+7+8=20`.

### Solution
* We need to check the horizontal distance (HD) from root for all nodes.
* HD for root is 0.
* For right child we will +1 (add 1) to HD
* For left child we will -1 (subtract 1) to HD
* We can easily maintain a hash map for horizontal distance corresponding to each vertical line.
* Then, we can traverse the Binary Tree and update our hash map.

#### Algorithm - To find the vertical sum of binary tree

```java
class Algorithm {

    /**
     *
     * @param tree binary tree
     * @param line vertical line number
     * @return sum of those nodes falls under that vertical line
     */
    public int sumOfVerticalLine(BinaryTree tree, int line) {
        Map<Integer, Integer> sums = new LinkedHashMap<>();

        Queue<TraversalNode> nodes = new LinkedList<>();
        nodes.add(new TraversalNode(tree.getRoot(), 0));

        while (!nodes.isEmpty()) {

            TraversalNode node = nodes.remove();
            Integer value = (Integer) node.node.getValue();
            sums.put(node.hd, sums.getOrDefault(node.hd, 0) + value);

            if (node.node.getLeft() != null) {
                nodes.add(new TraversalNode(node.node.getLeft(), node.hd - 1));
            }

            if (node.node.getRight() != null) {
                nodes.add(new TraversalNode(node.node.getRight(), node.hd + 1));
            }
        }

        List<Integer> hds = sums.keySet().stream().sorted().collect(Collectors.toList());
        return sums.get(hds.get(line - 1));
    }

    public static class TraversalNode {
        private Node node;
        /**
         * horizontal distance of node
         */
        private int hd;

        public TraversalNode(Node node, int hd) {
            this.node = node;
            this.hd = hd;
        }
    }

}
```

#### Test Scenario

```java

public class AlgorithmTest {

    @Test
    public void canEvaluateVerticalSumOfBinaryTree1() {
        BinaryTree tree = new BinaryTree(new Node(5));

        tree.insert(new Node(4));
        tree.insert(new Node(9));
        tree.insert(new Node(3));
        tree.insert(new Node(7));
        tree.insert(new Node(8));
        tree.insert(new Node(6));

        assertEquals(20, new Algorithm().sumOfVerticalLine(tree, 3));
    }


    @Test
    public void canEvaluateVerticalSumOfBinaryTree2() {
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

        assertEquals(21, new Algorithm().sumOfVerticalLine(tree, 3));
    }

}
```
