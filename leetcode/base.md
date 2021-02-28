# 算法:
## 排序
以下四个排序建议手撕:
1. 选择排序
    在`[i+1, len)`这个区间找出小于i的值,与i做交换;
2. 插入排序
    将i倒序判断插入`[0, i-1]`这个区间的合适位置;因为能提前break,所以性能优于选择排序;有一种优化方式,建议手撕理解.
3. 归并排序
    有自顶向下和自底向上两种方式;自底向上归排性能略弱于自顶向下,但是没有用到数组的key值,也就是说可以用在链表上.
    在数据量小的时候建议使用插入排序(例如小于15个数);注意取中间值用减法来实现;
    
4. 快速排序
    partition操作+随机化;对近似有序的数组退化为O(n^2),数据量小的时候建议用插排代替.

选择、插入是O(n^2)级别, 归并快排是O(nlogn)级别


5. 对时间复杂度的理解:


6. 线段树了解吗？字典树？

7. 几种基本排序算法说一下，问了堆的时间复杂度，稳定性，为什么不稳定

8. topk问题，海量数据topk（回答成切分多次加载内存，然后用维持k长度的有序链表，然后被说时间复杂度不好，提示说还是用堆

9. 实现一个hashmap，解决hash冲突的方法，解决hash倾斜的方法

10. 判断二叉树是否为满二叉树

11. 链表和数组相比, 有什么优劣？

12. 如何判断两个无环单链表有没有交叉点?如何判断两个有环单链表有没有交叉点?如何判断一个单链表有没有环, 并找出入环点?

13. 


## 问题
1. 给定一个集合，求集合的子集

2. 求两个树的共同子树

3. 如何判断一个树是另一个树的子树

4. 自旋锁是什么，用过吗

5. 如果希望既有顺序，又可以快速访问，你会选择什么数据结构

6. TreeMap的原理说一下

7. 在1个10G大小的文件中，存储的都是int型的数据，如何在内存使用小于8M的情况下进行排序

8. 给你一个10PB文件  3000台机器。如何做字典树排序

9. 100亿个数选top5，小根堆

10. 从无限的字符流中, 随机选出 10 个字符, 没见过也没想出来，查了一下是蓄水池采样算法，经典面试题，没刷题吃亏了

11. 算法题, M*N 横向纵向均递增的矩阵找指定数, 只想到 O(M+N)的解法 补充: 这几天刷 leetcode 碰到这题了, 240. Search a 2D Matrix II. 办法是从左下角或右下角开始查找.

12. 算法题: N 场演唱会, 以 [{startTime, endTime}…] 的形式给出, 计算出最多能听几场演唱会,先讲了思路, 按 endTime 升序排列，再顺序取最多场次,(讲完思路之后)屏幕共享给我, 用你最熟悉的语言把这个算法实现,用 go 实现了一版

    你用了贪心法, 贪心可能会存在什么问题?局部最优，在这个问题里，只能找到一个可能解，无法找到所有排列方式
