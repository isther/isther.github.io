---
title: "Codeforces Round #759 (Div. 2)"
date: 2021-12-13T12:50:51+08:00
categories: ["Algorithm"]
tags: ["CPP", "C", "Algorithm", "Codeforces", "Div.2"]
# description: "Desc Text."

# weight: 1

TocOpen: false
draft: false

hideSummary: false
hidemeta: false
disableShare: false

comments: true
canonicalURL: "https://www.niuwx.cn/"
---

做题笔记。

**2021.12.12 23:15  --  2021.12.13  01:15**

<!--more-->

### [A. Life of a Flower](https://codeforces.com/contest/1585/problem/A)

#### 题意

给花浇水，纯模拟，有以下要求：

- 连续两天不浇水，花会死。
- 如果浇水，花长高`1cm`。
- 连续两天浇水，花长高`5cm`而非`1cm`。

#### 思路

纯模拟，依次处理每天的浇水情况就可以。

#### 代码

```c++
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define lson 2 * i
#define rson 2 * i + 1
#define LS l, mid, lson
#define RS mid + 1, r, rson
#define UP(i, x, y) for (int i = x; i < y; i++)
#define DOWN(i, x, y) for (int i = x; i > y; i--)
#define MS(a, x) memset(a, x, sizeof(a))
#define GCD(x, y) __gcd(x, y)
#define INF 0x3f3f3f3f
#define EPS 1e-8
#define MOD 1e9 + 7
const int N = 1e5 + 5;

int read() {
    int x;
    cin >> x;
    return x;
}
/*********Solve**********/
vector<int> v;
void solve() {
    int n = read();

    v.clear();
    UP(i, 0, n) {
        v.push_back(read());
    }

    int sum = 1 + v[0];
    UP(i, 1, n) {
        if (!v[i] && !v[i - 1]) {
            printf("-1\n");
            return;
        }

        if (v[i]) {
            if (v[i - 1]) {
                sum += 5;
            } else {
                sum += 1;
            }
        }
    }
    cout << sum << endl;
}

main() {
    // ios::sync_with_stdio(0);
    int t = read();
    // int t = 1;

    while (t--) {
        solve();
    }
    return 0;
}
```

### [B. Array Eversion](https://codeforces.com/contest/1585/problem/B)

#### 题意

给出数组`a`，进行以下操作：

令$x=a_n$，进行变换，使得`x`左侧的所有元素都小于`x`，右侧的所有元素都大于`x`，并且不能改变元素的相对位置。

进行`k`次操作之后，数组将不会改变。

求最小的`k`。

#### 思路

先以这个样例为例进行分析`(2,5,1,4,3)`，变换一次之后`(2,1,3,5,4)`，再进行一次变换`(2,1,3,4,5)`，此时无需再次进行变换。可以发现，如果最后一位并不是最大值时，则可以继续进行变换。

既然每次都要求以数组的最后一个数为基准，那可以从后往前遍历，不断寻找符合条件的`x`，如果`a[i]>x`，则说明又可以进行一次变换。

#### 代码

```c++
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define lson 2 * i
#define rson 2 * i + 1
#define LS l, mid, lson
#define RS mid + 1, r, rson
#define UP(i, x, y) for (int i = x; i < y; i++)
#define DOWN(i, x, y) for (int i = x; i > y; i--)
#define MS(a, x) memset(a, x, sizeof(a))
#define GCD(x, y) __gcd(x, y)
#define INF 0x3f3f3f3f
#define EPS 1e-8
#define MOD 1e9 + 7
const int N = 1e5 + 5;

int read() {
    int x;
    cin >> x;
    return x;
}
/*********Solve**********/
vector<int> v;
void solve() {
    int n = read();

    v.clear();
    UP(i, 0, n) {
        v.push_back(read());
    }

    int ans = 0;
    int x = v[n - 1];
    DOWN(i, n - 1, -1) {
        if (v[i] > x) {
            ans++;
            x = v[i];
        }
    }
    cout << ans << endl;
}

main() {
    // ios::sync_with_stdio(0);
    int t = read();
    // int t = 1;

    while (t--) {
        solve();
    }
    return 0;
}
```

### [C. Minimize Distance](https://codeforces.com/contest/1585/problem/C)

#### 题意

在一条坐标轴上搬运货物，n为需要送货的地点，k为一次可以携带的货物数量，每次都是从原点出发，送完最后一次货物无需回到原点，求最小路程。

#### 思路

由于给出的坐标有正有负，都是从原点出发，所以可以将正负两边拆成两次来考虑，求出总和之后，再将正负两边最大的一个坐标减去。

模拟貌似有点问题，上DP。

考虑`x>0`的情况，将坐标升序排序，设$dp_i$为送完前`i`个点的最小路程，则有转移方程$dp_i=dp_{i-k}+2*x_i$。

#### 代码

```c++
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define lson 2 * i
#define rson 2 * i + 1
#define LS l, mid, lson
#define RS mid + 1, r, rson
#define UP(i, x, y) for (int i = x; i < y; i++)
#define DOWN(i, x, y) for (int i = x; i > y; i--)
#define MS(a, x) memset(a, x, sizeof(a))
#define GCD(x, y) __gcd(x, y)
#define INF 0x3f3f3f3f
#define EPS 1e-8
#define MOD 1e9 + 7
const int N = 1e5 + 5;

int read() {
    int x;
    cin >> x;
    return x;
}
/*********Solve**********/
multiset<int> v1;
multiset<int> v2;
void solve() {
    int n = read(), k = read();
    v1.clear(), v2.clear();

    int m = 0;
    UP(i, 0, n) {
        int temp = read();
        if (temp >= 0)
            v1.insert(temp);
        else
            v2.insert(abs(temp));
        m = max(m, abs(temp));
    }
    int sum = 0;
    auto count = [&](multiset<int> v) {
        int sz = v.size();
        int dp[sz + 1];
        dp[0] = 0;

        UP(i, 1, sz + 1) {
            dp[i] = (i - k > 0 ? dp[i - k] : 0) + 2 * (*v.begin());
            v.erase(v.begin());
        }
        sum += dp[sz];
    };

    count(v1), count(v2);
    cout << sum - m << endl;
}

main() {
    // ios::sync_with_stdio(0);
    int t = read();
    // int t = 1;

    while (t--) {
        solve();
    }
    return 0;
}
```
