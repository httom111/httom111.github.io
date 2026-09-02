---
title: LeetCode_KMP
published: 2023-02-15
description: KMP 算法的前缀表构建思路与字符串匹配算法的完整推导，附 Java 实现。
tags: [LeetCode, KMP, 字符串]
category: 算法
---

# KMP算法

最近在刷题时遇到了KMP算法的题目，感觉挺有意思的所以在这里记录一篇博客来大概讲一下自己的理解，并且会给到具体的代码。

## 使用场景

## 算法原理

## 前缀表

前缀表是用来会退的，它记录了当模式串与主串不匹配时，模式串应该从哪里重新开始匹配。

## 求KMP的next数组

先在前面介绍next数组是干嘛的，每个下标代表什么意思

首先最直观的算法是直接用for循环暴力递归出每个index的下标，但是这样没有很好的利用前缀表的特性，**即每个index下标表示在整个index之前的字符串中，有多大长度相同的前后缀**。

我们可以使用这个特性来继续在递归中使用已经计算过的数据来节省我们计算时间的开销。

### 实现思路

首先，我假设要计算一个字符串 StringA 的前缀表。 首先，我们构建一个新的int数组来作为我们的返回值。同时建立一个char的数组来方便我们后面的使用（这个属于个人习惯）

```java
int[] res = new int[StringA.length()];
char[] charArrays = StringA.toCharArray();
```

然后，我们设置一个新的变量j = 0来表示当前相同长度的前缀的尾部。一个变量i = 1表示当前相同长度的后缀的尾部。
如果一开始直接把j 和 i都设置为0没有任何意义，因为此时i和j永远相等，所以我们直接默认把0的位置的值设置为0即可。

然后，我们使i不断向后遍历。

```java
int[] res = new int[StringA.length()];
char[] charArrays = StringA.toCharArray();

int j = 0;
res[0] = 0;
for (int i = 1; i < StringA.length(); i++) {
    ....
}
```

在每次循环中，我们比较字符串在位置i与j的元素是否相等，如果相等，表示目前的前缀与后缀的尾部是匹配的。我们可以把j加一，因为此时相等前后缀的长度为1.

```java
if (charArrays[i] == charArrays[j]) {
    j++;
}
```

如果当当前i和j的元素不等时，如果此时j不为0，因此我们需要回溯到"j"的先前值（存储在"res"数组中）以找到与当前字符匹配的适当后缀。即 res[j-1]，所以我们不断重复这个过程，直到charArrays[i] = charArrays[j]，或者j 为 0.

```java
while (charArrays[i] != charArrays[j] && j != 0) {
    j = res[j - 1];
}
```

处理完这个后，我们就可以写入当前的j到res[i]中了，因为当前j就是字符串s[0…i]的相等前后缀的长度了。

```java
int[] res = new int[StringA.length()];
char[] charArrays = StringA.toCharArray();

int j = 0;
res[0] = 0;
for (int i = 1; i < StringA.length(); i++) {
    while (charArrays[i] != charArrays[j] && j != 0) {
        j = res[j - 1];
    }

    if (charArrays[i] == charArrays[j]) {
        j++;
    }

    res[i] = j
}

return res;
```

---

# KMP算法（续）

## 思路

KMP的基本思路是，用前缀表来简化搜索的过程，即不用重复检查已经检查过的地方。例如我有以下文本串和模式串。

文本串: aabaabaafa
模式串: aabaaf

我们可以先构建一个模式串的前缀表，即
[0, 1, 0, 1, 2, 0]

然后，开始同时遍历文本串和模式串，用指针i和j两两比较大小。当两个串的当前元素都匹配时，往下遍历。当两个元素不匹配时，我们需要回溯模式串的指针到该指针前一个字符的前缀表的数值。

该GIF来源为：[programmercarl.com](https://www.programmercarl.com/0028.%E5%AE%9E%E7%8E%B0strStr.html#%E5%85%B6%E4%BB%96%E8%AF%AD%E8%A8%80%E7%89%88%E6%9C%AC)

![KMP 算法演示](./images/kmp.gif)

为什么我们可以直接回到当前指针前一个字符的前缀表值就可以了呢？是因为我们要检查模式串前面的前缀是否与当前文本串前面的后缀匹配。如果找到了匹配的部分，我们可以回溯j到该前缀，从而避免重新匹配前缀。

所以整体算法如下：

```java
public int strStr(String haystack, String needle) {
    int[] prefixTable = buildPrefixTable(needle);

    char[] hayCharArray = haystack.toCharArray();
    char[] needleArray = needle.toCharArray();
    int j = 0;
    for (int i = 0; i < haystack.length(); i++) {
        while (j > 0 && hayCharArray[i] != needleArray[j]) {
            j = prefixTable[j - 1];
        }

        if (hayCharArray[i] == needleArray[j]) {
            j++;
        }

        if (j == needleArray.length) {
            return i - needleArray.length + 1;
        }
    }
    return -1;
}
```
