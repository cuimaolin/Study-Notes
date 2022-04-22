

### 股票买卖问题

数组定义

```python
dp[i][k][0 or 1]
0 <= i <= n-1, 1 <= k <= K
# n 为天数，大K为最多交易数
# 此问题共n×K×2种状态，全部穷举就能搞定

for 0 <= i < n:
	for 1 <= k <= K:
		for s in {0, 1}:
			dp[i][k][s] = max(buy, sell, rest)
            
# dp[3][2][1]的含义就是：今天是第三天，我现在手上持有着股票，至今最多进行2次交易
```

状态转移方程：

```
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1]+prices[i])
# 解释： 今天我没有持有股票，有两种可能：
# 要么是我昨天就没有持有，然后今天选择rest，所以我今天还是没有持有；
# 要么是我昨天持有股票，但是今天我sell了，所以我今天没有持有股票；

dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
# 解释： 今天我持有着股票，有两种可能：
# 要么我昨天就持有着股票，然后今天选择rest，所以我今天还持有着股票；
# 要么我昨天本没有持有，但今天我选择buy，所以今天我就持有股票了
```

bad case：

```python
dp[-1][k][0] = 0
# 解释：因为i是从0开始的，
# 所以i=-1意味着还没有开始，这时候利润当时是0

dp[-1][k][1] = -infinity
# 解释：还没开始的时候，是不可能持有股票的
# 用负无穷表示这种不可能

dp[i][0][0] = 0
# 解释：因为k是从1开始的，
# 所以k=0意味着根本不允许交易，这时候利润当然是0

dp[i][0][1] = -infinity
# 解释：不允许交易的情况下，是不可能持有股票的
# 用负无穷表示这种不可能
```

#### [121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;
        int[][] dp = new int[n][2];
        for (int i = 0; i < n; i ++){
            if (i == 0){
                dp[i][0] = 0;
                dp[i][1] = -prices[i];
                continue;
            }
            int tmp = dp[i][0];
            dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1] + prices[i]);
            dp[i][1] = Math.max(dp[i-1][1], tmp - prices[i]);
        }
        return dp[n-1][0];
    }
}
```

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;
        int dp_i_0 = Math.max(0, -Integer.MAX_VALUE + prices[0]);
        int dp_i_1 = Math.max(-Integer.MAX_VALUE, -prices[0]);
        for (int i = 1; i < n; i ++){
            dp_i_0 = Math.max(dp_i_0, dp_i_1 + prices[i]);
            dp_i_1 = Math.max(- prices[i], dp_i_1);
        }
        return dp_i_0;
    }
}
```

#### [122. 买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

```java
class Solution {
    public int maxProfit(int[] prices) {
        int dp_i_0 = 0, dp_i_1 = Integer.MIN_VALUE;
        for (int i = 0; i < prices.length; i ++){
            int tmp = dp_i_0;
            dp_i_0 = Math.max(dp_i_0, dp_i_1 + prices[i]);
            dp_i_1 = Math.max(tmp - prices[i], dp_i_1);
        }
        return dp_i_0;
    }
}
```

#### [309. 最佳买卖股票时机含冷冻期](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

```java
class Solution {
    public int maxProfit(int[] prices) {
        int dp_i_0 = 0, dp_i_1 = Integer.MIN_VALUE;
        int dp_pre = 0; // 代表 dp[i-2][0]
        for (int i = 0; i < prices.length; i++){
            int tmp = dp_i_0;
            dp_i_0 = Math.max(dp_i_0, dp_i_1 + prices[i]);
            dp_i_1 = Math.max(dp_i_1, dp_pre - prices[i]);
            dp_pre = tmp;
        }
        return dp_i_0;
    }
}
```

#### [714. 买卖股票的最佳时机含手续费](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

```java
class Solution {
    public int maxProfit(int[] prices, int fee) {
        int dp_i_0 = 0, dp_i_1 = Integer.MIN_VALUE;
        for (int i = 0; i < prices.length; i ++){
            int tmp = dp_i_0;
            dp_i_0 = Math.max(dp_i_0, dp_i_1 + prices[i]);
            dp_i_1 = Math.max(dp_i_1, tmp - prices[i] - fee);
        }
        return dp_i_0;
    }
}
```

#### [123. 买卖股票的最佳时机 III](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)

```java
class Solution {
    public int maxProfit(int[] prices) {
        int dp_i_2_0 = 0, dp_i_2_1 = Integer.MIN_VALUE;
        int dp_i_1_0 = 0, dp_i_1_1 = Integer.MIN_VALUE;
        for (int i = 0; i < prices.length; i ++){
            dp_i_2_0 = Math.max(dp_i_2_0, dp_i_2_1 + prices[i]);
            dp_i_2_1 = Math.max(dp_i_2_1, dp_i_1_0 - prices[i]);
            dp_i_1_0 = Math.max(dp_i_1_0, dp_i_1_1 + prices[i]);
            dp_i_1_1 = Math.max(dp_i_1_1, - prices[i]);
        }
        return dp_i_2_0;
    }
}
```

#### [188. 买卖股票的最佳时机 IV](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/)

```java
class Solution {
    public int maxProfit(int max_k, int[] prices) {
        int n = prices.length;
        if (max_k > n / 2){
            return maxProfit_inf(prices);
        }
        int[][][] dp = new int[n][max_k+1][2];
        for (int i = 0; i < n; i ++){
            for (int k = max_k; k >= 1; k --){
                if (i == 0){
                    dp[i][k][0] = 0;
                    dp[i][k][1] = -prices[i];
                    continue;
                }
                dp[i][k][0] = Math.max(dp[i-1][k][0], dp[i-1][k][1] + prices[i]);
                dp[i][k][1] = Math.max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i]);
            }
        }
        return dp[n-1][max_k][0];
    }
    public int maxProfit_inf(int[] prices) {
        int dp_i_0 = 0, dp_i_1 = Integer.MIN_VALUE;
        for (int i = 0; i < prices.length; i ++){
            int tmp = dp_i_0;
            dp_i_0 = Math.max(dp_i_0, dp_i_1 + prices[i]);
            dp_i_1 = Math.max(tmp - prices[i], dp_i_1);
        }
        return dp_i_0;
    }
}
```

