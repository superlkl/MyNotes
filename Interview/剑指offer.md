# 509.斐波那契数列

- 写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项。斐波那契数列的定义如下：

```java
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1
```

- 斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出

- 答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1

- 示例 1：

```java
输入：n = 2
输出：1
```

- 示例 2：

```java
输入：n = 5
输出：5
```

- 提示：0 <= n <= 100
- 思路

![image-20200608190155989](图片.assets/image-20200608190155989.png)

>- dp[n]返回斐波那契数列的第n个数
>- dp[0]=0，dp[1]=1，dp[n]=dp[n-1]+dp[n-2]

- 题解：使用动态规划

```java
public class Solution509 {
    public int fib(int n) {
        if (n < 2) return n;
        int[] dp = new int[n+1]; //给n+1个空间
        dp[1] = 1;
        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i-1] + dp[i-2];
            System.out.println(dp[i]);
            dp[i] %= 1000000007; //题目要求取模,避免数太大超过了int型的存储范围
            System.out.println(dp[i]);
        }
        return dp[n];
    }
}
```