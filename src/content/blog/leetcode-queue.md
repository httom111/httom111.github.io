---
title: 'LeetCode-Queue'
description: '队列与优先队列的基础介绍，以及用单调队列解决滑动窗口最大值问题的完整推导。'
pubDate: '2023-01-09'
tags: ['LeetCode', '队列']
---

# 队列题目

## Introduction

### 队列

队列是一种常见的数据结构，基础的队列应该有以下两种API

```java
public void push(int val);

public int pop();
```

队列遵循先进先出原则，则每个进去的push进去的元素在pop时，顺序是一样的。
比如一个队列Q进行了如下操作

```java
Q.push(1);
Q.push(2);
Q.push(3);
println(Q.pop());
println(Q.pop());
println(Q.pop());
```

则返回的结果应该是与push的顺序一致，为1,2,3。

### 优先队列

优先队列是一种特殊的队列(虽然按照我个人的理解这玩意和队列一点关系没有)。他的api操作和普通的队列一样，但是唯一的区别在于，他会按照某种顺序(由程序员定义，比如大小序，字节序等)来保存你push进去的数据，每次返回时会返回这个顺序的最大值或者最小值（由程序员定义）。

最常见的优先队列的实现为Heap，是一种以二叉树为基础的实现。
以下是所有常见的优先队列各种操作的时间复杂度：

![优先队列各操作复杂度](/images/leetcode/queue-time-heap.png)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">引用：<a href="https://en.wikipedia.org/wiki/Priority_queue">维基百科</a></center>

## Main Content

题目：239. 滑动窗口最大值
[LeetCode 239](https://leetcode.cn/problems/sliding-window-maximum/comments/)
首先最明显的解法是暴力解法，即每次循环k个数据算出最大值。这样的时间复杂度为O(k*n).

我们可以用优先队列来减少时间复杂度。根据题目要求，k的值不会小于数组长度nums.length，所以我们先把nums中前k个数据装入一个大顶堆(从大到小排序的优先队列)。然后每当窗口滑动一次后，就把队列中不在窗口的值去掉，加入新的值，再把当前队列的最大值记录下来，直到窗口到末尾，返回所有记录的最大值。这样的好处是可以保证每次队列中只有窗口中的值。如下所示

```
数据：3,3,4,1,2,6,5,1. 窗口大小为3.

[3,3,4],1,2,6,5,1  List:4,3,3 -> 4
3,[3,4,1],2,6,5,1  List:4,3,1 -> 4
3,3,[4,1,2],6,5,1  List:4,2,1 -> 4
3,3,4,[1,2,6],5,1  List:6,2,1 -> 6
3,3,4,1,[2,6,5],1  List:6,2,5 -> 6
3,3,4,1,2,[6,5,1]  List:4,5,1 -> 5
```

假设使用的是基于二插树的优先队列，每次维护队列的复杂度为O(log(n))，一共要进行n次操作，所以整个的时间复杂度为O(nlog(n)).

那么，我们有没有什么办法可以让每次维护队列时的复杂度降低为O(1)，来使整个算法的时间复杂度更低呢？通过观察，我们可以发现我们并不用时刻保持队列有k个数据，只要保持有当前窗口的最大值，以及可能成为后续窗口最大值的值即可。
比如，我有以下数据：

```
3,3,4,1,2,6,5,1. 窗口大小为3.

当窗口覆盖后前三个元素后，我们有只需要存储4即可，因为前面的3我一定不会成为后续的最大值。
[3,3,4],1,2,6,5,1  List:4
当窗口向后滑动后
3, [3,4,1],2,6,5,1
```

此时，该窗口的最大值认为4，但是，因为4在1的前面，当窗口离开4时，1有可能成为后续窗口的最大值，所以此时我们需要把1也保存起来。
简单来说，每次向list插入数据A时，我们把所有list中小于A的值都去掉，把A插入到list的最末尾，这种数据结构称作为单调队列。
整个流程如下：

```
数据：3,3,4,1,2,6,5,1. 窗口大小为3.

[3,3,4],1,2,6,5,1  List:4 -> 4
3,[3,4,1],2,6,5,1  List:4,1 -> 4
3,3,[4,1,2],6,5,1  List:4,2 -> 4
3,3,4,[1,2,6],5,1  List:6 -> 6
3,3,4,1,[2,6,5],1  List:6,5 -> 6
3,3,4,1,2,[6,5,1]  List:6,5 -> 5
```

实现如下

```java
private class MyQueue239 {
    private Deque<Integer> deque;
    public MyQueue239() {
        deque = new LinkedList<>();
    }

    public void pop(int value) {
        if (max() == value) {
            deque.removeFirst();
        }
    }

    public void add(int value) {
        while (!deque.isEmpty() && deque.getLast() < value) {
            deque.pollLast();
        }
        deque.addLast(value);
    }

    public int max() {
        return deque.getFirst();
    }
}
```

有了这个数据结构，整个算法的实现就很简单，我们只要遍历整个数据，不断的把尾数据加入单调队列同时去掉已经离开队列中的值，记录每次滑动时的最大值即可。
代码如下：

```java
public int[] maxSlidingWindow(int[] nums, int k) {
        MyQueue239 myQueue239 = new MyQueue239();

        int[] res = new int[nums.length - k + 1];
        int index = 0;
        int tail = 0;
        for (; tail < k; tail++) {
            myQueue239.add(nums[tail]);
        }
        res[index] = myQueue239.max();
        myQueue239.pop(nums[index++]);

        for (; tail < nums.length; tail++) {
            myQueue239.add(nums[tail]);
            res[index] = myQueue239.max();
            myQueue239.pop(nums[index++]);
        }
        return res;
    }
```
