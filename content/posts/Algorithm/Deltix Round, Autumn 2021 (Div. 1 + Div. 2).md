---
title: "Deltix Round, Autumn 2021 (Div. 1 + Div. 2)"
date: 2021-11-29T13:33:51+08:00
categories: ["Algorithm"]
tags: ["CPP", "C", "Algorithm", "Codeforces", "Div.1", "Div.2"]
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

**2021.11.28 22:35  --  2021.11.29  1:00**

<!--more-->

第一次参加Codeforces的比赛，太菜了...

### [A. Divide and Multiply](https://codeforces.com/contest/1609/problem/A)

#### 题意

据说是500分的题，难度不是很大。大概题意是给一组数，每次操作可以将一个数（必须是偶数）除`2`，同时将另一个数乘`2`，求执行最佳操作之后数组和的最大值。

#### 思路

1. 找到数组元素`a[i]`因式分解后`2`的个数$t_i$，$a[i] /= 2*t_i$，处理之后得到了`2`的个数和`t`，且所有数组元素都为奇数。
2. 找到处理后的所有数组元素的最大值，乘$2^t$即可得到最大项，加上其余数组元素即可得到最大数组和。

#### 代码

```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int maxn = 25;

int n;
ll arr[maxn];

ll fast(ll a, ll base, ll power) {
    ll ans = a;
    while (power) {
        if (power & 1)
            ans *= base;
        base *= base;
        power >>= 1;
    }
    return ans;
}
int main(int argc, char const *argv[]) {
    int t;
    scanf("%d", &t);
    while (t--) {
        scanf("%d", &n);
        int max = 0;
        ll power = 0;
        ll ans = 0; //long long 防爆
        for (int i = 0; i < n; i++) {
            scanf("%d", &arr[i]);

            while (arr[i] % 2 == 0) {
                arr[i] /= 2;
                power++;
            }

            if (arr[i] > arr[max])
                max = i;
            ans += arr[i];
        }
        ans += fast(arr[max], 2, power) - arr[max];//猥琐一点加了快速幂防止超时
        cout << ans << endl;
    }
    return 0;
}
```

### [B. William the Vigilant](https://codeforces.com/contest/1609/problem/B)

#### 题意

给一个字符串`s`，只包含字符a、b、c，再给定一个数字`q`，接下来给出`q`组操作`(pos,c)`，每次操作将字符串`s`中第`pos`个字符变成给出的字符`c`，然后输出改变后的字符串还需要最少修改几个字符使得这个字符串没有子串`abc`。

#### 思路

这题暴力遍历会超时。可以提前统计字串数量，每一次操作分情况讨论`abc`字串的数量变化。

#### 代码
```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 1e5 + 5;

int n, q;
string s;
int pos;
char c;
int ans;

void init() {
    ans = 0;
    for (int i = 0; i < n; i++)
        if (i + 2 < n && s[i] == 'a' && s[i + 1] == 'b' && s[i + 2] == 'c')
            ans++;
}

void find() {
    pos = pos - 1;
    if (s[pos] == 'a') {
        if (s[pos + 1] == 'b' && s[pos + 2] == 'c')
            if (c != 'a')
                ans--;

        if (c == 'b' && s[pos - 1] == 'a' && s[pos + 1] == 'c')
            ans++;

        if (c == 'c' && s[pos - 2] == 'a' && s[pos - 1] == 'b')
            ans++;
    }

    if (s[pos] == 'b') {
        if (s[pos - 1] == 'a' && s[pos + 1] == 'c')
            if (c != 'b')
                ans--;

        if (c == 'a' && s[pos + 1] == 'b' && s[pos + 2] == 'c')
            ans++;

        if (c == 'c' && s[pos - 2] == 'a' && s[pos - 1] == 'b')
            ans++;
    }

    if (s[pos] == 'c') {
        if (s[pos - 2] == 'a' && s[pos - 1] == 'b')
            if (c != 'c')
                ans--;

        if (c == 'a' && s[pos + 1] == 'b' && s[pos + 2] == 'c')
            ans++;

        if (c == 'b' && s[pos - 1] == 'a' && s[pos + 1] == 'c')
            ans++;
    }

    s[pos] = c;
    cout << ans << endl;
}

void solve() {
    scanf("%d %c", &pos, &c);

    find();
}

int main() {
    scanf("%d%d", &n, &q);
    cin >> s;

    init();
    while (q--) {
        solve();
    }
    return 0;
}

```

### [C. Complex Market Analysis](https://codeforces.com/contest/1609/problem/C)

#### 题意

题目比较好理解，给定`n`, `e`和一个含`n`个元素的数组。

`(i,k)`需要满足以下条件：

1. $1<=i,k$
2. $i+e*k<=n$
3. $a_i \cdot a_{i+e} \cdot a_{i+2e} \cdot... \cdot a_{i+ke}$ 是一个质数。

求出满足条件的`(i,k)`的对数。

#### 思路

考虑到第三个条件的限制，需要满足多项式的每一项`有且只有`一个质数，其他项都为`1`。刚开始做就是暴力遍历，但是超时，需要继续剪枝。考虑到有且只有一个质数，那么可以从质数下手，去找满足条件的`1`，这样消耗就会小很多。

#### 代码

```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int maxn = 1e6 + 5;

int n, e;
int arr[maxn];
int prime[maxn];

void isprime() {
    for (int i = 1; i < maxn; i++)
        prime[i] = 1;
    prime[1] = 0;

    for (int i = 2; i < maxn; i++) {
        if (prime[i] == 1)
            for (int j = 2; j * i < maxn; j++) {
                prime[i * j] = 0;
            }
    }
}

void solve() {
    ll cnt = 0;
    for (int i = 1; i <= n; i++) {
        //******************************
        if (prime[arr[i]] == 1) { //找到质数
            ll l = 0, r = 0;
            for (int t = i + e; t <= n; t += e) {//向右找1
                if (arr[t] == 1)
                    r++;
                else
                    break;
            }

            for (int t = i - e; t >= 1; t -= e) {//向左找1
                if (arr[t] == 1)
                    l++;
                else
                    break;
            }

            cnt += l + r + l * r; //统计答案
        }
        //******************************
    }
    cout << cnt << endl;
}

int main() {
    isprime();//数据范围1e6，质数筛打表节约判断时间
    int t;
    scanf("%d", &t);
    while (t--) {
        scanf("%d%d", &n, &e);
        for (int i = 1; i <= n; i++) {
            scanf("%d", &arr[i]);
        }
        solve();
    }
    return 0;
}
```

统计答案这一步，这个质数左边有`l`个`1`，右边有`r`个`1`。

1. 只考虑左边及质数，共`l`种情况。
2. 只考虑质数及右边，共`r`种情况。
3. 考虑左边+质数+右边，共`l*r`种情况。

### [D. Social Network](https://codeforces.com/contest/1609/problem/D)

#### 题意

共`n`个人，`d`个关系对`(a,b)`，每个关系对可以介绍`a`和`b`两个人认识， 每次介绍之后输出熟人最多的数量。

题目有个坑点，在分析第二个样例的时候可以发现第四次操作，`1`、`4`连接之后，答案明明应该是`3`，但是正确答案却是`4`。

题目中还有一句话需要注意：

![social_network](/social_network.png)

即如果`1`、`4`已经认识，那么就把`1`或`4`介绍给另一个大连通块认识。

#### 思路

并查集，先检查是否在同一连通块，如果在同一连通块，那就与另一个大连通块连通。

#### 代码

```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int maxn = 1005;

int n, d;
int parent[maxn];
int temp[maxn];
int sz[maxn];

void init() {
    for (int i = 1; i <= n; i++) {
        parent[i] = i;
        sz[i] = 1;
    }
}

int find(int x) {
    return x == parent[x] ? x : parent[x] = find(parent[x]);
}

bool merge(int x, int y) {
    x = find(x);
    y = find(y);

    if (x != y) {
        parent[x] = y;
        sz[y] += sz[x];
        return true;
    }
    return false;
}

int main(int argc, char const *argv[]) {
    scanf("%d%d", &n, &d);
    init();

    int cnt = 1;//大连通块数量
    for (int i = 0, x, y; i < d; i++) {
        scanf("%d%d", &x, &y);
        if (!merge(x, y))
            cnt++;

        vector<int> temp;
        for (int i = 1; i <= n; i++)
            if (parent[i] == i)
                temp.push_back(sz[i]);

        sort(temp.rbegin(), temp.rend());
        cout << accumulate(temp.begin(), temp.begin() + cnt, 0) - 1 << endl;
    }
    return 0;
}
```

