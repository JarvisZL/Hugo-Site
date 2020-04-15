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