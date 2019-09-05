# leetcode 关键问题题解

## 腾讯50题

### [Reverse integer](https://leetcode-cn.com/problems/reverse-integer/) 

​	目前的办法是通过字符串比较来判断是否溢出，更高效的解法是边转换边判断。整体容易溢出不好控制，分布来判断就很安全。

### [Palindrome Number](https://leetcode-cn.com/classic/problems/reverse-integer/description/) 

​	最直接的方法将数字取模之后放入vector，双指针进行比较。但这也是低效的办法。

### [Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/description/)

​	要尝试一些其他方法，看看效率，如二分法

### [remove dumplicates from sorted array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/) : two pointer

​	这道题自己用朴素的方法作的，会出一些边界问题，漏洞多。但是如果改用推荐的solution的two pointer来作，就不会出问题。还是对two pointer理解的不够。

### [Maximum Subarray](https://leetcode.com/problems/maximum-subarray/description/):  一维DP

​	解一：

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int sum = 0;
        int max = INT_MIN;
        
        for(int i = 0; i < nums.size(); i++){
            sum += nums[i];
            
            max = max > sum ? max : sum;
            
            sum = sum < 0 ? 0 : sum;
        }
        
        return max;
    }
};
```



​	解法二：

```c++
dp[i]: maximum sum including nums[i]
dp[i] = dp[i-1] + nums[i] if dp[i-1] >= 0
dp[i] = nums[i] if dp[i-1] < 0;

class Solution {
    public int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        int global_max = nums[0];
        int dp = nums[0];
        for (int i = 1; i < nums.length; i++) {
            if (dp < 0) {
                dp = nums[i];
            } else {
                dp += nums[i];
            }
            if (dp > global_max) {
                global_max = dp;
            }
        }
        return global_max;
    }
}
```



​		其中解法一是自己的，想了很久才想到，以为是一道数学思维题目。看到题解二，用DP思想很轻松。

### DP问题要素

​	分解、没有后向性。

### [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/solution/) : 一维DP

```c++
class Solution {
public:
    // n = 1  1
    // n = 2  2 可能从第一级走1步，从开始直接走两步
    // n = 3  1 + 2  要么从step1走两步，要么从step2走一步（step1走1步再走1步已经包含在第二种情况）
    // f(n) = f(n-1) + f(n - 2)
    int climbStairs(int n) {
        int a = 1, b = 2;
        if(n <= 2) return n;
        
        for(int i = 3; i <= n ; i++){
            int tmp = a + b;
            a = b;
            b = tmp;
        }
        
        return b;
    }
};
```

​	官方题解才是DP的正规思路，先暴力 -> 画图 -> 记忆式搜索 -> DP

### [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/)

​		c++ stl vector的new，二维数组用法

解法一：暴力法

```c++
    // f(x) = max{arr{x} - arr{i}}, i = 0 ... x-1
    // max profit = max(f(x), x = 1 ... arrsize -1 )
int maxProfit(vector<int>& prices) {
    if(prices.size() < 2) return -1;
    vector<int> profit(prices.size());
    int max_p = 0;


    for(int i = 1; i < prices.size() - 1; i ++){
        int max_profit = INT_MIN;
        for(int j = 0; j < i; j++){
            max_profit = max(max_profit, prices[i] - prices[j]);
        }
        profit[i] = max_profit;
        max_p = max(max_p, profit[i]);
    }

    return max_p;
}
```

解法二：一维DP

```c++
   // f(x) = max{arr{x} - arr{i}}, i = 0 ... x-1
    // f(x+1) = max(f(x) + arr(x+1) - arr(x), arr(x+1) - arr(x))
    // max profit = max(f(x), x = 1 ... arrsize -1 )
    int maxProfit(vector<int>& prices) {
        if(prices.size() < 2) return 0;
        vector<int> profit(prices.size());
        profit[0] = 0;
        int max_p = 0;
        
        
        for(int i = 1; i < prices.size(); i ++){
            int t = prices[i] - prices[i-1];
            profit[i] = max(profit[i-1] + t, t);
            max_p = max(max_p, profit[i]);
        }
        
        return max_p;
    }
```

解法三：空间优化

```c++
    // f(x) = max{arr{x} - arr{i}}, i = 0 ... x-1
    // f(x+1) = max(f(x) + arr(x+1) - arr(x), arr(x+1) - arr(x))
    // max profit = max(f(x), x = 1 ... arrsize -1 )
public:
    int maxProfit(vector<int>& prices) {
        if(prices.size() < 2) return 0;
        int first = 0;
        int second = 0;
        int max_p = 0;
        
        
        for(int i = 1; i < prices.size(); i ++){
            int t = prices[i] - prices[i-1];
            second = max(first + t, t);
            first = second;
            max_p = max(max_p, second);
        }
        
        return max_p;
    }
```

​		层层递进，首先是暴力法搜索，发现有重叠部分，再思考重叠部分关系（这一步最重要），最后优化空间。