【腾讯50题】

[Reverse integer](https://leetcode-cn.com/problems/reverse-integer/) : 目前的办法是通过字符串比较来判断是否溢出，更高效的解法是边转换边判断。整体容易溢出不好控制，分布来判断就很安全。

[Palindrome Number](https://leetcode-cn.com/classic/problems/reverse-integer/description/) :  最直接的方法将数字取模之后放入vector，双指针进行比较。但这也是低效的办法。

[Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/description/) : 要尝试一些其他方法，看看效率，如二分法

[remove dumplicates from sorted array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/) : 这道题自己用朴素的方法作的，会出一些边界问题，漏洞多。但是如果改用推荐的solution的two pointer来作，就不会出问题。还是对two pointer理解的不够。
