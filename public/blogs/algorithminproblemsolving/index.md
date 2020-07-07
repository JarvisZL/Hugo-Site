# AlgorithmInproblemsolving


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

## 动规和贪心

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

### 单元最短路-Bellmanford 
- The Bellman–Ford algorithm is an algorithm that computes shortest paths from a single source vertex to all of the other vertices in a weighted digraph.**(O(VE))**
- If a graph contains a "negative cycle" that is reachable from the source, the Bellman–Ford algorithm can detect and report the negative cycle
- works:
  - Before show how it works, I need to introduce the relax operation. assume we use an array **d** to record the current shortest path from single source to others. we name the single source **s** and two other vetices **i** and **j** which are connected by an edge. the relax is that if $d[i] < d[j] + w(i,j)$, then we set $d[i]$ with $d[j] + w(i,j)$
  - it works as follows:
    - Initialize the array d with the weight of edges.
    - loop V - 1 times, each time relax every edge in the graph.
    - if there is one edge can be relaxed after V - 1 loops, then there is a negative cycle in the graph. 

### 单元最短路-Dijksta
- The Dijstra algorithm is an algorithm that computes shortest paths from a single source vertex to all of the other vertices in a weighted digraph.
- It assumes there is no negative edge in the graph. $O(V^2)$ for array-based priorityqueue, and $O(VlgV + E)$ for Fibonacci heap
- works:
  - ....
  - it works as follows:
    -  Initialize the array d with the weight of edges and mark all nodes unvisited.
    -  Repeatly, mark the node  **i** visited and node **i** satisfies the d[i] is the smallest in values of unvisited nodes in array d . and then relax the edges which connect with the node **i**.
    -  when all the nodes are visited, we get the shortest paths record in array d.


### 多源最短路-Floyd-Warshall
- Floyd–Warshall algorithm is an algorithm for finding shortest paths in a weighted graph with positive or negative edge weights (but with no negative cycles).
- it works as follows:
  - use a matrix **d** to record the shortest paths.
  - Initialize.
  - The main block of the algorithm has three layers loop. the second loop traverses through the start point i, and the third loop traverses through the end point j. The outter loop traverses through the node k that may in the path from i to j, and set d[i][j] with the minimum of (d[i][k] + d[k][j])

## 流

### 流网络(flow network)
-  a flow network is a directed graph where each edge has a capacity and each edge receives a flow. The amount of flow on an edge cannot exceed the capacity of the edge.  
-  The node where all the flows come from called source, the node where all the flows goes to called sink.  A flow must satisfy the restriction that the amount of flow into a node equals the amount of flow out of it, unless it's source or sink.

### 残存网络(residuals network)
- the residuals capacity of an edge is its capacity minus the amount of flow on it.
- if the residuals capacity of an edge is positive, add it in the residuals network.

### 增广路径(augmenting paths)
- an augmenting path is a path in the residuals network.
- A network is at maximum flow if and only if there is no augmenting path in the residual network $G_f$.


## P vs  NP
- P: if a problem can be solved by some algorithms in polynomial time, the problem is a P problem.
- NP: if a problem can be verified with a answer in polynomial time, the problem is a NP problem.
- NP-hard: NP-hard problems are those at least as hard as NP problems. all the NP problems can be reduced to NP-hard problems in polynomial time.
- NP-completeness: if a problem is a NP problem and also a NP-hard problem, then it's a NP-competeness problem.

- NP-completeness problem: 
  - 图中最大团问题(团：任意两点都有一条边即完全图)
  - 图中最小顶点覆盖问题
  - 哈密尔顿回路问题
  - 旅行商问题
