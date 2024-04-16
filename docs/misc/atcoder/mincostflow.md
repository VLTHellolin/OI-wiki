用于解决 [最小费用最大流问题](../../graph/flow/min-cost.md)。

## 构造函数

``` cpp
mcf_graph<Cap, Cost>(int n);
```

构造一个 $n$ 点 $0$ 边的图，`Cap` 和 `Cost` 是边容量限制和费用的数据类型。

**约束条件**

- $0\le n\le 10^8$。
- `Cap` 和 `Cost` 为 `int` 或者 `ll`。

**复杂度**

- $O(n)$。

## 成员函数

### add_edge

``` cpp
int add_edge(int from, int to, Cap cap, Cost cost);
```

建一条 `from` 到 `to` 的边，容量限制 `cap`，流量 $0$，费用 `cost`。返回边的编号。

**约束条件**

- $0\le \mathrm{from}, \mathrm{to} < n$。
- $0\le \mathrm{cap}, \mathrm{cost}$。

**复杂度**

- 均摊 $O(1)$。

### flow

``` cpp
pair<Cap, Cost> flow(int s, int t);                 // (1)
pair<Cap, Cost> flow(int s, int t, Cap flow_limit); // (2)
```

1. 计算 $s$ 到 $t$ 的最小费用最大流。**不能**运行多次。
2. 计算 $s$ 到 $t$ 在 `flow_limit` 流量限制下的最小费用最大流。**不能**运行多次。

**约束条件**

- $0\le s, t < n$。
- $s\not = t$。
- **不能**调用多次函数。
- 流量答案在 `Cap` 范围内。
- 费用答案在 `Cost` 范围内。
    - 记 $a$ 为所有边中最大费用。
    - 对于 `int`，$0\le na\le 2\times 10^9+1000$。
    - 对于 `ll`，$0\le na\le 8\times 10^{18}+1000$。

**复杂度**

- $O(F(n+m)\log(n+m))$，$F$ 是流量。

### slope

``` cpp
vector<pair<Cap, Cost>> slope(int s, int t);                 // (1)
vector<pair<Cap, Cost>> slope(int s, int t, Cap flow_limit); // (2)
```

计算流量与最小成本之间的变化曲线。具体地，定义 $g(x)$ 是当流量恰好是 $x$ 时的 $s-t$ 流量最小费用，则返回 $g(x)$ 在 $x\in[0, \mathrm{maxflow}]$ 时的点集。满足以下性质：

- 第一个点是 $(0, 0)$。
- 对于 (1)，最后一个点是 $(\mathrm{maxflow}, g(\mathrm{maxflow}))$。
- 对于 (2)，记 $a=\min(\mathrm{maxflow}, \mathrm{flow_limit})$，则最后一个点是 $(a, g(a))$。
- `.first` 与 `.second` 均严格递增。
- 没有任何三个点在同一条直线上。

**约束条件**

同上。

**复杂度**

同上。

### get_edge 与 edges

``` cpp
struct mcf_graph<Cap, Cost>::edge {
    int from, to;
    Cap cap, flow;
    Cost cost;
};

mcf_graph<Cap, Cost>::edge get_edge(int i); // (1)
vector<mcf_graph<Cap, Cost>::edge> edges(); // (2)
```

1. 返回编号为 $i$ 的边。
2. 返回所有边的列表。

**约束条件**

1. $0\le i < m$。

**复杂度**

1. $O(1)$。
2. $O(m)$。

## 示例

尝试使用 AtCoder Library 通过 [MinCostFlow](https://atcoder.jp/contests/practice2/tasks/practice2_e)。

??? 代码

    ``` cpp
    #include <atcoder/mincostflow>
    #include <iostream>

    using namespace std;
    using namespace atcoder;

    const long long BIG = 1'000'000'000;

    int main() {
        int n, k;
        cin >> n >> k;

        /**
        * generate (s -> row -> column -> t) graph
        * i-th row correspond to vertex i
        * i-th col correspond to vertex n + i
        **/
        mcf_graph<int, long long> g(2 * n + 2);
        int s = 2 * n, t = 2 * n + 1;

        // we can "waste" the flow
        g.add_edge(s, t, n * k, BIG);

        for (int i = 0; i < n; i++) {
            g.add_edge(s, i, k, 0);
            g.add_edge(n + i, t, k, 0);
        }

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                long long a;
                cin >> a;
                g.add_edge(i, n + j, 1, BIG - a);
            }
        }

        auto result = g.flow(s, t, n * k);
        cout << 1LL * n * k * BIG - result.second << endl;

        vector<string> grid(n, string(n, '.'));
        auto edges = g.edges();
        for (auto e : edges) {
            if (e.from == s || e.to == t || e.flow == 0) continue;

            grid[e.from][e.to - n] = 'X';
        }

        for (int i = 0; i < n; i++) {
            cout << grid[i] << endl;
        }
        return 0;
    }
    ```