---
title: CSA - Firestarter
tags:
  - CS Academy
  - Graphs
  - Shortest Path
categories:
  - ACM
date: 2018-07-06 10:30:28
---



# Problem

給你有權圖G,N個點,M條邊
然後Q筆更新
給你T,A,B,X
代表時間T的時候 ,在AB邊上在距離A X單位的地方會開始燒
問你什麼時候這張圖會被燒完
火會往所有相鄰處燒 燒的速度1/s

<!--more-->

# Constraints and Notes
* \\(1\leq N \leq 10^{5}\\)
* \\(1\leq Q \leq 10^{5}\\)
* \\(1\leq M \leq 2*10^{5}\\)
* \\(1\leq L_i \leq 10^{9},1\leq i\leq M\\)
* \\(0\leq T_i \leq 10^{9},1\leq i\leq Q\\)
* \\(0\leq X_i \leq length\: of\: edge,1\leq i\leq Q\\)
* 圖是相連的
* 圖沒有重邊或是自環
* Q筆更新出現的邊保證一定屬於那M條邊



# Solution
這題還蠻有意思的 雖然code蠻長的
需要一點點腦洞 處理情況在邊上的問題
首先 更新點會出現在邊上
但邊長\\(10^{9}\\)無法處理
所以我們乾脆在這些**`起火點`**新增點
這樣就不會有在**邊上**的問題
那因為T秒後才開始燒所以在這之前有跟沒有一樣
開始燒之後 判斷燒完圖的方法就是看到所有地方的最大距離 這樣才燒的完
那我們在新增一個**`源點`**
這個**`源點`**連接所有**`起火點`**
而邊的距離就是他*開始燒以前所經過的時間*
那接下來就只要求所有點與點之間的最少所需時間
那就是圖上的最短路了
求完最短路後 題目所問的是圖燒完的時間
所以窮舉所有邊
答案至少是到A,B兩點之中最久的那個時間
再來就是燒中間的邊所需要的時間了
如果中間的再第二個點開始燒之前就燒完 所需時間就是第二個點到達時間
否則
還要在加上兩邊一起燒中間的邊的時間
答案=第二個點到達時間+(`總長`-`兩點開始燒的時間差`)/2 
因為兩邊等速燒所以直接除2

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
int n,m,q;
map<PII,int>mp;
vector<PII>g[N];
struct edge{LL v,cst;};
int main(){Accel

	// 建圖
    cin>>n>>m>>q;
    vector<int>u(m),v(m);
    vector<LL>l(m);
    REP(i,m){
        cin>>u[i]>>v[i]>>l[i];
        if(u[i]>v[i])swap(u[i],v[i]);
        mp[MP(u[i],v[i])]=i;
        mp[MP(v[i],u[i])]=i;
    }
    vector<vector<PLL>>f(m);
    vector<LL>t(n+q+1);
	// 起火點
    FOR(i,1,q){
        cin>>t[i+n];
        int a,b;
        LL d;
        cin>>a>>b>>d;
        int j=mp[MP(a,b)];
		//統一位置 
        if(a>b){
			swap(a,b);
			d=l[j]-d;
		}
		f[j].PB(MP(d,n+i));
        
    }
    int sz=n+q+1;
    vector<vector<edge>>g(sz);
    REP(i,m){
		// 源點 連接 起火點
		sort(ALL(f[i]));
		for(auto x:f[i]){
			g[0].PB(edge{x.S,t[x.S]});
		}
		if(SZ(f[i])==0){
			g[u[i]].PB(edge{v[i],l[i]});
			g[v[i]].PB(edge{u[i],l[i]});
		}
		else{
			//一條邊上可能不只一個起火點 統一位置後 直接左到右建圖
			int h=SZ(f[i]);
			REP(j,h-1){
				g[f[i][j].S].PB(edge{f[i][j+1].S,f[i][j+1].F-f[i][j].F});
				g[f[i][j+1].S].PB(edge{f[i][j].S,f[i][j+1].F-f[i][j].F});
			}
			g[u[i]].PB(edge{f[i][0].S,f[i][0].F});
			g[f[i][h-1].S].PB(edge{v[i],l[i]-f[i][h-1].F});
			g[f[i][0].S].PB(edge{u[i],f[i][0].F});
			g[v[i]].PB(edge{f[i][h-1].S,l[i]-f[i][h-1].F});
		}
	}
    //最短路
    vector<LL>dis(sz,1e18);
    dis[0]=0;
    priority_queue<PLL, vector<PLL>, greater<PLL> > qu;
    qu.push(MP(0,0));
    while(SZ(qu)){
		LL d=qu.top().F;
		int u=qu.top().S;
		qu.pop();
		if(dis[u]<d)continue;
		
		for(auto i:g[u]){
			int v=i.v;LL c=i.cst;
			if(dis[v]>dis[u]+c){
				dis[v]=dis[u]+c;
				qu.push(MP(dis[v],v));
			}
		}
		
	}
	//窮舉邊 還要考慮燒邊的時間
	double ans=0;
	FOR(u,1,sz-1){
		for(auto i:g[u]){
		    //源點到起火點的邊不用考慮
		    if(i.v==0)continue;
			LL x=dis[u],y=dis[i.v],cst=i.cst;
			
			if(x>y)swap(x,y);
			ans=max(ans,1.0*y);
			if(cst>y-x){
				ans=max(ans,1.0*y+0.5*(cst-(y-x)));
			}
		}
	}
   	cout<<fixed<<setprecision(1)<<ans<<endl;
}
```