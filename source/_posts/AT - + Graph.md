---
title: AT - + Graph
tags:
  - AtCoder
  - Graphs
categories:
  - ACM
date: 2018-07-09 20:39:42
---

腦袋卡卡的甚麼都想不出來

# Problem
Kenkoooo 發現一個simple connected graph.1~n的點給你邊u,v然後還有s
他要在每個點上寫一個正數
任一邊滿足`u+v=s`
問你有幾種可能

# Constraints
* \\(2\leq n\leq 10^{5}\\)
* \\(1\leq m\leq 10^{5}\\)
* \\(1\leq u_i < v_i \leq n\\)
* \\(2\leq s_i\leq 10^{9}\\)
* if \\(i\neq j\\) then,\\(u_i\neq u_j\\) or \\(v_i\neq v_j\\)

# Solution
我們先考慮Sample Test
Input:
4 3
1 2 6
2 3 7
3 4 5

![](https://i.imgur.com/Kz64cYf.png)

目前Node 1範圍[1,INF] 因為要求正數
來看邊 1 2 6
因為val[1]+val[2]=6
現在Node 1的範圍變成[1,5] val[2]在[5,1]
那如果把所有圖的限制都考慮進去就得出了答案
問題來了 怎麼維護這個範圍

先假設點1為一個固定的數字 比如:1
跑整張圖
但像上個例子的點2 就會暫時維持在點5 最大值
過程中有些點要考慮**他能多大**或者**他能多小**
可以用 vis[v]=vis[u]^1 0,1來區分 像二分圖或相鄰不同色一樣

再來還有一個問題是**環**
遇到已經遇過的 就是有**環**的時候
兩種情況
他的vis[u]的情況是不同的
那我們只要判斷他們何不合就可以 就跟沒環的情況一樣
反之
![](https://i.imgur.com/N4oOehs.png)
如圖考慮1-2 2-3後
範圍值變化
[1,INF]
1-2 ->[1,5]
2-3 ->[2,5]

val[1]=0
val[2]=6
val[3]=-1
考慮邊1-3發現**環**
而且vis[u]情況相同
當前要求8 目前0-1 差距9
這裡就發現無解了
為甚麼?
因為假設你把1的值+1,2就會-1,3就會+1 整體+2
簡單說奇環的時候與要求相差必為偶數 否則無解 此時vis[]情況也是相反
vis[]其實可以想成是當1增加時是否增加還是減少的type 和1一樣就是同個type剛好也會出現奇環

換成這張圖
![](https://i.imgur.com/ZXqJwJH.png)
現在有解了
不過需要增加Node 1的值
實作上就是更新min, max的值
設相差為diff
變動量就是\\(\frac {diff}{2}\\)

還有一點要注意就是
如果當前點的type是要增加的 結果diff是負數 則無解 反之亦同

基本上的想法就是維護滿足條件的min,max來求可能數
在滿足條件是某方面單調的時候可以這樣搞


# Code
```cpp
#include <bits/stdc++.h>
#define LL long long
#define Accel ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
#define Re(a,b) memset(a,b,sizeof a)
#define F first
#define S second
#define MP make_pair
#define PB push_back
#define ALL(X) (X).begin(), (X).end()
#define FOR(I, A, B) for (int I = (A); I <= (B); I++)
#define REP(I, N) for (int I = 0; I < (N); I++)
#define PII pair<int,int>
#define PLL pair<LL,LL>
#define SZ(a) ((int)a.size())
//#pragma GCC optimize(2)
using namespace std;
const int N=1e5+10;
const int M=1e9+7;
const int inf=1e9+7;
const double eps=1e-6;
typedef complex<double>C;
const double PI(acos(-1.0));
//for(int mask=i;mask>0;mask=(mask-1)&i)
//int a[25]= { 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97 };
//__gcd, atan2(y,x)=y/x , __int128
//c(n,k)=c(n-1,k-1)+c(n-1,k)
//c(i,r),i for r to n =c(n+1,r+1)
//void add(int x){while(x<N)BIT[x]++,x+=x&-x;}
//void add(int x){while(x<N)BIT[x]++,x+=x&-x;}
//int sum(int x){int t=0;while(x){t+=BIT[x];x-=x&-x;}return t;}
//LL pw(LL a,LL b){LL t=1;for(;b;b>>=1,a=a*a%M)b&1?t=t*a%M:0;return t;}
//log() = ln() , log(x)/log(y)=log(y)-base-x
//INT_MAX 127,INT_MIN 128
//int dw[4][2]={{0,1},{1,0},{0,-1},{-1,0}};
//int dw[8][2]={{0,1},{1,0},{0,-1},{-1,0},{1,1},{1,-1},{-1,1},{-1,-1}};
//cout<<fixed<<setprecision(12)<<ans<<endl;
//__builtin_popcount(mask)
int u,v;
LL s;
vector<pair<int,LL>>g[N];
int n,m;
vector<int>vis;//一邊初始最小維護變大的值 ,另邊初始最大維護變小的值 
vector<int>w;
LL mn,mx;
void dfs(int u){
	for(auto it:g[u]){
		int v=it.F;LL s=it.S;
		if(vis[v]==-1){
			vis[v]=vis[u]^1;
			w[v]=s-w[u];
			if(vis[v]){
				mn=max(mn,1LL-w[v]);
			}
			else{
				mx=min(mx,w[v]-1LL);
			}
			if(mn>mx){
				cout<<0<<endl;
				exit(0);
			}
			dfs(v);
		}
		else{
			if(vis[u]!=vis[v]){
				if(w[u]+w[v]!=s){
					cout<<0<<endl;
					exit(0);
				}
			}
			else{
				LL dis=s-(w[u]+w[v]);
				if(dis==0)continue;
				if(dis&1){//改變一定是偶數 
					cout<<0<<endl;
					exit(0);
				}
				if(vis[u]^(dis>0)){//相差的正負和可以改變的正負不同 
					cout<<0<<endl;
					exit(0);
				}
				if(dis<0)dis=-dis;
				mx=min(mx,dis/2);
				mn=max(mn,dis/2);
			}
		}
		
	}
}
int main(){Accel

	cin>>n>>m;
	REP(i,m){
		cin>>u>>v>>s;
		
		g[u].PB(MP(v,s));
		g[v].PB(MP(u,s));
	}
	// 設原點值0  維護原點變動範圍 
	mn=1,mx=1e18;
	vis.resize(n+1,-1);
	w.resize(n+1);
	vis[1]=1;
	dfs(1);
	//cout<<mn<<" "<<mx<<endl;
	cout<<max(0LL,mx-mn+1)<<endl;
}
```