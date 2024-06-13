---
title: POJ 2456 Aggressive Cow
date: 2020-04-03 12:43:00 +0800
tags: 
- 二分搜索
img: ./software.jpg
---

## **题目信息**

作者：不详
链接：[http://poj.org/problem?id=2456](https://link.zhihu.com/?target=http%3A//poj.org/problem%3Fid%3D2456)
来源：PKU JudgeOnline

### **Aggressive cows**[[1\]](https://zhuanlan.zhihu.com/p/120450034#ref_1)

Time Limit: 1000MS
Memory Limit: 65536K

**描述**

Farmer John has built a new long barn, with N (2 &lt;= N &lt;= 100,000) stalls. The stalls are located along a straight line at positions x1,...,xN (0 &lt;= xi &lt;= 1,000,000,000).
His C (2 &lt;= C &lt;= N) cows don't like this barn layout and become aggressive towards each other once put into a stall. To prevent the cows from hurting each other, FJ want to assign the cows to the stalls, such that the minimum distance between any two of them is as large as possible. What is the largest minimum distance?

**输入**

- Line 1: Two space-separated integers: N and C
- Lines 2..N+1: Line i+1 contains an integer stall location, xi

**输出**

- Line 1: One integer: the largest minimum distance

**样例输入**

```text
5 3
1
2
8
4
9
```

**样例输出**

```text
3
```

**提示**

OUTPUT DETAILS:
FJ can put his 3 cows in the stalls at positions 1, 4 and 8, resulting in a minimum distance of 3.
Huge input data,scanf is recommended.

------

## 题目解读与分析

（如果您已熟练掌握二分搜索，可以跳过本段与下一段，直接浏览代码）

- **这是一个搜索问题，适宜使用二分搜索**

首先观察样例输入，输入的牛棚坐标不一定有序，反手一个快速排序，闷声发大财（逃

题目的意图是，给定一系列正整数 ![[公式]](https://www.zhihu.com/equation?tex=x_%7B1%7D%2Cx_%7B2%7D..x_%7Bn%7D) ，最大化其中任意 ![[公式]](https://www.zhihu.com/equation?tex=c) 个数中两两之差的最小值（以下用 D 表示）。能不能直接利用输入的数据计算出这个“最大的最小值”呢？以我浅薄的智慧想不到可行的解决方案。相比起来，通过多次试探找出合理的 D 无疑更有可操作性。因此，这是一个搜索问题，或者至少将其当做一个搜索问题来对待。

- **为什么采用二分搜索？**

现在我们需要在1~ ![[公式]](https://www.zhihu.com/equation?tex=D_%7Bmax%7D) 之间试出一个满足题意的 D 。

最简单粗暴的方法就是从![[公式]](https://www.zhihu.com/equation?tex=D_%7Bmax%7D) 开始降序验证 D 是否可行，并输出第一个可行的 D，但是这样的线性搜索太慢了。输入n个牛棚的坐标，最大的计算次数就是 ![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B10%5E%7B9%7D%7D%7BC%7D%5Ccdot+n) ，约等于 ![[公式]](https://www.zhihu.com/equation?tex=10%5E%7B9%7D) 次，会超时。所以需要使用二分搜索。

- **怎么验证一个D是否可行？**

如何确定一个D是否可行呢？我们只要把第一头牛放在第一个牛棚里，把第二头牛放在 D 距离以外第一个遇到的牛棚，以此类推，最后看一看能不能放得下所有的牛。显然，D 有个不可逾越的上界，就是假设所有牛在总长度上（最右牛棚 - 最左牛棚）均匀分布时的间距，这个值就是 ![[公式]](https://www.zhihu.com/equation?tex=D_%7Bmax%7D) 。

- **二分搜索**
  
  - **如何通过二分搜索寻找最优解？**
      对区间[ L, R ]，当中间值mid不可行时，所有比mid大的D都不可行，右界左移开始下一轮；当mid可行时，不能确定mid右侧是否还有更大的可行解，因此用一个变量暂存当前可行解，然后左界右移，开始下一轮。当二分循环结束时，暂存的可行解即为所求最优解。
  - **初始右界的确定**
    \#——#——#——#——#——#
    0——1——2——3——4——5
    根据植树原理，右边界，即均匀分布时的距离，等于总长度 ![[公式]](https://www.zhihu.com/equation?tex=%5Cdiv) （牛的头数 - 1）
  - **边界的移动**
    左边界右移时，如果将left更新为mid，当left与right相差1时，mid = (left + right) / 2的值总等于left，会引起死循环，所以应该将left更新为mid+1
    由于数组是升序的，右边界左移时，不需考虑。想一想，这是为什么？

------

## 细节说明

- **输入方式**
  题目中提到“Huge input data,scanf is recommended.”，因此使用scanf_s而不是cin输入
- **命名与程序结构**
  为了增加程序的可读性、可修改性，可以将所有变量与函数都用英文单词（一些含义明确的局部变量除外）命名，并且将二分搜索的过程、验证最小距离D可行性的过程，抽象为函数。详见参考题解
- **排序**
  除非题目要求或有特殊需求，否则应尽量使用内置的排序。一般来说，语言、编译器提供的排序算法更安全、更高效

------

## 参考题解

[完整代码](./poj2456.7z)如下，仅供参考

```cpp
#include <iostream>
#include <cstdio>
#include <cstdlib>
using namespace std;

//global
const int MAX_N = 100000;
const int MAX_STALL = 1000000000;
int N = 0,C = 1;
int stall[MAX_N] = {0};

//prototype
bool IsDOK(int D);
int SearchLargestMinDistance(int left, int right);
int cmpInt(const void * a, const void * b);
void Init();

int main()
{
    Init();
    int L = 1, R = (stall[N-1] - stall[0]) / (C - 1);
    int result = SearchLargestMinDistance(L, R);
    cout << result;

    return 0;
}

void Init()
{
    //input boundary data
    cin >> N >> C;

    //error check
    if(N > MAX_N || N < C || C < 2)
    {
        cout << "Input error! Out of range." << endl;
        exit(-1);
    }

    //input stall data and sort it
    for(int i = 0;i<N;i++)
    {
        scanf_s("%d",&stall[i]);
        if(stall[i] > MAX_STALL || stall[i] < 0)
        {
            cout << "Input error! Out of range." << endl;
            exit(-1);
        }
    }

    qsort(stall, N, sizeof(int), cmpInt);
}


int SearchLargestMinDistance(int left, int right)
{
    //init
    int mid = (left + right) / 2;
    int result = 0;

    //binary search
    while(right - left >= 0)
    {
        if(IsDOK(mid))
        {
            left = mid + 1;
            result = mid;
        }
        else
        {
            right = mid - 1;
        }
        mid = (left + right) / 2;
    }

    return result;
}

bool IsDOK(int D)
{
    //init
    int i = 0,p = stall[0];
    int restCow = C - 1;

    for(i = 1;i<N;i++)
    {
        if(stall[i] >= p + D)
        {
            p = stall[i];
            restCow--;
        }
    }

    return restCow <= 0;
}

int cmpInt(const void * a, const void * b)
{
    return ( *(int*)a - *(int*)b );
｝
```

## 参考

1. Aggressive cows http://poj.org/problem?id=2456
