---
title: CSA - Restricted Arrays
date: 2018-06-26 16:17:16
categories:
- ACM
tags:
- CS Academy
- Combinatorial
---
## Problem
You are given an array *A* of size *N* and an integer *K*. How many arrays *B*  can you build such that:
* \\(max(B_i)\\) is minmum
* \\(B_i\\)>0
* for any two indices *i* and *j*,\\(|i-j|<K:B_i=B_j\: if A_i=A_j\: else B_i\neq B_j\\)

<!--more-->

## Input 
* \\(1\leq K\leq N\leq 10^{5}\\)
* \\(1\leq A_i\leq N\\)

## Output 
the numbers of arrays *B* that satisfy the problem, and an modulo \\(10^{9}+7\\)



## Solution
the constraint meet only for every k consecutive indices  *i-k+1~i*
We count the unique numbers from *i-k+1~i*, define it *c[i]*  
Then find the maximum,define it Mx, and Mx is the max number for *B* arrays  
Now we can count the possible \\(B_i\\)  
Maintain segement with length *K*  
We count only if it is the **first occurence number** in the segment ,because others must be the same as the first one  
Then , the possible ways is Mx-c[i]+1 or Mx-(c[i]-1)  

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
int n,k;
int c[N];
int a[N];
int d[N][2];//the different number in i-k+1~i,and whether the first occurance or not
int s;
void add(int x,int v){
	if(!c[x])s++;
	c[x]+=v;
	if(!c[x])s--;
}
int main(){Accel
	cin>>n>>k;
	int x;
	int t=0;
	FOR(i,1,n){
		cin>>a[i];
		add(a[i],1);
		d[i][0]=s;
		d[i][1]=(c[a[i]]==1);
		t=max(t,s);
		if(i>=k){
			add(a[i-k+1],-1);
		}
	}
	
	LL ans=1;
	FOR(i,1,n){
		if(d[i][1])
			ans=ans*(t-d[i][0]+1)%M;
	}
	cout<<ans<<endl;
	
}
```