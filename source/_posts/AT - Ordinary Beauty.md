---
title: AT - Ordinary Beauty
tags:
  - Probabilities
  - AtCoder
categories:
  - ACM
date: 2018-07-09 17:22:08
---

一個簡單的題目但我卡很久因為我超廢
測資有假解可以過 ㄏㄏ

# Problem

定義the *beauty* of the sequence(\\(a_1\\),...\\(a_m\\)) 是其中任意兩個相鄰的pair差距是d的個數
Example,d=1時 (3,2,3,10,9) 是3
給你n,m,d
\\(1\leq a_i \leq n\\)
在\\(n^{m}\\)種排列裡面求出sequence的平均*beauty*

簡單說 ans=\\(\frac{\sum beauty}{n^{m}}\\)

# Contraints
* \\(0\leq d<n \leq 10^{9}\\)
* \\(2 \leq m \leq 10^{9}\\)
* 都是整數

# Solution
考慮兩個相鄰的數
符合條件的可能為(a,a+d)且\\(1\leq a < a+d \leq n\\)
這樣的可能有 n - d個 而且當d不等於0的時候 (a+d,a) 也是符合的因此要double
在長m的sequence 同一個(a,a+d) 有m-1種位置的可能 剩下的我們不管他 所以有 \\(n^{m-2}\\)種情況

因為當我們只考慮某個pair的貢獻的時候 其他的pair不會被算入其中
Example:(1,2,3,4)當d=1時候
考慮(1,2) 這個sequence貢獻有1
考慮(2,3) 這個sequence貢獻有1
考慮(3,4) 這個sequence貢獻有1
考慮所有 這個sequence貢獻有1+1+1=3
所以分別考慮不同pair的時候 直接加總/\\(n^{m}\\)即可

總結:
When d is not 0
考慮pair個數有(n-d)*2種
有m-1位置和剩餘的\\(n^{m-2}\\)種其他東西的排列

\begin{equation} \label{eq1}
\begin{split}
ans & = \frac {2\times(n-d)\times(m-1)\times n^{m-2}}{n^{m}} \\
 & = \frac {2\times(n-d)\times(m-1)}{n^{2}} \\
\end{split}
\end{equation}

when d is 0

\begin{equation} \label{eq2}
\begin{split}
ans & = \frac {n\times(m-1)\times n^{m-2}}{n^{m}} \\
 & = \frac {m-1}{n} \\
\end{split}
\end{equation}

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
LL n,m,d;
int main(){Accel
	cin>>n>>m>>d;

	double ans=n-d;
	if(d)ans*=2;
	ans*=m-1;
	ans/=n;
	ans/=n;
	cout<<fixed<<setprecision(12)<<ans<<endl;
}
```