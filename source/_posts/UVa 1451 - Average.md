---
title: UVa 1451 - Average
date: 2018-06-25 10:14:00
categories: ACM
tags:
- Slope Optimization
- DP
- Geometry
- UVa
---

## Problem
Given a binary string and a length lower bound L , find a subsequence
of the string whose length is at least L and whose averge is maximum

Average of binary string is the number of one devided by length

<!--more-->

## Input
* \\(1\leq n \leq 10^{5}\\) , length of string
* \\(1\leq L \leq 1000\\) , substring length lower bound

## Output
Print the starting and ending index of the subsequence
If two or more subsequences have the maximum average ,then find the shortest one; and if two or more shortest subsequences with the maximum average exist, then find the one with the smallest starting index.



## Solution
precompute the perfix of the string, make n point (i,s[i]) where s[i] is \\(\sum_{1}^{i}one's\\).  Now the problem is two find the max slope between two point and the interval of them on X-axis

Time Complexity: O(n)

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
int T;
int a[N];
int q[N];
int s,t;
int n,len;
double get(int l,int r){
	return 1.0*(a[r]-a[l])/(r-l);
}
double ans;
int l,r;
void solve(){
	ans=1.0*a[n]/n;
	s=0,t=-1;
	l=0,r=n;
	FOR(i,len,n){
		int tmp=i-len;
		while(s<t&&get(q[t],tmp)<=get(q[t-1],q[t]))t--;
		q[++t]=tmp;
		while(s<t&&get(q[s],i)<=get(q[s+1],i))s++;
		double tp=get(q[s],i);
		if(tp>ans){
			ans=tp;
			l=q[s],r=i;
		}
		else if(tp==ans&&i-q[s]<r-l){
			l=q[s],r=i;
		}
	}
}
int main(){Accel
	cin>>T;
	while(T--){
		cin>>n>>len;
		string str;
		cin>>str;
		
		a[0]=0;
		FOR(i,1,n){
			a[i]=a[i-1]+str[i-1]-'0';
		}
		solve();
		
		cout<<l+1<<" "<<r<<endl;
		
	}
}
```
