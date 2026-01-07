---
title: 算法模板：动态规划
date: 2026-01-05
tags: [算法, 模板, DP]
---

# 动态规划模板

这里记录了一些常用的 DP 模板。

```cpp
for (int i = 1; i <= n; i++) {
    for (int j = m; j >= w[i]; j--) {
        dp[j] = max(dp[j], dp[j - w[i]] + v[i]);
    }
}
```

## 背包问题
- 0/1 背包
- 完全背包
- 多重背包

