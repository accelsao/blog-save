---
title: CSA - City Break
date: 2018-06-26 14:55:35
categories:
- ACM
tags:
- CS Academy 
- Two Pointer
---

## Problem
There are N cities on a circle,you know the distance between any two consecutive cities.  
You start in the city with index S, at every step you move to the closest city that was not previously visited  
(in case there is more than one closest city you choose the one with minimum index).  
What’s the distance you cover until you visit all the N cities?  

<!--more-->

## Input
* \\(1\leq N \leq 10^{5}\\)
* \\(1\leq S \leq N\\)
* The distance between any 2 consecutive cities is an integer between 1 and \\(10^{​7}\\)
* The sum of distances is \\(\leq 10^{8}\\)

## Output
The minmum distance to visit N cities



## Solution
Every step you have to find an unvisited city.  
To find the min distance, there is no point to go pass the unvisited city.  
So maintain the possible left and right city  until you find all cities  

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
const int N=1e6+1;
const int M=1e9+7;
const int inf=1e9+7;
const int INF=1e18;
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
int n,s;
int d[N];
int a[N];
int get(int u,int v){
	int dis=0;
	if(u>v)swap(u,v);

	dis=a[v]-a[u];
	dis=min(dis,a[n]-dis);
	return dis;
}
int main(){Accel
	cin>>n>>s;
	s--;
	REP(i,n){
		cin>>d[i];
		a[i+1]=a[i]+d[i];		
	}
	int l=(s-1+n)%n;
	int r=(s+1)%n;
	LL tot=0;
	REP(tm,n-1){
		int d1=get(s,l);
		int d2=get(s,r);
		if(d1<d2||d1==d2&&l<r){
			tot+=d1;
			s=l;
			l=(l-1+n)%n;
		}
		else{
			tot+=d2;
			s=r;
			r++;r%=n;
		}
	}
	cout<<tot<<endl;
	

		
}
/*
4 1
1 10 1 1
*/
```