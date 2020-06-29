---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "AlgorithmInproblemsolving"
date: 2020-06-27T17:06:38+08:00
lastmod: 2020-06-27T17:06:38+08:00
draft: false
description: ""
show_in_homepage: true
description_as_summary: false
license: ""

tags: ["algorithm", "problem-solving"]
categories: []

featured_image: ""
featured_image_preview: ""

comment: true
toc: true
auto_collapse_toc: true
math: true
---

## 排序算法

### 快速排序
- Quicksort is a divide-and-conquer algorithm. It works by selecting a 'pivot' element from the array and partitioning the other elements into two sub-arrays, according to whether they are less than or greater than the pivot. The sub-arrays are then sorted recursively. 

<!-- ```c++
class Solution {
public:
    int partition(vector<int>& nums, int l, int r){
        int val = nums[r];
        int i = l-1;
        for(int j = l; j < r; ++j){
            if(nums[j] < val){
                i++;
                int tmp = nums[j];
                nums[j] = nums[i];
                nums[i] = tmp;
            }
        }
        i++;
        nums[r] = nums[i];
        nums[i] = val;
        return i;
    }
    void quicksort(vector<int>& nums, int l, int r){
        if(l < r){
            int p = partition(nums, l, r);
            quicksort(nums, l, p-1);
            quicksort(nums, p+1, r);
        }
    }
    vector<int> sortArray(vector<int>& nums) {
        quicksort(nums,0,nums.size()-1);
        return nums;
    }
};
``` -->

### 归并排序
- Conceptually, a merge sort works as follows:
  - Divide the unsorted list into n sublists, each containing one element (a list of one element is considered sorted).
  - Repeatedly merge sublists to produce new sorted sublists until there is only one sublist remaining. This will be the sorted list.

<!-- ```c++
class Solution {
public:
    void mergesort(vector<int>& nums, int l, int r, vector<int>& tmp){
        if(l < r){
            int m = (l+r)/2;
            mergesort(nums,l,m,tmp);
            mergesort(nums,m+1,r,tmp);
            tmp.clear();
            int i1 = l, i2 = m+1;
            while(i1 <= m && i2 <= r){
                if(nums[i1] < nums[i2]){
                    tmp.push_back(nums[i1]);
                    i1++;
                }
                else{
                    tmp.push_back(nums[i2]);
                    i2++;
                }
            }
            while(i1 <= m){
                tmp.push_back(nums[i1]);
                i1++;
            }
            while(i2 <= r){
                tmp.push_back(nums[i2]);
                i2++;
            }
            copy(tmp.begin(), tmp.end(), nums.begin() + l);
        }
    }
    vector<int> sortArray(vector<int>& nums) {
        vector<int> tmp;
        mergesort(nums,0,nums.size()-1,tmp);
        return nums;
    }
};
``` -->

### 冒泡排序
- Repeat traversing through the array and swapping adjacent inversions until there are no inversions.
<!-- ```c++
class Solution {
public:
    vector<int> sortArray(vector<int>& nums) {
        for(int i = 0; i < nums.size(); ++i){
            for(int j = 0; j < nums.size() - 1; ++j){
                if(nums[j] > nums[j+1]){
                    int tmp = nums[j];
                    nums[j] = nums[j+1];
                    nums[j+1] = tmp;
                }
            }
        }
        return nums;
    }
};
``` -->

### 选择排序
- Loop n times, find the i-th smallest element in the i-th loop, then exchange with the i-th element in the array, and finally get an ordered array.
<!-- ```c++
class Solution {
public:
    vector<int> sortArray(vector<int>& nums) {
        for(int i = 0; i < nums.size(); ++i){
            int min_index = i;   
            for(int j = i+1; j < nums.size(); ++j){
               if(nums[j] < nums[min_index]){
                   min_index = j;
               }
            }
            int tmp = nums[i];
            nums[i] = nums[min_index];
            nums[min_index] = tmp;
        }
        return nums;
    }
};
``` -->

### 插入排序
- Loop n times, In the i-th loop, insert the i-th element into the first i-1 elements of the array.


### 动态规划
-  Dynamic programming simplifies a complicated problem by breaking it down into simpler sub-problems in a recursive manner.
-  It is essentially exhaustive, but avoids repeated calculation of sub-problems by saving the results of sub-problems already calculated.
  
#### 最优子结构
-  if a problem can be solved optimally by breaking it into sub-problems and then recursively finding the optimal solutions of the sub-problems, then it is said to have **optimal substructure.**


### 贪心算法
- the greedy algorithm also ask for the problem satitfying the optimal substructure. But unlike dynamic programming, it makes the current optimal choice every time and then solves the remaining sub-problem. it constructs global optimal solution using local optimal solutions.
- cut-paste证明


## 图算法

### 深度优先搜索
- Depth-first search (DFS) is an algorithm for traversing or searching tree or graph data structures. The algorithm starts at the given node and explores as far as possible along each branch before backtracking.
- it can be used to get the topological sorting of a tree or a graph.  or the strong connected component(Any point in the component can be passed to all the points in this component)


### 广度优先搜索
- Breadth-first search (BFS) is an algorithm for traversing or searching tree or graph data structures. It starts at the given node and explores all of the neighbor nodes at the present depth before moving on to the nodes at the next depth level.

### 最小生成树-prim
- Prim's algorithm is a greedy algorithm that finds a minimum spanning tree for a weighted undirected graph.
- The algorithm may informally be described as performing the following steps:
  - Initialize a tree with a single vertex, chosen arbitrarily from the graph.
  - Grow the tree by one edge: of the edges that connect the tree to vertices not yet in the tree, find the minimum-weight edge, and transfer it to the tree.
  - Repeat step 2 (until all vertices are in the tree).

### 最小生成树-kruskal
- Kruskal algorithm is a greedy algorithm that finds a minimum spanning tree for a connected weighted graph. it works as follows:
  - Create a forest F, where each vertex in the graph is a seperate tree.
  - Create a set S containing all the edges.
  - then do the following things: while S is not empty
    - remove a edge e with minimum weight from S.
    - if the edge e connects two different trees then add it in F to combine two trees into one single tree.