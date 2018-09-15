---
title: UVa 10594 - Data Flow
date: 2018-06-25 17:25:15
comment: true
categories:
- ACM
tags:
- UVa
- Flow
---
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
## Problem and Solution
Find the min cost max flow the network
## Input
* \\(2\leq N \leq 100\\), N nodes
* \\(1\leq M \leq 5000\\), M edges
* D , data to send
* K , limit for every edges 

##  Output
Minumum possible time, otherwise print "Impossible."
<!--more-->

## Min cost max flow
For this problem
we need the find mcmf (s=0,t=n)
Add edge 0->1 with cap D , because we need to send D date to n
Add the left  with cap K ,the limit

## Code
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
const int N=1e3+1;
const int M=1e9+7;
const int inf=1e9+7;
const double eps=1e-6;
const LL INF=1e18;
typedef complex<double>C;
const double PI(acos(-1.0));
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
int n,m;
LL D,K;
struct Edge{
	int from,to,v,cap,flow;
	LL cost;
	Edge(int u,int v,int c,int f,LL w) {  
        from = u;  
        to = v;  
        cap = c;  
        flow = f;  
        cost = w;  
    }  
};
LL flw,cst;
struct MCMF{
	vector<Edge>E;
	vector<int>g[N];
	queue<int>q;
	int p[N],a[N];
	void init(int n){
		FOR(i,0,n+1)g[i].clear();
		E.clear();
	}
	int sz;
	void add(int u,int v,int cap,LL cst){
		E.PB(Edge(u,v,cap,0,cst));
		E.PB(Edge(v,u,0,0,-cst));
		sz=SZ(E);
		g[u].PB(sz-2);
		g[v].PB(sz-1);
	}
	int d[N],inq[N];
	bool bellmanford(int s,int t){
		FOR(i,0,n+1)d[i]=inf,a[i]=0,p[i]=0;
		d[s]=0;
		a[s]=inf;
		q.push(s);
		while(SZ(q)){
			int u=q.front();q.pop();
			inq[u]=0;
			for(auto v:g[u]){
				Edge &e=E[v];
				if(e.cap>e.flow&&d[e.to]>d[u]+e.cost){
					d[e.to]=d[u]+e.cost;
					p[e.to]=v;
					a[e.to]=min(a[u],e.cap-e.flow);
					if(!inq[e.to])
						q.push(e.to),inq[e.to]=1;
				}
			}
		}
		
		if(d[t]==inf){
			return false;
		}
		flw+=a[t];
		cst+=1LL*a[t]*d[t];
		for(int u=t;u!=s;u=E[p[u]].from){
			E[p[u]].flow+=a[t];
			E[p[u]^1].flow-=a[t];
		}
		return 1;
	}
	LL mcmf(int s,int t){
		cst=flw=0;
		while(bellmanford(s,t));
		return flw;
		
	}
}B;
struct P{
	int u,v;LL c;
};
int main(){Accel
	while(cin>>n>>m){
		B.init(n);
		vector<P>p(m);
		REP(i,m){
			cin>>p[i].u>>p[i].v>>p[i].c;
		}
		cin>>D>>K;
		REP(i,m){
			
			B.add(p[i].u,p[i].v,K,p[i].c);
			B.add(p[i].v,p[i].u,K,p[i].c);
		}
		B.add(0,1,D,0);
		int ans=B.mcmf(0,n);
		if(ans==D)cout<<cst<<endl;
		else cout<<"Impossible.\n";
	
	}
}
```
