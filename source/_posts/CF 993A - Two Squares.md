
---
title: CF 993A - Two Squares
tags:
  - Codeforces
  - Geometry
categories:
  - ACM
date: 2018-06-27 15:28:51
---
## Problem
You are given two squares, one with sides parallel to the coordinate axes, and another one with sides at 45 degrees to the coordinate axes. Find whether the two squares intersect.

## Input
Two lines each containg 4 coordinates are either in clockwise or counterclockwise order
All values are integer between -100~100

<!--more-->

##  Solution
**Since the value are between -100~100**
Just check all the point !!!

To check whether the point is in A , which is parallel to the axes just find the leftmost,rightmost,topmost,bottommost point

Then for B, check whether the distance to the center of it , is \\(\leq\\) the corner to the center

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
const int N=1e5+10;
const int M=1e9+7;
const int inf=1e9+7;
const LL INF=1e18;
const double eps=1e-6;
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
int n,k;
struct Sqr{
	int x[4],y[4];
	int lx,tx;
	int ly,ty;
	bool contain(int x,int y){
		return x>=lx&&x<=tx&&y>=ly&&y<=ty;
	}
}A,B;
int cx,cy;
int dis(int x,int y){
	return abs(x-cx)+abs(y-cy);
}
int main(){Accel
	REP(i,4)
		cin>>A.x[i]>>A.y[i];
	
	REP(i,4)
		cin>>B.x[i]>>B.y[i];
	REP(i,4){
		A.lx=A.x[0];
		for(int j=1;j<4;j++)A.lx=min(A.lx,A.x[j]);
		A.tx=A.x[0];
		for(int j=1;j<4;j++)A.tx=max(A.tx,A.x[j]);
		A.ly=A.y[0];
		for(int j=1;j<4;j++)A.ly=min(A.ly,A.y[j]);
		A.ty=A.y[0];
		for(int j=1;j<4;j++)A.ty=max(A.ty,A.y[j]);
	}
	REP(i,4){
		cx+=B.x[i],cy+=B.y[i];
	}
	cx/=4,cy/=4;
	
	
	FOR(i,-100,100)
		FOR(j,-100,100)
			if(A.contain(i,j)&&dis(i,j)<=dis(B.x[0],B.y[0])){
				cout<<"YES\n";
				return 0;
			}
	cout<<"NO\n";
	
	
}
```



