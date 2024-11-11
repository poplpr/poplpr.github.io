---
layout: post
title: Atcoder Beginner Contest 324 F Beautiful Path 题解-分数规划
categories: [AtCoder, abc, xcpc, 分数规划, 算法]
description: Atcoder Beginner Contest 324 F Beautiful Path 题解-分数规划
keywords: abc, xcpc, 分数规划, 算法
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

为了更好的阅读体验，请点击[这里](https://www.cnblogs.com/bringlu/p/17764921.html)

分数规划小技巧：**尽可能将式子写成存在某种取值，使得不等式成立的形式。** 不然可能需要绕几个弯才能想出来。

[题目链接](https://atcoder.jp/contests/abc324/tasks/abc324_f)

> 题目大意：给出一个 DAG，每条边有一个 $b_i, c_i$，保证从编号小的边向编号大的边连边，且 $1$ 到 $n$ 必有路径，求 $1$ 到 $n$ 路径上的 $\max \frac{\sum b}{\sum c}$。

分数规划常规做法：二分答案 $x$，下面比较一下两种设法：

1. $x > \max \frac{\sum b}{\sum c} \iff$ 从 $1$ 到 $n$ 的所有路径都满足 $x > \frac{\sum b}{\sum c}$ 这一条件 $\iff$ 从 $1$ 到 $n$ 的所有路径都满足 $\sum (xc - b) > 0$ 这一条件（这里必须是大于号接下来才是最短路，不然是最长路）$\iff$ 从 $1$ 到 $n$ 点的最短路 $d_n > 0$。
   - 满足这个条件证明 $x$ 过大，使 $R \leftarrow M$
   - 否则 $L \leftarrow M$
2. $x < \max \frac{\sum b}{\sum c} \iff$ 从 $1$ 到 $n$ 的存在某条路径满足 $x < \frac{\sum b}{\sum c}$ 这一条件 $\iff$ 从 $1$ 到 $n$ 的存在某条路径满足 $\sum (xc - b) < 0$ 这一条件（这里如果是小于号就是天然的求最短路，如果是大于号你还需要取负修改成小于号）$\iff$ 从 $1$ 到 $n$ 点的最短路 $d_n < 0$。
   - 满足这个条件证明 $x$ 过小，使 $L \leftarrow M$
   - 否则 $R \leftarrow M$

设法 2 可以让你的脑子少转个圈，这样转化为存在某种方案满足这个条件，其实也是方便地用最大/最小值来验证不等式是否成立。

赛时用的设法 1，脑袋被转得晕乎乎的。

再推一点东西吧。在满足设法 2 的前提下，有如下两个快速结论：

1. 最大化 $\frac{\sum b}{\sum c} \Longrightarrow$ 设 $x > \max \frac{\sum b}{\sum c} \iff \max \sum (b-cx) > 0$ 
   - 存在最大值 $\sum (b-cx) > 0 \Longrightarrow L \gets M$
   - 否则 $R \gets M$
2. 最小化 $\frac{\sum b}{\sum c} \Longrightarrow$ 设 $x < \min \frac{\sum b}{\sum c} \iff \min \sum (b-cx) < 0$ 
   - 存在最小值 $\sum (b-cx) < 0 \Longrightarrow R \gets M$
   - 否则 $L \gets M$

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
typedef long double db;

#define IL inline
#define fi first
#define se second
#define mk make_pair
#define pb push_back
#define SZ(x) (int)(x).size()
#define ALL(x) (x).begin(), (x).end()
#define dbg1(x) cout << #x << " = " << x << ", "
#define dbg2(x) cout << #x << " = " << x << endl

template <typename T>
void _debug(const char* format, T t) {
    cerr << format << '=' << t << endl;
}

template <class First, class... Rest>
void _debug(const char* format, First first, Rest... rest) {
    while (*format != ',') cerr << *format++;
    cerr << '=' << first << ',';
    _debug(format + 1, rest...);
}

template <typename T>
ostream& operator<<(ostream& os, const vector<T>& V) {
    os << "[ ";
    for (const auto& vv : V) os << vv << ", ";
    os << ']';
    return os;
}
#ifdef LOCAL
    #define dbg(...) _debug(#__VA_ARGS__, __VA_ARGS__)
#else
    #define dbg(...) 
#endif

template<typename Tp> IL void read(Tp &x) {
    x=0; int f=1; char ch=getchar();
    while(!isdigit(ch)) {if(ch == '-') f=-1; ch=getchar();}
    while(isdigit(ch)) { x=x*10+ch-'0'; ch=getchar();}
    x *= f;
}
template<typename First, typename... Rest> IL void read(First &first, Rest&... rest) {
    read(first); read(rest...);
}
int buf[42];
template<typename Tp> IL void write(Tp x) {
    int p = 0;
    if(x < 0) { putchar('-'); x=-x;}
    if(x == 0) { putchar('0'); return;}
    while(x) {
        buf[++p] = x % 10;
        x /= 10;
    }
    for(int i=p;i;i--) putchar('0' + buf[i]);
}
template<typename First, typename... Rest> IL void write(const First& first, const Rest&... rest) {
    write(first); putchar(32); write(rest...);
}

const int N = 200000 + 5;

struct E {
    int u, v, b, c;
    db d;
    E(int u = 0, int v = 0, int b = 0, int c = 0, db d = 0.0):u(u), v(v), b(b), c(c), d(d) {}
};

const db inf = 1e18;

int n, m;
db d[N];
vector<E> G[N];

bool check(db x) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < G[i].size(); j++) {
            E e = G[i][j];
            G[i][j].d = e.c * x - e.b;
        }
    }
    fill(d, d + n, inf);
    d[0] = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < SZ(G[i]); j++) {
            E& e = G[i][j];
            d[e.v] = min(d[e.v], d[i] + e.d);
        }
    }
    return d[n-1] > 0;
}

void solve() {
    read(n, m);
    for (int i = 0; i < m; i++) {
        int u, v, b, c; read(u, v, b, c); u--; v--;
        G[u].pb(E(u, v, b, c, 0.0));
    }
    db L = 0.0, R = 10000.0;
    while (R - L > 1e-14) {
        db M = (L + R) / 2.0;
        if (check(M)) {R = M;}
        else L = M;
    }
    printf("%.16Lf\n", L);
}

int main() {
#ifdef LOCAL
    freopen("test.in", "r", stdin);
    // freopen("test.out", "w", stdout);
#endif
    int T = 1;
    // read(T);
    while(T--) solve();
    return 0;
}
```