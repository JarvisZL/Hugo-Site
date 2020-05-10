---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "OJtemplate"
date: 2020-04-11T21:13:31+08:00
lastmod: 2020-04-11T21:13:31+08:00
draft: false
description: ""
show_in_homepage: false
description_as_summary: false
license: ""

tags: [OJ模板]
categories: []

featured_image: ""
featured_image_preview: ""

comment: true
toc: true
auto_collapse_toc: true
math: true
---

一些常用算法的模板
<!--more-->

### 队列版拓扑排序
```C++
//统计每个点的入度，压入入度为0。
//弹出后相当于删除这个点，所以其连接的点的入度-1，然后重复。
//拓扑排序就是弹出顺序，弹出的结点数量==总结点数量，没有环路，反之有环路。
class Solution {
public:
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        vector<int> degree(numCourses,0);
        for(int i = 0; i < prerequisites.size(); ++i){
            degree[prerequisites[i][1]]++;
        }
        queue<int> que;
        for(int i = 0; i < degree.size(); ++i){
            if(degree[i] == 0){
                que.push(i);
            }
        }
        int cnt = 0;
        while(!que.empty()){
            int tmp = que.front();
            que.pop();
            cnt++;
            for(int i = 0; i < prerequisites.size(); ++i){
                if(prerequisites[i][0] == tmp){
                    degree[prerequisites[i][1]]--;
                    if(degree[prerequisites[i][1]] == 0){
                        que.push(prerequisites[i][1]);
                    }
                }
            }
        }
        return cnt == numCourses;
    }
};
``


### 最小生成树-kruscal
```C++
int T,N;
struct Node{
    int u,v,w;
    bool operator < (const struct Node& tmp) const {
        return w < tmp.w;
    }
};
vector<struct Node> edges;
vector<int> parents;

int find(int index){
    if(parents[index] != index)
        parents[index] = find(parents[index]);
    return parents[index];
}

int main(){
    cin >> T;
    while(T--){
        cin >> N;
        edges.clear();
        parents.clear();
        int ans = 0;
        for(int i = 0; i < N; ++i)
            parents.push_back(i);
        for(int i = 0; i < N; ++i){
            for(int j = 0; j < N; ++j){
                int tmpw;
                cin >> tmpw;
                if(i >= j) continue;
                struct Node tmpnode = {i,j,tmpw};
                edges.push_back(tmpnode);
            }
        }
        sort(edges.begin(), edges.end());
        for(int i = 0; i < edges.size(); ++i){
            //cout << edges[i].u << " " << edges[i].v << " " << edges[i].w << endl;
            int pu = find(edges[i].u);
            int pv = find(edges[i].v);
            if(pv != pu){
                // 输出最小生成树最大值
                ans = max(edges[i].w, ans);
                parents[pu] = pv;
            }
        }
        cout << ans <<endl;
    }
}
```

### 最小生成树-prim算法
```C++
int T,N;
int main(){
    cin >> T;
    while(T--){
        cin >> N;
        vector<int> matrix[502];
        bool visit[502];
        memset(visit,0,sizeof(visit));
        priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>> > que;
        int ans = 0;
        for(int i = 0; i < N; ++i){
            for(int j = 0; j < N; ++j){
                int tmp; cin >> tmp;
                matrix[i].push_back(tmp);
            }
        }
        for(int i = 1; i < N; ++i){
            //默认根据first排序
            que.push(make_pair(matrix[0][i],i));
        }
        visit[0] = true;
        while(!que.empty()){
            int dis = que.top().first;
            int u = que.top().second;
            que.pop();
            //cout << u << " " << dis << endl;
            if(visit[u]) continue;
            visit[u] = true;
            ans = max(ans,dis);
            for(int i = 0; i < N; ++i){
                if(visit[i]) continue;
                //cout << "push: " << i << " " << matrix[u][i] << endl;
                que.push(make_pair(matrix[u][i],i));
            }
        }
        cout << ans << endl;
    }
    return 0;
}
```

### 单源最短路-Bellman_Ford
```C++
#define INF 0x3f3f3f3f
#define MAXM 105
struct E{
    int u,v,w;
};
int d[MAXM], N,A,B;;
vector<struct E> G;

void Bellman_Ford(){
    for(int i = 0; i < N - 1; ++i){
        for(int j = 0; j < G.size(); ++j){
            if(d[G[j].v] > d[G[j].u] + G[j].w){
                d[G[j].v] = d[G[j].u] + G[j].w;
            }
        }
    }
    cout << (d[B] == INF ? -1 : d[B]);
}
int main(){
    cin >> N >> A >> B;
    memset(d,INF,sizeof(d));
    d[A] = 0;
    for(int i = 1; i <= N; ++i){
        int K; cin >> K;
        for(int j = 0; j < K; ++j){
            int t; cin >> t;
            struct E e = {i,t,0};
            e.w = j == 0 ? 0 : 1;
            G.push_back(e);
        }
    }

    Bellman_Ford();
    return 0;
}
```

### 单源最短路-Dijkstra
```C++
#define MAXM 105
#define INF 0x3f3f3f3f
int d[MAXM], N,A,B;;
vector<int> G[MAXM];
bool visit[MAXM];

void Dijkstra(){
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>> > que;
    que.push(make_pair(d[A],A));
    while(!que.empty()){
        int dis = que.top().first;
        int from = que.top().second;
        que.pop();
        if(visit[from]) continue;
        visit[from] = true;
        for(int i = 1; i <= N; ++i){
            if(G[from][i] != -1 && !visit[i]){
                if(d[i] > dis + G[from][i]){
                    d[i] = dis + G[from][i];
                    que.push(make_pair(d[i],i));
                }
            }
        }
    }
    cout << (d[B] == INF ? -1 : d[B]);
}

int main(){
    cin >> N >> A >> B;
    memset(d,INF,sizeof(d));
    memset(visit,0,sizeof(visit));
    d[A] = 0;
    for(int i = 1; i <= N; ++i){
        int K; cin >> K;
        G[i] = vector<int>(N+1, -1);
        for(int j = 0; j < K; ++j){
            int t; cin >> t;
            G[i][t] = j == 0 ? 0 : 1;
        }
    }

    Dijkstra();
    return 0;
}
```

### 最短路-Floyd-Warshall
```C++
//leetcode 网络延迟时间
 int networkDelayTime(vector<vector<int>>& times, int N, int K) {
        int G[101][101];
        memset(G,0x3f3f3f3f,sizeof(G));
        for(int i = 1; i <= N; ++i){
            G[i][i] = 0;
        }
        for(int i = 0; i < times.size(); ++i){
            //有向图
            G[times[i][0]][times[i][1]] = times[i][2];
        }
        for(int k = 1; k <= N; ++k){
            for(int i = 1; i <= N; ++i){
                for(int j = 1; j <= N; ++j){
                    G[i][j] = min(G[i][j],G[i][k]+G[k][j]);
                }
            }
        }
        int ans = 0;
        for(int i = 1; i <= N; ++i){
            ans = max(ans, G[K][i]);
        }
        
        return (ans == 0x3f3f3f3f ? -1 : ans);
    }
```

### 最大匹配-匈牙利算法
```C++
//leetcode LCP4
//MaxmNode = 64
vector<int> G[64];
int match[64];
bool check[64];
int dx[4] = {0,0,1,-1},dy[4] = {1,-1,0,0};
bool dfs(int u){
    for(int i = 0; i < G[u].size(); ++i){
        if(!check[G[u][i]]){
            check[G[u][i]] = true;
            if(match[G[u][i]] == -1 || dfs(match[G[u][i]])){
                match[u] = G[u][i];
                match[G[u][i]] = u;
                return true;
            }
        }
    }
    return false;
}
int domino(int n, int m, vector<vector<int>>& broken) {
    for(int i = 0; i < n; ++i){
        for(int j = 0; j < m; ++j){
            vector<int> tmp = {i, j};
            //函数find属于namespace std
            if((find(broken.begin(),broken.end(),tmp)) != broken.end()) continue;
            for(int k = 0; k < 4; ++k){
                int x = i + dx[k], y = j + dy[k];
                if(x < 0 || x >= n || y < 0 || y >= m) continue;
                tmp = {x, y};
                if((find(broken.begin(),broken.end(),tmp)) != broken.end()) continue;
                G[i*m+j].push_back(x*m+y);
            }
        }
    }
    memset(match,-1,sizeof(match));
    int ans = 0;
    for(int i = 0; i < n*m; ++i){
        if(match[i] == -1){
            memset(check,0,sizeof(check));
            if(dfs(i))
                ans++;
        }
    }
    return ans;
}  
```

### 归并排序求逆序对
```C++
class Solution {
    int mergesort(vector<int>& nums, vector<int>& tmp,int l, int r){
        if(l >= r)
            return 0;
        int mid = (l+r)/2;
        int ret = 0;
        ret += mergesort(nums,tmp,l,mid) + mergesort(nums,tmp,mid+1,r);
        int li = l, ri = mid + 1;
        tmp.clear();
        while(li <= mid && ri <= r){
            if(nums[li] <= nums[ri]){
                tmp.push_back(nums[li++]);
            }
            else{
                tmp.push_back(nums[ri++]);
                ret += mid - li + 1;
            }
        }
        while(li <= mid){
            tmp.push_back(nums[li++]);
        }
        while(ri <= r){
            tmp.push_back(nums[ri++]);
        }
        copy(tmp.begin(), tmp.end(),nums.begin()+l);
        return ret;
    }
public:
    int reversePairs(vector<int>& nums) {
        vector<int> tmp;
        return mergesort(nums,tmp,0,nums.size()-1);
    }
};
```

### tarjan LCA离线算法(最近公共子结点)
```C++
//该算法使用DFS+并查集的方式
int find(int x){
    if(x != f[x])
        f[x] = find(f[x]);
    return f[x];
}
void join(int x, int y){
    int px = find(x);
    int py = find(y);
    if(px != py)
        f[py] = px;//此处应该跟调用时对应
}

void tarjan(int x){
    visit[x] = 1;
    //vector<int> G[MAXM];
    for(int i = 0; i < G[x].size(); ++i){
        tarjan(G[x][i]);
        join(x,G[x][i]);
    }

    //查询
    //vector<int> Q[MAXM]表示其中一个查询点是x时另一个查询点的集合
    for(int i = 0; i < Q[x].size(); ++i){
        if(visit[Q[x][i]] == 1)
            //ans is the set of answers
            ans.insert(find(Q[x][i]));
    }
}

```