---
title: AT - Saving Snuuk
tags:
  - Shortest Path
  - AtCoder
  - Graphs
categories:
  - ACM
date: 2018-07-10 10:58:26
---

題目有狗難看懂的啦...看懂後就結束了

# Problem


給你一張圖u-v邊有a和b
你要從s到t 然後每次經過邊需要花一些錢
有兩種貨幣
a代表第一種貨幣的花費 ,b是另外一種
一開始你有\\(10^{15}\\)的a貨幣
對於點i 在i-1 years前 才可以交換錢 之後都不能交換
交換時候當你有X個a你就要全換成X個b

重點問題 問你在[0,n-1]years的**不同時間點**所需要的花費
所以當你0year出發 點1~n都可以交換
但當你n-1year出發 只有點n可以交換 
總共有n個答案

# Constraints
* \\(2\leq n \leq 10^{5}\\)
* \\(1\leq m \leq 10^{5}\\)
* \\(1\leq s,t \leq n\\)
* \\(s\neq t\\)
* \\(1\leq u_i<v_i \leq n\\)
* \\(1\leq a_i<b_i \leq 10^{9}\\)
* if \\(i\neq j\\),then \\(u_i\neq u_j\\) or \\(v_i\neq v_j\\)
* connected graph
* intergers

# Solution
題目看懂就差不多了
注意因為交換是全額
現在假設我們在i處交換
s-t所花費就是s-i用a,i-t用b
i-t所花費又等於t-i的花費
那直接對s,t找到其他點的最短路
然後窮舉交換的地點就可以求解了
技巧就是從最後一個開始求最佳解
因為第iyear能交換的地方,i-1year都能交換(再多一個i點而已)

# Code
```py
n, m, s, t = map(int, input().split())
s-=1
t-=1
G = [[] for i in range(n)]
for i in range(m):
    u,v,a,b=map(int,input().split())
    G[u-1].append((v-1,a,b))
    G[v-1].append((u-1,a,b))
 
V=10**15
ans=[0]*n
from heapq import heappush,heappop
def dijkstra(u,k):
    D=[10**18]*n
    q=[(0,u)]
    D[u]=0
    while q:
        d,u=heappop(q)
        if D[u]<d: continue
        for v ,a ,b in G[u]:
            w=b if k else a
            if d + w < D[v]:
                D[v]=d+w
                heappush(q,(d+w,v))
    return D
D1=dijkstra(s,0)
D2=dijkstra(t,1)
tmp=1e18
for i in reversed(range(n)):
    tmp=min(tmp,D1[i]+D2[i])
    ans[i]=10**15-tmp
print(*ans,sep="\n")
```