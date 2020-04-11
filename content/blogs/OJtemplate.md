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
#include <iostream>
#include <vector>
#include <algorithm>

using  namespace std;

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