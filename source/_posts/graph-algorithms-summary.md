---
title: 图论算法总结（搜索、拓扑排序、最短路、最小生成树、二分图）
date: 2021-12-26 09:36:51
tags:
- Notes
- Algorithm
- Data Structrue
- Graph theory
categories: 数据结构与算法
---

## 前言

最近在 [AcWing](https://www.acwing.com/activity/content/11/) 学习图论的相关算法，包含了图的存储、搜索、拓扑排序、最短路算法、最小生成树算法、二分图算法等知识。因为涉及到的内容与代码较多，所以将这些内容汇总成笔记，以便日后回顾。
<!--more-->

## 树的存储和图的存储

- **邻接矩阵**

  空间开销为点的数目的平方，当图较为稠密时使用，稀疏图的话会有很多矩阵中的元素没有值，浪费掉了。存储方式使用一个二维矩阵即可。

  ```cpp
  int N;
  int graph[N][N];
  ```

- **邻接表**

  空间开销为边的数目，当图较为稀疏的时候使用，存储方式为一个多层链表，即每个顶点都有一个专属的链表，每个链表中存着与当前顶点有边的其他顶点。初始化的代码如下所示：

  ```cpp
  int N, M;
  // 下面几个变量存储的分别为各个顶点的链表头，当前边的终点，当前边的权值，下一条边的编号，和当前边的编号 
  int h[N], e[M], w[M], ne[M], idx = 0;
  memset(h, -1, sizeof(h));
  void add(int a, int b, int c) {
  		// 更新当前边的终点，当前边的权值，当前边下一条边的序号和头插法更新当前顶点连接的第一条边
  		e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
  }
  ```



## 树与图的搜索

- **深度优先搜索**

  ```cpp
  int dfs(int u) {
  		st[u] = true;
  
  		for (int i = h[u]; i != -1; i = ne[i]) {
  				int j = e[i];
  				if (!st[j]) dfs(j);
  		}
  }
  ```

  [例题：树的重心](https://www.acwing.com/activity/content/problem/content/909/)

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  const int N = 1e5 + 5, M = 2 * N;
  int ans = N;
  int n;
  bool st[N];
  int h[N], e[M], ne[M], idx = 0;
  
  void add(int a, int b) {
      e[idx] = b, ne[idx] = h[a], h[a] = idx++;
  }
  
  int dfs(int u) {
      st[u] = true;
      int res = 0, sum = 1;
      for (int i = h[u]; i != -1; i = ne[i]) {
          int j = e[i];
          if (st[j]) continue;
          int s = dfs(j);
          res = max(res, s);
          sum += s;
      }
      res = max(res, n - sum);
      ans = min(ans, res);
      return sum;
  }
  
  int main()
  {
      memset(h, -1 ,sizeof(h));
      cin >> n;
      for (int i = 1; i < n; i++) {
          int a, b;
          cin >> a >> b;
          add(a, b);
          add(b, a);
      }
      dfs(1);
      cout << ans << endl;
      return 0;
  }
  ```

- **宽度优先搜索**

  ```cpp
  queue<int> q;
  st[1] = true;
  q.push(1);
  
  while (q.size()) {
  		int t = q.front();
  		q.pop();
  
  		for (int i = h[t]; i != -1; i = ne[i]) {
  				int j = e[i];
  				if (!st[j]) {
  						st[j] = true;
  						q.push(j);
  				}
  		}
  }
  ```

  [例题：图中点的层次](https://www.acwing.com/problem/content/849/)

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  const int N = 1e5 + 5;
  int h[N], e[N], ne[N], idx = 0;
  int d[N];
  bool st[N];
  
  void add(int a, int b) {
      e[idx] = b, ne[idx] = h[a], h[a] = idx++;
  }
  
  int main()
  {
      int n, m;
      cin >> n >> m;
      // init the distance and the adj list
      memset(h, -1, sizeof(h));
      memset(d, -1, sizeof(d));
      while (m--) {
          int a, b;
          cin >> a >> b;
          add(a, b);
      }
      queue<int> q;
      // push the starting point
      q.push(1);
      d[1] = 0;
      while(q.size()) {
          int u = q.front();
          q.pop();
          // iterate all the linked point of current point
          for (int i = h[u]; i != -1; i = ne[i]) {
              int j = e[i];
              // push the point into the queue
              if (!st[j]) {
                  st[j] = true;
                  q.push(j);
                  // update the distance
                  d[j] = d[u] + 1;
              }
          }
      }
      cout << d[n] << endl;
      return 0;
  }
  ```



## 拓扑排序

- **思想：**邻接表存图，存完之后看看有没有入度为0的点，有的话将其加入队列，之后利用BFS的过程，每次遍历一个点，将其导出的边取消掉，从而获得新的入度为0的点，直至所有点都被遍历到。

- [例题**：**拓扑排序](https://www.acwing.com/problem/content/850/)

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  const int N = 1e5 + 5, M = 2 * N;
  int h[N], e[M], ne[M], idx = 0;
  int d[N];
  int top[N];
  
  void add(int a, int b) {
      e[idx] = b, ne[idx] = h[a], h[a] = idx++;
  }
  
  int main()
  {
      int n, m, a, b, cnt = 0;
      memset(h, -1, sizeof(h));
      cin >> n >> m;
      while(m--) {
          cin >> a >> b;
          add(a, b);
          d[b]++;
      }
      queue<int> q;
      for (int i = 1; i <= n; i++) {
          if (!d[i]) q.push(i);
      }
      while(q.size()) {
          int cur = q.front();
          q.pop();
          top[cnt++] = cur;
          for (int i = h[cur]; i != -1; i = ne[i]) {
              int j = e[i];
              d[j]--;
              if (!d[j]) {
                  q.push(j);
              }
          }
      }
      if (cnt != n) cout << -1 << endl;
      else {
          for (int i = 0; i < n; i++) {
              cout << top[i] << " ";
          }
          cout << endl;
      }
      return 0;
  }
  ```



## 最短路算法

### Dijkstra 算法（单源最短路）

- **思想：**每次找到当前距离源点最近的一个点，将其序号记录下来，同时以该点为中转点，更新（松弛）其他点到源点的距离。重复上述的过程 n - 1 次，即可更新所有点，获得答案。

- [例题：最短路](https://www.acwing.com/problem/content/851/)

  **邻接矩阵版本（复杂度：O(N * N)）**

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  const int N = 505;
  int g[N][N];
  int dist[N];
  bool st[N];
  
  int main()
  {
      int n, m;
      cin >> n >> m;
      // init the distance and adj matrix
      memset(dist, 0x3f, sizeof(dist));
      memset(g, 0x3f, sizeof(g));
      // build the adj matrix and remove the repeated edge
      for (int i = 0; i < m; i++) {
          int a, b, c;
          cin >> a >> b >> c;
          g[a][b] = min(g[a][b], c);
      }
      dist[1] = 0;
      // dijkstra, iterate n - 1 times
      for (int i = 0; i < n - 1; i++) {
          int t = -1;
          // search the nearest and unvisited point to the start
          for (int j = 1; j <= n; j++) {
              if (!st[j] && (t == -1 || dist[j] < dist[t])) {
                  t = j;
              }
          }
          
          // update the other points through current point
          for (int i = 1; i <= n; i++) {
              dist[i] = min(dist[i], dist[t] + g[t][i]);
          }
          
          // mark visited
          st[t] = true;
      }
      if (dist[n] > 0x3f3f3f3f / 2) cout << -1 << endl;
      else cout << dist[n] << endl;
      return 0;
  }
  ```

  

  - **邻接表版本（复杂度：O(MN)）**

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  const int N = 505, M = 1e5 + 5;
  int h[N], e[M], ne[M], w[M], idx = 0;
  int dist[N];
  bool st[N];
  
  void add(int a, int b, int c) {
      e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
  }
  
  int main()
  {
      int n, m;
      cin >> n >> m;
      // init the distance and adj list
      memset(dist, 0x3f, sizeof(dist));
      memset(h, -1, sizeof(h));
      while (m--) {
          int a, b, c;
          cin >> a >> b >> c;
          add(a, b, c);
      }
      
      dist[1] = 0;
      // dijkstra: iterate n - 1 times
      for (int i = 0; i < n - 1; i++) {
          int t = -1;
          // search the nearest and unvisited point to the start
          for (int i = 1; i <= n; i++) {
              if (!st[i] && (t == -1 || dist[t] > dist[i])) {
                  t = i;
              }
          }
          st[t] = true;
          // update the other points through current point
          for (int i = h[t]; i != -1; i = ne[i]) {
              int j = e[i];
              dist[j] = min(dist[j], dist[t] + w[i]); 
          }
      }
  
      if (dist[n] > 0x3f3f3f3f / 2) cout << -1 << endl;
      else cout << dist[n] << endl;
      return 0;
  }
  ```

  

  - **堆优化邻接表版本（复杂度：O(MlogN)）**

  **思想：**用pair小根堆存储与源点最近的点的编号，以及该点到源点的距离。之后小根堆堆顶的邻接表，如松弛了某个别的点，则将该点加入到优先队列中，直至优先队列为空。

  [例题：堆优化最短路](https://www.acwing.com/problem/content/852/)

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  typedef pair<int, int> PII;
  const int N = 1e6 + 5;
  int h[N], e[N], w[N], ne[N], idx = 0;
  int dist[N];
  bool st[N];
  
  void add(int a, int b, int c) {
      e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
  }
  
  int main()
  {
      int n, m;
      cin >> n >> m;
      memset(h, -1, sizeof(h));
      memset(dist, 0x3f3f3f3f, sizeof(dist));
      while (m--) {
          int a, b, c;
          cin >> a >> b >> c;
          add(a, b, c);
      }
      // init and push the start point
      priority_queue<PII, vector<PII>, greater<PII>> q;
      dist[1] = 0;
      q.push({0, 1});
      // iterate the priority_queue
      while(q.size()) {
          // get the nearest point
          auto p = q.top();
          q.pop();
          // judge visited or not
          int cur = p.second;
          if (st[cur]) continue;
          st[cur] = true;
          // update all the other points throught current point
          for (int i = h[cur]; i != -1; i = ne[i]) {
              int j = e[i];
              if (dist[j] > dist[cur] + w[i]) {
                  dist[j] = dist[cur] + w[i];
                  q.push({dist[j], j});
              }
          }
      }
      if (dist[n] > 0x3f3f3f3f / 2) cout << -1 << endl;
      else cout << dist[n] << endl;
      return 0;
  }
  ```



### Bellman-ford 算法（单源最短路）

- **思想：**该算法可以处理存在负权边（负权回路）的图，进行 n-1 次循环，每次循环将所有的边松弛一下，循环结束后，所有点到源点的距离即为最短距离。同时，可以更改循环次数，从而获得最多经过 k 条边的最短距离。

- [例题：有边数限制的最短路](https://www.acwing.com/problem/content/855/)

  **直接存边版本（复杂度：O(M * N)）**

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  const int N = 505, M = 1e5 + 5;
  const int INF = 0x3f3f3f3f;
  int dist[N], last[N];
  
  struct edge {
      int a, b, c;
  }edges[M];
  
  int main()
  {
      int n, m, k;
      cin >> n >> m >> k;
      memset(dist, 0x3f, sizeof(dist));
      dist[1] = 0;
      for (int i = 0; i < m; i++) {
          int a, b, c;
          cin >> a >> b >> c;
          edges[i] = {a, b, c};
      }
      
      // iterate k times and update all the edges
      for (int i = 0; i < k; i++) {
          memcpy(last, dist, sizeof(dist));
          for (int i = 0; i < m; i++) {
              auto e = edges[i];
              if (dist[e.b] > last[e.a] + e.c) {
                  dist[e.b] = last[e.a] + e.c;
              }
          }
      }
      
      if (dist[n] > INF / 2) cout << "impossible" << endl;
      else cout << dist[n] << endl;
      return 0;
  }
  ```



### SPFA 算法（单源最短路）

- **思想：**该算法可以处理存在负权边的图，但不能处理负权回路。数据结构采用邻接表，算法思路采用 BFS，利用队列存储距离和顶点号，每次取出队头的顶点后，松弛与该点相连的所有其他点。在BFS的过程中，弹出一个点标记为false，已在队列中的点则不需要重复加入，只需松弛即可。

- [例题：spfa求最短路](https://www.acwing.com/problem/content/853/)

  **邻接表版本（复杂度：有优化，但最坏仍会达到O(M * N)）**

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  typedef pair<int, int> pii;
  const int N = 1e5 + 5;
  int e[N], w[N], ne[N], h[N], idx = 0;
  int dist[N];
  bool st[N];
  
  void add(int a, int b, int c) {
      e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
  }
  
  int main()
  {
      int n, m;
      cin >> n >> m;
      memset(h, -1, sizeof(h));
      memset(dist, 0x3f, sizeof(dist));
      dist[1] = 0;
      queue<pii> q;
      while (m--) {
          int a, b, c;
          cin >> a >> b >> c;
          add(a, b, c);
      }
      q.push({0, 1});
      st[1] = true;
      // BFS
      while(q.size()) {
          auto p = q.front();
          q.pop();
          int cur = p.second;
          st[cur] = false;
          // update all the adjacent point
          for (int i = h[cur]; i != -1; i = ne[i]) {
              int j = e[i];
              if (dist[j] > dist[cur] + w[i]) {
                  dist[j] = dist[cur] + w[i];
                  // if marked, then no need to push
                  if (!st[j]) {
                      st[j] = true;
                      q.push({dist[j], j});
                  }
              }
          }
      }
      if (dist[n] > 0x3f3f3f3f / 2) cout << "impossible" << endl;
      else cout << dist[n] << endl;
      return 0;
  }
  ```



### SPFA 算法（判断负环）

- **思想：**该算法可以处理存在负权边的图，但不能处理负权回路。数据结构采用邻接表，算法思路采用 BFS，利用队列存储距离和顶点号，每次取出队头的顶点后，松弛与该点相连的所有其他点。在BFS的过程中，弹出一个点标记为false，已在队列中的点则不需要重复加入，只需松弛即可。

- [例题：SPFA判断负环](https://www.acwing.com/problem/content/description/854/)

  **邻接表版本（复杂度：有优化，但最坏仍会达到O(M * N)）**

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  const int N = 1e5 + 5;
  int e[N], w[N], ne[N], h[N], idx = 0;
  int dist[N];
  int cnt[N];
  bool st[N];
  
  void add(int a, int b, int c) {
      e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
  }
  
  int main()
  {
      int n, m;
      cin >> n >> m;
      bool flag = true;
      memset(h, -1, sizeof(h));
      memset(dist, 0x3f, sizeof(dist));
      queue<int> q;
      while (m--) {
          int a, b, c;
          cin >> a >> b >> c;
          add(a, b, c);
      }
      for (int i = 1; i <= n; i++) {
          // set all the point as start point
          q.push(i);
          st[i] = true;
      }
      // BFS
      while(q.size() && flag) {
          int cur = q.front();
          q.pop();
          st[cur] = false;
          // update all the adjacent point
          for (int i = h[cur]; i != -1; i = ne[i]) {
              int j = e[i];
              if (dist[j] > dist[cur] + w[i]) {
                  dist[j] = dist[cur] + w[i];
                  // update the length of the edge
                  cnt[j] = cnt[cur] + 1;
                  if (cnt[j] >= n) {
                      flag = false;
                      break;
                  }
                  // if marked, then no need to push
                  if (!st[j]) {
                      st[j] = true;
                      q.push(j);
                  }
              }
          }
      }
      if (!flag) cout << "Yes" << endl;
      else cout << "No" << endl;
      return 0;
  }
  ```



### Floyd 算法（多源最短路）

- **思想：**邻接矩阵存图的距离（不再需要dist数组），自身距离初始化为0，其余距离初始化为正无穷，算法的核心为不断利用中间点更新源点与汇点之间的最短距离，实现方式为三层循环，最外一层循环为中间点，两层内层循环分别为源点和汇点。

- [例题：Floyd求最短路](https://www.acwing.com/problem/content/856/)

  **邻接矩阵版本（复杂度：O(N * N * N)）**

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  const int N = 205;
  const int INF = 0x3f3f3f3f;
  int g[N][N];
  int dist[N][N];
  
  int main()
  {
      int n, m, k;
      cin >> n >> m >> k;
      // initialize the adjacent matrix
      memset(g, 0x3f, sizeof(g));
      for (int i = 1; i <= n; i++) {
          g[i][i] = 0;
      }
      // read the edge and keep the min edge
      while (m--) {
          int a, b, c;
          cin >> a >> b >> c;
          g[a][b] = min(g[a][b], c);
      }
      // floyd
      for (int k = 1; k <= n; k++)
          for (int i = 1; i <= n; i++)
              for (int j = 1; j <= n; j++)
                  g[i][j] = min(g[i][j], g[i][k] + g[k][j]);
      
      while (k--) {
          int x, y;
          cin >> x >> y;
          if (g[x][y] > INF / 2) cout << "impossible" << endl;
          else cout << g[x][y] << endl;
      }
      return 0;
  }
  ```



## 最小生成树算法

### Prim 算法（找最近点，适合点较少的稠密图）

- **思想：**邻接矩阵存图，维持一个连通分量，与 dijkstra 算法类似，每次找到与该连通分量最近的点（dijkstra是找与源点最近的点），将其加入连通分量中，累加其距离，并借助该点更新其他点与连通分量的距离。循环该过程 n 次后，即找到当前的最小生成树。

- [例题：Prim算法求最短生成树](https://www.acwing.com/problem/content/860/)

  **邻接表版本（复杂度：O(N * N)）**

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  const int N = 505;
  const int INF = 0x3f3f3f3f;
  int g[N][N];
  int dist[N];
  bool st[N];
  
  int main() 
  {
      int n, m;
      cin >> n >> m;
      // initialize the matrix
      memset(g, 0x3f, sizeof(g));
      memset(dist, 0x3f, sizeof(dist));
      memset(st, 0, sizeof(st));
      // read the edge and remove the repeated edege
      while (m--) {
          int a, b, c;
          cin >> a >> b >> c;
          g[a][b] = g[b][a] = min(g[a][b], c);
      }
      int res = 0;
      // iterate n times and add the new point
      for (int i = 0; i < n; i++) {
          int t = -1;
          for (int j = 1; j <= n; j++) {
              if (!st[j] && (t == -1 || dist[j] < dist[t]))
                  t = j;
          }
          if (i && dist[t] == INF) {
              res = INF;
              break;
          }
          
          // calculate the distance and mark visited 
          if (i) res += dist[t];
          st[t] = true;
          
          // update the distance of the other points
          for (int j = 1; j <= n; j++) {
              if (!st[j] && dist[j] > g[t][j]) {
                  dist[j] = g[t][j];
              }
          }
      }
      if (res > INF / 2) cout << "impossible" << endl;
      else cout << res << endl;
      return 0;
  }
  ```



### Kruskal 算法（找最短边，适合点较多的稀疏图）

- **思想：**边结构体存图，并查集存顶点。首先按照边的距离排序后遍历所有边，每次获取边的两个顶点，如果两个顶点不在同一连通分量，则合并，并将该边加入最小生成树，如果最后加入边的距离不等于 n - 1，则代表当前图没有最小生成树。

- [例题：Kruskal算法求最小生成树](https://www.acwing.com/problem/content/861/)

  **邻接矩阵版本（复杂度：O(MlogM)）**

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  const int N = 1e5 + 5, M = 2e5 + 5;
  int p[N];
  struct edge {
      int a, b, c;
  }edges[M];
  
  bool cmp (edge ea, edge eb) {
      return ea.c < eb.c;
  }
  
  int find(int x) {
      if (p[x] != x) p[x] = find(p[x]);
      else return x;
  }
  
  int main()
  {
      int n, m;
      cin >> n >> m;
      for (int i = 0; i < m; i++) {
          int a, b, c;
          cin >> a >> b >> c;
          edges[i] = {a, b, c};
      }
      sort(edges, edges + m, cmp);
      for (int i = 1; i <= n; i++) p[i] = i;
      int res = 0, cnt = 0;
      // iterate all the edges and merge the new point to the set
      for (int i = 0; i < m; i++) {
          auto e = edges[i];
          int a = e.a, b = e.b, c = e.c;
          a = find(a), b = find(b);
          if (a != b) {
              p[a] = b;
              res += c;
              // calculate the current size of edges
              cnt++;
          }
      }
      if (cnt != n - 1) cout << "impossible" << endl;
      else cout << res << endl;
      return 0;
  }
  ```



## 二分图算法

### 染色法判定二分图（DFS）

- **思想：**邻接表存图，采用DFS遍历所有与当前点相连的点，如相连点未被标记（未标记为0，其余两个颜色为1和2），则标记为不同的颜色，若已标记，则判断颜色是否相等，相等则返回false。

- [例题：染色法判定二分图](https://www.acwing.com/problem/content/862/)

  **邻接表版本（复杂度：O(M * N)）**

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  const int N = 1e5 + 10, M = 2 * N;
  int h[N], e[M], ne[M], idx = 0;
  int st[N];
  
  void add(int a, int b) {
      e[idx] = b, ne[idx] = h[a], h[a] = idx++;
  }
  
  // dfs
  bool dfs(int u, int color) {
      st[u] = color;
      // iterate all the adjacent point
      for (int i = h[u]; i != -1; i = ne[i]) {
          int j = e[i];
          // if not visited, give it another color
          if (!st[j]) {
              if (!dfs(j, 3 - color)) return false;
          }
          // if visited, judge the color
          else if (st[j] == color) return false;
      }
      return true;
  }
  
  int main()
  {
      int n, m;
      cin >> n >> m;
      memset(h, -1, sizeof(h));
      bool flag = true;
      while (m--) {
          int a, b;
          cin >> a >> b;
          add(a, b);
          add(b, a);
      }
      // iterate all the points
      for (int i = 1; i <= n; i++) {
          if (!st[i]) {
              if (!dfs(i, 1)) {
                  flag = false;
                  break;
              }
          }
      }
      if (flag) cout << "Yes" << endl;
      else cout << "No" << endl;
      return 0;
  }
  ```



### 染色法判定二分图（BFS）

- **思想：**邻接表存图，采用BFS从起点开始访问，同时查询所有与当前点相连的点，如相连点未被标记（未标记为0，其余两个颜色为1和2），则标记为不同的颜色，若已标记，则判断颜色是否相等，相等则返回false。

- [例题：染色法判定二分图](https://www.acwing.com/problem/content/862/)

  **邻接表版本（复杂度：O(M * N)）**

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  typedef pair<int, int> PII;
  int n, m;
  const int N = 1e5 + 5, M = 2e5 + 5;
  int h[N], e[M], ne[M], idx = 0;
  PII q[N];
  int st[N];
  
  void add(int a, int b) {
      e[idx] = b, ne[idx] = h[a], h[a] = idx++;
  }
  
  bool bfs(int u) {
      // add the start point
      int hh = 0, tt = -1;
      q[++tt] = {u, 1};
      st[u] = 1;
      while (hh <= tt) {
          auto t = q[hh++];
          int cur = t.first, c = t.second;
          // iterate all the adjacent points
          for (int i = h[cur]; i != -1; i = ne[i]) {
              int j = e[i];
              // if not visited, give it another color
              if (!st[j]) {
                  st[j] = 3 - c;
                  q[++tt] = {j, 3 - c};
              }
              // if visited, judge the color duplicate or not
              else if (st[j] == c) return false;
          }
      }
      return true;
  }
  
  int main()
  {
      cin >> n >> m;
  
      memset(h, -1, sizeof h);
  
      while (m -- )
      {
          int a, b;
          cin >> a >> b;
          add(a, b), add(b, a);
      }
  
      bool flag = true;
      for (int i = 1; i <= n; i ++ )
          if (!st[i])
          {
              if (!bfs(i))
              {
                  flag = false;
                  break;
              }
          }
  
      if (flag) cout << "Yes" << endl;
      else cout << "No" << endl;
      return 0;
  }
  ```



### 匈牙利算法求最大匹配

- **思想：**邻接表存图，遍历第一部分的点，统计最大匹配数。统计的过程为遍历第二部分的点，找到所有相邻点，并尝试与第一部分当前正在遍历的点进行匹配，如果第二部分的点未被匹配或者匹配的第一部分点可以找到其他类型的匹配，则**优先匹配当前这一对**。

- [例题：二分图的最大匹配](https://www.acwing.com/problem/content/863/)

  **邻接表版本（复杂度：O(MN)）**

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  const int N = 505, M = 1e5 + 5;
  int h[N], e[M], ne[M], idx = 0;
  int match[N];
  bool st[N];
  
  void add(int a, int b) {
      e[idx] = b, ne[idx] = h[a], h[a] = idx++;
  }
  
  // find the couple
  int find(int x) {
      // find all the possible match
      for (int i = h[x]; i != -1; i = ne[i]) {
          int j = e[i];
          // if not visited
          if (!st[j]) {
              // mark visited
              st[j] = true;
              // if not match or if we can find a new couple
              // match it
              if (match[j] == 0 || find(match[j])) {
                  match[j] = x;
                  return true;
              }
          }
      }
      return false;
  }
  
  int main()
  {
      int n1, n2, m;
      cin >> n1 >> n2 >> m;
      memset(h, -1, sizeof(h));
      while (m--) {
          int a, b;
          cin >> a >> b;
          add(a, b);
      }
      
      int res = 0;
      for (int i = 1; i <= n1; i++) {
          // clear the state and find some new couples
          memset(st, false, sizeof(st));
          if (find(i)) res++;
      }
      cout << res << endl;
      return 0;
  }
  ```