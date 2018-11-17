# 		半月记2018.11.17

这半个月在工作上没有百分之百投入，所以自己的时间多一些，但是效率并不高，后面下一版本的工作会展开，自己的时间就少一些，后面要强迫自己高效。

### 原计划

​	原本打算年前，即十、十一、十二和一月份四个月集中火力干数据结构与算法，只做easy和medium难度的题目，每天三题，年前可完成360题到400题左右。年后三月中旬以前完成100题，这样总题数目大概在450左右。

leetcode现在有933题，hard难度的约占15%，锁定的题目也有大概18%，所以还剩650道左右。

​	除刷题外，操作系统、网络也有很多要看。另外就是最近想试一试big data相关的东西，做底层的系统软件风险高、难度大，所以想接触下这块。所以刷题的进度要加快了，年前必须达到450道左右。



### 现状

​	刷题大概有一个月了，现在总共刷了88到题，主要集中在array和link list。正确率不高，每到题只是按照自己能想到的最好解法来解决，没有追求极致的效率。

![image](https://github.com/shaohv/shaohv.resources/raw/master/2018_fourth_quarter/half_month_2018.11.17_1.png)

#### linked list

​	问题比较少，总共34到题，以完成19道，基本是一次ac。

​	有下列几道题是需要注意的：

1. 逆序，这个属于基本功，很多问题都要在这个题目的基础上完成；
2. 判断是否有环，以及其升级版，判断有环，并且给出位置（没有做出来）；
3. O(nlogn) 原地排序（也没有作出来）；
4. deep copy linked list（没做出来）；



还有easy题目没有做，如果有关键的再补充到这里。作linked list题目，有几个小技巧：   

1. dummy node，避免head的特殊处理；
2. 快慢指针：在判断是否有环，删除最后第k个node这类题目中很有用；

#### array


​	数组一共有144题，其中hard难度20题，锁定14题，完成68题，大概完成easy &  hard难度的60%。这是第一边，没有能力也没打算一次全部去搞定，有些东西是融会贯通的，比如后面专门做了dp的题后，对dp理解可能更深入，回头来看array的题目，可能就有思路了。所以第一边60%，第二边90%这样去规划。

​	数组的题目应该是最灵活的（包括后面的string题目也应该非常灵活），除了需要dp, greedy, backtracking, d & c各种算法策略之外，偶尔也有一些窍门题目，例如求解数组中的major element，就需要用到摩尔投票来实现O(n)的复杂度解法。这些东西可能是各路大神历经多年总结分析出来的，不通过训练是不可能掌握的。

​	以下就已作的68道题来分类说明一下：

##### 分析规律型

[Container With Most Water](https://leetcode.com/problems/container-with-most-water) : 这个题首先一看，是有多个变量的，一个是纵轴的元素值，另一个是横轴两个元素之间的距离，这两者都影响most water计算；如果我们采取从数组首尾向中间靠拢，这样距离就是越来越小，我们只需要处理纵轴比当前值大的情况。

[Pascal's Triangle II](https://leetcode.com/problems/pascals-triangle-ii) : 三角型摆正成二维数组就一切明白了。

[Find Peak Element](https://leetcode.com/problems/find-peak-element) ：找peak point，那么在peak point前都是ascending趋势了。

##### 二分查找 ( binary search )

[Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix) : binary search可以用在有序或者局部有序的场景，可以求解target的位置，也可以求解target所在区间（target不存在数组中）。这道题要关键是如果用bs搜纵向值。

[Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii) : 局部有序场景使用bs。

[Find Minimum in Rotated Sorted Array II](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii) : 旋转后局部bs。

[Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array) ：bs找区间。

[Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array) ：bs找旋转点。

##### 双指针 ( two pointers )

[Remove Duplicates from Sorted Array II](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii)  : 两个指针，一个指向已经去重的，一个指向待去重的。

##### 回溯法 ( backtracking )

[3Sum Closest](https://leetcode.com/problems/3sum-closest) : 最直接的做法是backtracking，但backtracking这类方法虽然思路简单容易实现，但效率真的低，好多题backtracking后会time exceed。这道题backtracking简单优化后，勉强能ac，但更好的做法是two pointers，外层循环从前到后扫描数组，内层循环双指针来寻找target。

[Subsets II](https://leetcode.com/problems/subsets-ii) : 经典backtracking，这类题目有个需要注意的问题是，求解子集时不重复的方法。

[Combination Sum III](https://leetcode.com/problems/combination-sum-iii) ：经典回溯。

##### 动态规划 ( dynamic programming )

[Jump Game](https://leetcode.com/problems/jump-game) : 这道题我看来是dp，但leetcode的分类是greedy。

[Unique Paths II](https://leetcode.com/problems/unique-paths-ii) : 很经典的二维dp，对于学习dp很好。

[Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum) : 从左上到右下路径最小值，这也是二维dp，每一层缓存记录路径值。

[Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray) : 这道题我用第归做的，但是leetcode的分类是dp。

[Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii) : 经典dp。

​	题目不一一列举，把经典的列出来。已经做的这些题，出于对算法的理解有限，可能有些分类可能不明确，甚至不正确，只能等着后面熟悉后，再来重新分析一下。
### 下一步

​	接下来，计划把queue、stack和部分hash table的题目作掉。希望未来半个月能突破80道。