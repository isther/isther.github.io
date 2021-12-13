---
title: "Educational Codeforces Round 118 (Rated for Div. 2)"
date: 2021-12-02T13:33:51+08:00
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

**2021.12.01 22:35  --  2021.12.02  0.30**

<!--more-->

### [A. Long Comparison](https://codeforces.com/contest/1613/problem/A)

#### 题意

给出两个数字，这两个数字都遵循特定的格式，在$x$末尾追加$p$个`0`，比较这两个数的大小。

#### 思路

固定`x`的长度，比较`p`即可。

#### 代码

```c++
#include <bits/stdc++.h>
using namespace std;
#define INF 0x3f3f3f3f
#define int long long
#define maxn 1e5 + 5

int x[3];
int p[3];
int read() {
    int x;
    cin >> x;
    return x;
}

void read(int i) {
    x[i] = read(), p[i] = read();

    while (x[i] < 1e6) {
        x[i] *= 10;
        p[i]--;
    }
}

void solve() {
    read(1);
    read(2);

    if (p[1] < p[2]) {
        cout << "<" << endl;
    } else if (p[1] > p[2]) {
        cout << ">" << endl;
    } else {
        if (x[1] < x[2])
            cout << "<" << endl;
        else if (x[1] > x[2])
            cout << ">" << endl;
        else
            cout << "=" << endl;
    }
}

main() {
    int t = read();
    while (t--) {
        solve();
    }
    return 0;
}

```



### [B. Absent Remainder](https://codeforces.com/contest/1613/problem/B)

#### 题意

给出数组元素数量和一个数组，输出`[n/2]`组`a[i],a[j]`，需满足：

* $a[i] \ne a[j]$
* $a_i\mod a_j \ne a_i(i = 1...n)$

#### 思路

排序之后找到最小元素，其余元素对其取模不可能等于数组中其他元素。

#### 代码

```c++
#include <bits/stdc++.h>
using namespace std;
#define INF 0x3f3f3f3f
#define int long long
#define maxn 1e5 + 5

int read() {
    int x;
    cin >> x;
    return x;
}

void solve() {
    int n = read();

    vector<int> v;
    for (int i = 0; i < n; i++)
        v.push_back(read());

    sort(v.begin(), v.end());
    int cnt = 0;
    for (int i = 1; i < v.size(); i++) {
        cout << v[i] << " " << v[0] << endl;
        if (++cnt == n / 2)
            return;
    }
}

main() {
    int t = read();
    while (t--) {
        solve();
    }
    return 0;
}
```



### [C. Poisoned Dagger](https://codeforces.com/contest/1613/problem/C)

#### 题意

恶龙有`h`滴血，勇士丢出匕首攻击恶龙，匕首不会直接对恶龙造成伤害，但是有中毒效果，在接下来的`harm`秒内，恶龙每秒掉1滴血。中毒效果不会叠加。

例如：`k=4`，勇士在2，4，10秒时攻击恶龙(10滴血)。第`2`秒，第`3`秒各掉一滴血，到了第`4`秒，效果刷新，每秒掉一滴血到第`7`秒

，第`8`秒，第`9`秒不掉血，从第`10`秒开始掉血，一直到第 `13`秒，恶龙死亡。

给出勇士攻击次数以及攻击的时间点，恶龙的血量，求最小的`harm`。

#### 思路

二分。

#### 代码

```c++
#include <bits/stdc++.h>
using namespace std;
#define INF 0x3f3f3f3f
#define int long long
#define maxn 1e5 + 5

int n;
int h;
vector<int> v;
int read() {
    int x;
    cin >> x;
    return x;
}

bool judge(int mid) {
    int sum = mid;

    for (int i = 0; i < v.size(); i++) {
        sum += min(v[i], mid);
    }
    return sum >= h;
}

void solve() {
    n = read(), h = read();

    v.clear();
    vector<int> w;
    for (int i = 0; i < n; i++) {
        w.push_back(read());
        if (i)
            v.push_back(w[i] - w[i - 1]);
    }

    int l = 1, r = 1e18;
    while (l + 1 < r) {
        int mid = (l + r) >> 1;
        if (judge(mid))
            r = mid;
        else
            l = mid;
    }
    if (judge(l))
        cout << l << endl;
    else
        cout << r << endl;
}

main() {
    int t;
    cin >> t;
    while (t--) {
        solve();
    }
    return 0;
}
```

