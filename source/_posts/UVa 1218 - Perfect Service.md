---
title: UVa 1218 - Perfect Service
tags:
  - UVa
  - DP
  - Trees
categories:
  - ACM
date: 2018-07-06 14:16:09
---


# Problem
有N台電腦互相連接,考慮將其中一些電腦做為服務器,要求每台非服務器的電腦必須連接一個服務器且只能一個,問最少需要幾台做為服務器

<!--more-->

# Input
* \\(N\leq 10^{4}\\)




# Solution
最不拿手的DP問題
而且還是樹DP.. 再多練練吧

問題等價
給你一棵樹 求塗最少的點為黑色使得這張圖 白色的點都只跟一個黑色相鄰

### 討論情況
有以下幾種可能

先假設
U點為`目前`
V點為`孩子`
P點為`父親`

對於每個點有4種情況

* U為黑 P為白
* U為黑 P為黑
* U為白 P為白
* U為白 P為黑

### 定義狀態

當U為黑 V點可為黑也可為白 跟P無關
F(u,0)

當U為白 P為黑 V點只能為白 否則U不合
F(u,1)

當U為白 P為白 V點只能為黑且只有一個能為黑 否則U點不合
F(u,2)

### 狀態初始
轉移的方式 每次考慮的他V點們
所以由下到上
然後初始
F(u,0)=1 黑點所以 1
F(u,1),F(u,2) 沒有黑點 所以0

因為實作方便F(u,2)先給inf

### 狀態轉移

因為此點為黑 因此F(v,2)不合
F(u,0) = 1 + min( F(v,0) , F(v,1) )

因為V點只能為白且U點為白 F(v,1)不合
F(u,1) = sum( F(v,2) )

只能取一個V點為黑
注意F(u,1)把所有F(v,2)白色的情況加起來
那麼再把其中一個點扣掉換成黑色 就達成了只有一個點為黑的情況
F(u,2)=F(u,1)-F(v,2)+F(v,0)
實作的時候因為F(u,1)還沒算完 可以先求min(-F(v,2)+F(v,0))
之後再加回來

### 答案
min ( F(1,0) , F(1,2) )
F(1,1)的情況 1和1的孩子都不是服務器 不合 


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
int n;
vector<int>g[N];
int f[N][3];
//0 他是服務器  //都可 
//1 他不是服務器 但父親是    // 孩子必須不是 
//2                   不是   // 孩子必須是  
void dfs(int u,int p){
	
	f[u][0]=1;
	f[u][1]=0;
	f[u][2]=N;
	for(auto v:g[u])
		if(v!=p){
			dfs(v,u);
			f[u][0]+=min(f[v][0],f[v][1]);
			f[u][1]+=f[v][2];
			// 維護最小值 
			// 假設這點換成黑色的值 
			// f[u][1] 會把所有 f[v][2] 算上去 所以扣掉f[v][2] 表示把這點排除 ,在加回黑色的可能 f[v][0]
			f[u][2]=min(f[u][2],f[v][0]-f[v][2]);
		}
	//把都其他白色的可能數加回來 
	f[u][2]+=f[u][1];

	
}
int main(){Accel
	while(cin>>n){
		FOR(i,1,n)g[i].clear();
		int u,v;
		REP(i,n-1){
			cin>>u>>v;
			g[u].PB(v);
			g[v].PB(u);
		}
		dfs(1,-1);
		// f[1][1]的情況 1和1的孩子都不是服務器 不合 
		cout<<min(f[1][0],f[1][2])<<endl;
		cin>>n;
		if(n==-1)break;
	}
}
```

