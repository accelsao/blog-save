---
title: UVa 12524 - Arranging Heaps
layout: post
date: 2018-06-24 10:29:14
comment: true
categories: ACM
tags:
- DP
- Slope Optimization
- Convex hull
- UVa

---
## Problem
N堆東西要合併成K堆
每個東西有X,W 位置和重量
移動到Y的cost=W*(Y-X)
輸出最小的cost
## Input
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

* \\(1\leq N,K\leq10^{3}\\)
* \\(1\leq X,W\leq10^{6}\\)
*  X Strictly Increasing

<!--more-->

## Solution
首先明顯DP
窮舉放入的東西 1~i
定義: f[i][j]=放到第i個東西時 有j堆的 Cost
f[i][j]=min { f[k][j-1]+cost(k,i) } , 1<=k<i
cost(k,i)=k+1~i-1 的東西全部放到\\(X_i\\)的代價

<a href="https://www.codecogs.com/eqnedit.php?latex=cost(k,i)=\sum_{k&plus;1}^{i-1}&space;(X_i-X_j)*W_j" target="_blank"><img src="https://latex.codecogs.com/gif.latex?cost(k,i)=\sum_{k&plus;1}^{i-1}&space;(X_i-X_j)*W_j" title="cost(k,i)=\sum_{k+1}^{i-1} (X_i-X_j)*W_j" /></a>
因為 <a href="https://www.codecogs.com/eqnedit.php?latex=X_i-X_i=0" target="_blank"><img src="https://latex.codecogs.com/gif.latex?X_i-X_i=0" title="X_i-X_i=0" /></a>
<a href="https://www.codecogs.com/eqnedit.php?latex=cost(k,i)=\sum_{k&plus;1}^{i}&space;(X_i-X_j)*W_j" target="_blank"><img src="https://latex.codecogs.com/gif.latex?cost(k,i)=\sum_{k&plus;1}^{i}&space;(X_i-X_j)*W_j" title="cost(k,i)=\sum_{k+1}^{i} (X_i-X_j)*W_j" /></a>

將cost(k,i)展開

<a href="https://www.codecogs.com/eqnedit.php?latex=cost(k,i)=W_{i}*(X_i-X_{i})&plus;W_{i-1}*(X_i-X_{i-1})&plus;...&plus;W_{k&plus;1}*(X_i-X_{k&plus;1})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?cost(k,i)=W_{i}*(X_i-X_{i})&plus;W_{i-1}*(X_i-X_{i-1})&plus;...&plus;W_{k&plus;1}*(X_i-X_{k&plus;1})" title="cost(k,i)=W_{i}*(X_i-X_{i})+W_{i-1}*(X_i-X_{i-1})+...+W_{k+1}*(X_i-X_{k+1})" /></a>


定義:
<a href="https://www.codecogs.com/eqnedit.php?latex=sW_i=\sum_{1}^{i}W_i" target="_blank"><img src="https://latex.codecogs.com/gif.latex?sW_i=\sum_{1}^{i}W_i" title="sW_i=\sum_{1}^{i}W_i" /></a>
<a href="https://www.codecogs.com/eqnedit.php?latex=sH_i=\sum_{1}^{i}W_i*X_i" target="_blank"><img src="https://latex.codecogs.com/gif.latex?sH_i=\sum_{1}^{i}W_i*X_i" title="sH_i=\sum_{1}^{i}W_i*X_i" /></a>
<a href="https://www.codecogs.com/eqnedit.php?latex=cost(k,i)=X_i*(sW_i-sW_k)-(sH_i-sH_k)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?cost(k,i)=X_i*(sW_i-sW_k)-(sH_i-sH_k)" title="cost(k,i)=X_i*(sW_i-sW_k)-(sH_i-sH_k)" /></a>
<a href="https://www.codecogs.com/eqnedit.php?latex=f[i][j]=min{f[k][j-1]&plus;X_i*(sW_i-sW_k)-(sH_i-sH_k)}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?f[i][j]=min{f[k][j-1]&plus;X_i*(sW_i-sW_k)-(sH_i-sH_k)}" title="f[i][j]=min{f[k][j-1]+X_i*(sW_i-sW_k)-(sH_i-sH_k)}" /></a>
<a href="https://www.codecogs.com/eqnedit.php?latex=f[k][j-1]&plus;sH_k=X_i*sW_k&plus;sH_i-X_i*sW_i&plus;f[i][j]" target="_blank"><img src="https://latex.codecogs.com/gif.latex?f[k][j-1]&plus;sH_k=X_i*sW_k&plus;sH_i-X_i*sW_i&plus;f[i][j]" title="f[k][j-1]+sH_k=X_i*sW_k+sH_i-X_i*sW_i+f[i][j]" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=y=f[k][j-1]&plus;sH_k,m=X_i,x=sW_k,b=sH_i-X_i*sW_i&plus;f[i][j]" target="_blank"><img src="https://latex.codecogs.com/gif.latex?y=f[k][j-1]&plus;sH_k,m=X_i,x=sW_k,b=sH_i-X_i*sW_i&plus;f[i][j]" title="y=f[k][j-1]+sH_k,m=X_i,x=sW_k,b=sH_i-X_i*sW_i+f[i][j]" /></a>
符合 <a href="https://www.codecogs.com/eqnedit.php?latex=y=m*x&plus;b" target="_blank"><img src="https://latex.codecogs.com/gif.latex?y=m*x&plus;b" title="y=m*x+b" /></a>  可以維護斜率
m是嚴格遞增 斜率上升 維護下凸包
要求b要小

## Code
```cpp
#include <bits/stdc++.h>
#define ld long double
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
#define endl "\n" 
using namespace std;
//typedef complex<double>C;
//const double PI(acos(-1.0));
//const double eps(1e-8);
//for(int mask=i;mask>0;mask=(mask-1)&i)
//int a[25]= { 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97 };
//__gcd, atan2(y,x)=y/x , __int128
//c(n,k)=c(n-1,k-1)+c(n-1,k)
//c(i,r),i for r to n =c(n+1,r+1)
//void add(int x){while(x<N)BIT[x]++,x+=x&-x;}
//int sum(int x){int s=0;while(x){s+=BIT[x];x-=x&-x;}return s;}
//int find(int x){return x==p[x]?x:p[x]=find(p[x]);}
//LL pw(LL a,LL b){LL t=1;for(;b;b>>=1,a=a*a%M)b&1?t=t*a%M:0;return t;}
//log() = ln() , log(x)/log(y)=log(y)-base-x
//INT_MAX 127,INT_MIN 128
//int dw[4][2]={{0,1},{1,0},{0,-1},{-1,0}};
//int dw[8][2]={{0,1},{1,0},{0,-1},{-1,0},{1,1},{1,-1},{-1,1},{-1,-1}};
//cout<<fixed<<setprecision(12)<<ans<<endl;
//__builtin_popcount(mask)
const int N=1e3+10;
const LL INF=1e18;
const int M=1e9+7;
int n,k;
LL p[N],W[N];
LL sW[N],sH[N];
LL f[N][N];
struct SlopeDP{
	int s,t;
	LL X[N],Y[N];
	void init(){
		s=0,t=-1;
	}
	//下凸包 叉積<0 不優 拿掉
	LL cross(LL x1,LL y1,LL x2,LL y2){
		return 	x1*y2-x2*y1;
	}
	void add(LL x,LL y){
		while(s<t&&cross(X[t]-X[t-1],Y[t]-Y[t-1],x-X[t],y-Y[t])<0)
			t--;
		t++;
		X[t]=x,Y[t]=y;
	}
	LL get(LL m){
		while(s<t&&Y[s+1]-Y[s]-m*(X[s+1]-X[s])<0)s++;
		return Y[s]-m*X[s];
	}
}dp;

int main(){Accel

	while(cin>>n>>k){
		FOR(i,0,n)FOR(j,0,k)f[i][j]=1e18;
		//cout<<1<<endl;
		f[0][0]=0;
		FOR(i,1,n){
			cin>>p[i];
			cin>>W[i];
			sW[i]=sW[i-1]+W[i];
			sH[i]=sH[i-1]+p[i]*W[i];
		}
		/*
			f[i][j]=f[k][j-1]+p[i]*(sW[i]-sW[k])-(sH[i]-sH[k])
			
			f[k][j-1]+sH[k]=p[i]*sW[k]+f[i][j]+sH[i]-p[i]*sW[i]
			y              =m   *x    +b
			
		*/
		FOR(j,1,k){//從j堆DP 
			dp.init();
			FOR(i,j,n){
				dp.add(sW[i-1],f[i-1][j-1]+sH[i-1]);
				LL val=dp.get(p[i]);
				f[i][j]=val-(sH[i]-p[i]*sW[i]);
				
			}
			
		}
		cout<<f[n][k]<<endl;
	}
}
```
