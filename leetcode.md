## [310. 最小高度树](https://leetcode-cn.com/problems/minimum-height-trees/)

![image-20220406152904227](https://gitee.com/ingachin/mdimage/raw/master/image-20220406152904227.png)



dfs  逐层剔除叶节点，最后剩余的节点即为答案  



```c++
class Solution {
public:
    vector<int> findMinHeightTrees(int n, vector<vector<int>>& edges) {
        vector<vector<int>> graph(n);//临界表
        vector<int> ans;
        int degree[n];//统计每一个点的度
        if(n == 1){
            ans.push_back(0);
            return ans;
        }
        memset(degree,0,sizeof(degree));
        for(int i = 0;i < edges.size() ;i++){
            //构建邻接表，统计每个点的度
            graph[edges[i][0]].push_back(edges[i][1]);
            graph[edges[i][1]].push_back(edges[i][0]);
            degree[edges[i][0]]++;
            degree[edges[i][1]]++;
        }
        queue<int> q;
        for(int i = 0;i < n;i++){
            if(degree[i] == 1){//将每一个叶节点放入队列
                q.push(i);
            }
        }
        while(!q.empty()){//dfs
            ans.clear();
            int size = q.size();
            for(int j = 0;j < size;j++){
                int u = q.front();
                ans.push_back(u);
                q.pop();
                for(int i = 0;i < graph[u].size() ;i++){
                    degree[graph[u][i]] --;
                    if(degree[graph[u][i]] == 1){
                        q.push(graph[u][i]);
                    }
                }
            }
        }
        return ans;
    }
};
```

  

## 试题E:路径

![image-20220407095130056](https://gitee.com/ingachin/mdimage/raw/master/image-20220407095130056.png)

dijkstra算法

```c++
#include <bits/stdc++.h>
#define INF 0x3f3f3f3f
#define MAXLEN 2200
using namespace std;
int graph[MAXLEN][MAXLEN];
int dis[MAXLEN];
int vis[MAXLEN];
void dijikstra(){
	int n = 2021;
	for(int i = 1;i<=n;i++){//循环n次dijikstra
		int t = -1;//最小cost节点
		for(int j = 1;j<=n ;j++){//寻找当前距离最小节点
			if((dis[j] < dis[t] || t == -1) && !vis[j]){//如果找到节点比之前的节点的值要小且未被访问过，或者没有找到过节点则更新最小节点
				t = j;
			}
		}
		vis[t] = 1;//锁死当前选择的节点
		for(int j=1;j<=n;j++){
			dis[j] = min(dis[j] , dis[t] + graph[t][j]);//根据选择的最小节点更新最小路径
		}
	}
}
int gcd(int a, int b)
{
    return b ? gcd(b, a % b) : a;
}
int lcm(int a,int b){
    return a*b/gcd(a,b);
}
int main(){
	memset(dis,INF,sizeof(dis));
	memset(vis,0,sizeof(vis));
	memset(graph,INF,sizeof(graph));
    
	dis[1] = 0;//头节点的距离为0
    //根据题目描述建图
	for(int i = 1;i<=2021;i++){
		for(int j = 1;j<=2021;j++){
			if(i!=j){
                if(fabs(i-j)<=21){
                    graph[i][j]=lcm(i,j);
                    graph[j][i]=lcm(i,j);
                }
                else{
                    graph[i][j]=0x3f3f3f3f;
                    graph[j][i]=0x3f3f3f3f;
                }}
		}
	}
    
	dijikstra();
    
	cout<<dis[2021];

    
	return 0;
}
```

## [429. N 叉树的层序遍历](https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/)

![image-20220408195217136](https://gitee.com/ingachin/mdimage/raw/master/image-20220408195217136.png)

**思路：bfs广度优先搜索模板**

```c++
/*
// Definition for a Node.
class Node {
public:
    int val;
    vector<Node*> children;
    
    Node() {}
    
    Node(int _val) {
        val = _val;
    }
    
    Node(int _val, vector<Node*> _children) {
        val = _val;
        children = _children;
    }
};
*/
class Solution {
public:
    vector<vector<int>> levelOrder(Node* root) {
        queue<Node*> q;//队列存储元素
        vector<vector<int>> ans;
        if(!root) return ans;//如果根节点为空，直接返回空集合
        q.push(root);//将根节点放入队列当中
        while(!q.empty()){
                int size = q.size();//获取当前层队列的长度
                vector<int> temp;
                while(size--){//当前层不为空的时候
                    Node *p = q.front();q.pop();//弹出队列顶部的元素
                    temp.push_back(p->val);//将元素放入集合当中
                    for(auto u : p -> children){//将子节点放入队列当中
                        q.push(u);
                    }
                }
                ans.push_back(temp);
        }
        return ans;
    }
};
```

