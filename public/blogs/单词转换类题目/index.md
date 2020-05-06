# 单词转换类题目



leetcode 127 和 leetcode (51)433，大致题意就是给你一个开始字符串start，一个目标字符串end，以及一个字符串数组bank，需要找到一条最短的由start经过bank中的字符串到达end的转换路经，每次转换只能改变一个字符。

<!--more-->

很容易可以看出可以使用dfs或者bfs，但是如何简洁高效地表达转换过程？ 如果直接很暴力地对于当前字符串，遍历每一位，考虑每一位可替换的字符，形成新字符串后再遍历bank找到匹配的字符串的话，显然时间开销太大。
可以使用map来减小时间开销，只需要先预处理一遍，然后之后就不需要每次都遍历bank查找。

```C++
map<string,vector<string>> mymap;
map<string,vector<string>>::iterator myptr;
//遍历bank中每一个字符串
for(int i = 0; i < bank.size(); ++i){
    //考虑每一个位置
    for(int j = 0; j < bank[i].size(); ++j){
        //将第j个位置替换为通用匹配符*
        string tmp = bank[i].substr(0,j) + "*" + bank[i].substr(j+1);
        myptr = mymap.find(tmp);
        if(myptr == mymap.end()){
            //还没有这样的映射
            vector<string> newvec;
            newvec.push_back(bank[i]);
            mymap[tmp] = newvec;
        }
        else{
            //已经存在这样的映射
            vector<string> vec = myptr->second;
            vec.push_back(bank[i]);
            mymap[tmp] = vec;
        }
    }
}
```

处理完后map中的表现为"h*t" --> {"hot","hit"...},之后的bfs过程如下

```C++
set<string> ansset;
queue<pair<string,int>> que;
ansset.insert(start);
que.push(make_pair(start,0));
while(!que.empty()){
    string curstr = que.front().first;
    int deep = que.front().second;
    que.pop();
    for(int i = 0; i < curstr.size(); ++i){
        string tmpstr = curstr.substr(0,i)+"*"+curstr.substr(i+1);
        myptr = mymap.find(tmpstr);
        if(myptr != mymap.end()){
            vector<string> candidates = myptr->second;
            for(int j = 0; j < candidates.size(); ++j){
                if(candidates[j] == curstr) continue;
                if(candidates[j] == end) return deep + 1;
                if(ansset.count(candidates[j]) == 0){
                    que.push(make_pair(candidates[j],deep+1));
                    ansset.insert(candidates[j]);
                }
            }
        }
    }
}
```
