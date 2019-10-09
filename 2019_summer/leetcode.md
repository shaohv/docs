# leetcode 关键问题题解

## [腾讯50题](https://leetcode-cn.com/problemset/50/)

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

### [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/):一维DP

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

### [Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/description/)

​	它是一道DP吗

### [Reverse Words in a String III](https://leetcode.com/problems/reverse-words-in-a-string-iii/) : 属于基础套路题

```java
class Solution {
    private void reverseCharArr(char[] cs, int left, int right){
        while(left < right){
            char t = cs[left];
            cs[left] = cs[right];
            cs[right] = t;
            left ++; right --;
        }
    }
    
    public String reverseWords(String s) {
        if(s.length() == 0) return s;
        
        char[] rs = s.toCharArray();
        int begin = 0;
        for(int i=0; i<rs.length; i++){
            if(rs[i] == ' '){
                reverseCharArr(rs, begin, i-1);
                begin = i + 1;
            }
        }
        reverseCharArr(rs, begin, rs.length - 1);       
        return String.valueOf(rs);
    }
}
```

​		至此，腾讯50题，simple级别结束。

### [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring)

```c++
class Solution {
private:
    bool helper(string& s, int i, int j){
        while(i < j){
            if(s[i]!= s[j]){
                return false;
            }
            i++; j--;
        }
        
        return true;
    }
public:
    string longestPalindrome(string s) {
        if(s.length() < 2) return s;
        int left = 0;
        int len = s.length();
        int slen = 1;
        bool found = false;
        
        for(; !found && (len > 0); len --){
            for(int i = 0; i+len-1 < s.length(); i++){
                if(helper(s, i, i+len-1)){
                    left = i;
                    slen = len;
                    found = true;
                    break;
                }                
            }
        }
        
        return s.substr(left, slen);
    }
};
```

三层循环，即使题目限制了长度最大为1000，也time limit exceed.

换一种搜索方式，对每一个元素，往两边搜索

```cpp
void helper(string& s, int left, int right, int& start, int& maxi){
    if(s[left] != s[right]) return;
    while(left > 0 && right < s.length()-1 && s[left-1] == s[right+1]){
        left--; right ++;
    }
    
    if(right - left + 1 > maxi){
        start = left;
        maxi = right - left + 1;
    }
    return;
}

string longestPalindrome(string s) {
    int len = s.length();
    if( len < 2 ) return s;
    
    int maxi = 1, start_idx = 0;
    for(int i = 0; i < len; i++){
        helper(s, i, i, start_idx, maxi);
        helper(s, i, i+1, start_idx, maxi);
    }
    
    return s.substr(start_idx, maxi);
}
```

这就能满足时间要求。

另外还有dp算法，你会写吗

### [3sum](https://leetcode.com/problems/3sum):Two pointer

​	经典two pointer问题，[3sum closest](https://leetcode.com/problems/3sum-closest) 与3sum思路基本相似，细节处理作区分。

### [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array):binary search

​	这是一道经典binary search题目，只要能满足每次缩减一半搜索空间，我们首先都应该考虑二分。本题需注意sorted array旋转状态可能在搜索过程中消失。

```java
public int search(int[] nums, int target) {
    if(nums.length == 0) return -1;
    
    int left = 0, right = nums.length - 1;
    while(left + 1 < right){
        int mid = (left + right) / 2;
        if(nums[mid] == target) return mid;
        else{
            if(nums[left] > nums[right]){
                if(nums[left] < nums[mid]){
                    if(target > nums[mid]) left = mid;
                    else{
                        if(target <= nums[nums.length - 1]) left = mid;
                        else right = mid ;
                    }
                }
                else{
                    if(target < nums[mid]) right = mid;
                    else{
                        if(target >= nums[0]) right = mid;
                        else left = mid;
                    }
                }    
            }
            else{
                if(target > nums[mid]) left = mid;
                else right = mid;
            }
        }
    }
    
    if(nums[left] == target) return left;
    if(nums[right] == target) return right;
    return -1;
}
```

### [Multiply Strings](https://leetcode.com/problems/multiply-strings):细节题

​	方法一：每次处理num1 * num2[x]，然后将结果相加到最终结果；

​	方法二：每次处理num1[y]* nums2[x]，就将结果相加到最终结果，这是计算carray需计算两层，一层是相乘，一层是相加。

```java
public String multiply(String num1, String num2) {
    if(num1.length() == 0 || num2.length() == 0) return "";
    if(num1.charAt(0) == '0' || num2.charAt(0) == '0') return "0";
    
    char[] Res = new char[num1.length() + num2.length() + 1];
    for(int i=0; i<Res.length; i++){
        Res[i] = '0';
    }
    
    for(int i=0; i<num2.length(); i++){       
        // 计算num1 * num2[x]
        int carray = 0, A = num2.charAt(num2.length() - 1 - i) - '0';
        for(int j=0; j<num1.length(); j++){
            int B = num1.charAt(num1.length() - 1 - j) - '0';
            
            int cur = ( A * B + carray )% 10;
            carray = ( A * B + carray ) / 10 ;
        
            carray += (Res[Res.length-1-i - j] - '0' + cur) / 10;
            Res[Res.length-1-i - j] =(char) ((Res[Res.length-1-i - j] - '0' + cur) % 10 + '0');
        }
        
        Res[Res.length - 1 - i - num1.length()] = (char)(carray + '0');     
    }
    
    int startIdx = 0 ;
    while(startIdx < Res.length && Res[startIdx] == '0') startIdx++;
    return new String(Res, startIdx, Res.length - startIdx);
}
```

### [Permutations](https://leetcode.com/problems/permutations):backtracking 

​	典型的backtracking

### [Spiral Matrix](https://leetcode.com/problems/spiral-matrix):细节题

​	这个题目虽然是细节题，但也有方法可循，下面两种写法对比：

方法一：

```java
public List<Integer> spiralOrder(int[][] matrix) {
    if(matrix.length == 0 || matrix[0].length == 0) return new ArrayList<Integer>();
    int w = matrix[0].length, h = matrix.length;
    int idx = 0;
    
    List<Integer> re = new ArrayList<Integer>();
    
    while(true){
        boolean f = false;
        
        for(int i = 0 + idx; i <= w - 1- idx; i++){
            if(idx <= h-1-idx){
                re.add(matrix[idx][i]);
                f = true;                 
            }

        }
        
        for(int i =1+ idx; i <= h - 1 - idx; i++){
            if(idx <= w - 1 - idx){
                re.add(matrix[i][w-1-idx]);
                f = true;
            }
        }
        
        for(int i = w - 2 -idx; i >= idx ; i--){
            if(h-1-idx > idx){
                re.add(matrix[h-1-idx][i]);
                f = true;                    
            }
        }
        
        for(int i = h - 2 - idx; i > idx; i --){
            if(idx < w - 1 - idx){
                re.add(matrix[i][idx]);
                f = true;                  
            }
        }
        
        if(f == false) break;
        idx ++;
    }
    //System.out.println("idx="+idx);
    return re;
}
```

方法二：

```java
public List<Integer> spiralOrder(int[][] matrix) {
    if(matrix.length == 0 || matrix[0].length == 0) return new ArrayList<Integer>();
    int w = matrix[0].length, h = matrix.length, t = w * h;
    int idx = 0;
    
    List<Integer> re = new ArrayList<Integer>();

    
    while(re.size() < t){
        
        for(int i = 0 + idx; i <= w - 1 - idx; i++){
            re.add(matrix[idx][i]);         
        }
        
        for(int i =1+ idx; (re.size() < t) && i <= h - 1 - idx; i++){
            re.add(matrix[i][w-1-idx]);
        }
        
        for(int i = w - 2 -idx; (re.size() < t) && i >= idx ; i--){
                re.add(matrix[h-1-idx][i]);                
        }
        
        for(int i = h - 2 - idx; (re.size() < t) && i > idx; i --){
                re.add(matrix[i][idx]);
        }
        
        idx ++;
    }
    //System.out.println("idx="+idx);
    return re;
}
```

​	使用方法一，会多碰到一个问题，即元素可能添加重复了，所以每个for循环都会多一个if条件判断；使用方式二无此问题？

### [Unique Paths](https://leetcode.com/problems/unique-paths):dp

### [Subsets](https://leetcode.com/problems/subsets):backtracking

​	这道题需注意List<List<Integer>>这种二维泛型用法。

```java
class Solution {
    void helper(List<List<Integer>> re, List<Integer> tl, int[] nums, int idx){
        re.add(new ArrayList<Integer>(tl));
        
        for(int i = idx; i<nums.length; i++){
            tl.add(nums[i]);
            helper(re, tl, nums, i+1);
            tl.remove(tl.size()-1);
        }
    }
    
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> re = new ArrayList<List<Integer>>();
        List<Integer> tl = new ArrayList<Integer>();
        helper(re, tl, nums, 0);
        return re;
    }
}
```



### [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array)：priorityqueue, quickselect

​	priorityQueue要会使用

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        if(nums.length < k) return -1;
        Queue<Integer> q = new PriorityQueue<>(
            new Comparator<Integer>(){
                public int compare(Integer a, Integer b){
                    return a - b;
                }
            });
        
        for(int i = 0; i < nums.length; i++){
            if(q.size() < k){
                q.add(nums[i]);
            }
            else{
                if(q.peek() < nums[i]){
                    q.remove();
                    q.add(nums[i]);
                }
            }
        }
        
        return q.peek();
    }
}
```

​	quickselect 也必须掌握的一种思想。

### [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst): binary search tree

​	这道题最容易想到的是前序遍历，

```java
class Solution {
    public int kthSmallest(TreeNode root, int k) {
        Stack<TreeNode> st = new Stack<TreeNode>();
        
        TreeNode p = root;
        int idx = 0;
        
        while(null != p || !st.empty()){
            while(null != p){
                st.push(p);
                p = p.left;
            }
                
            p = st.pop();
            if(idx + 1 == k) {
                return p.val;
            }
            
            idx ++;
            p = p.right;
        }
        
        return 0;
    }
}
```

[优秀写法](https://leetcode.com/problems/kth-smallest-element-in-a-bst/discuss/63660/3-ways-implemented-in-JAVA-(Python):-Binary-Search-in-order-iterative-and-recursive)

### [Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree) : binaery tree

​	递归方法写出来，效率极低

```java
class Solution {    
    boolean contain(TreeNode root, TreeNode p){
        if(null == root) return false;
        if(root.val == p.val) return true;
        
        return (contain(root.left, p) || contain(root.right, p));
    }
    
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        while(true){
            if(root.val == p.val || root.val == q.val) return root;
            
            boolean lp = contain(root.left, p);
            boolean rp = contain(root.right, p);
            boolean lq = contain(root.left, q);
            boolean rq = contain(root.right, q);            
            
            if(lp && rq || rp && lq) return root;
            else if(lp && lq) root = root.left;
            else root = root.right;
        }
    }
}
```

