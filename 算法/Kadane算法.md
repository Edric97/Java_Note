## Kadane算法

这个算法，用一个例子来看：

——给定一个整数数组 `nums` ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

——例如：

```markdown
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
```



我的解释：

```markdown
		-2, 1, -3, 4, -1, 2, 1, -5, 4
sum -2, 1, -3, 4,  3, 5, 6,  1, 5
res -2, 1,  1, 4,  4, 5, 6,  6, 6
```

代码实现：

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int res = nums[0];
        int sum = 0;
        for (int num : nums) {
            if (sum > 0)
                sum += num;
            else
                sum = num;
            res = Math.max(res, sum);
        }
        return res;
    }
}
```



这道题目叫做“最大子序和”，是动态规划的经典题目。这个算法，也是动态规划的经典算法。

问题在于，数组中新的遍历到元素是否可以加入到之前状态的dp数组中，Kadane算法说明，当前sum > 0的时候才可以加入前面的状态，否则就树立一个新状态，因为要是sum <= 0，说明前面的状态对当前元素是没有增益的。

类似的题目可以在： https://leetcode-cn.com/leetbook/read/dynamic-programming-1-plus/5og59t/ 中找到。
