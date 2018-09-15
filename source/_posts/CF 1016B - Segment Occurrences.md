---
title: CF 1016B - Segment Occurrences
tags:
  - Brute Force
categories:
  - ACM
date: 2018-08-11 14:54:59
---
太久沒寫了 廢掉了QQ
<!--more-->
# Problem
給你n,m,q分別代表String a,b長度和區間詢問數
求每筆區間詢問有幾個string a的substring等於string b
substring的l,r包含於詢問的l,r

# Input
* \\(1 \le n, m \le 10^3\\)
* \\(1 \le q \le 10^5\\)

# Solution
對每個n的位置當頭找符合數量
\\(O(n \cdot m)\\)預處理
partial sum 維護
\\(O(1)\\)詢問
總複雜度\\(O(n \cdot m + q)\\)

# Code
```cpp
int n,m,q;
string a,b;
int f[N];
int main(){Accel
    cin>>n>>m>>q;
   	cin>>a>>b;
   	for(int i=0;i<n-m+1;i++){
   		bool ok=1;
		REP(j,m)
			if(a[i+j]!=b[j]){ok=0;break;}
		f[i+1]=f[i]+ok;
	}
	for(int i=max(0,n-m+1);i<n;i++)f[i+1]=f[i];
	int l,r;
	REP(i,q){
		cin>>l>>r;
		r-=m-1;
		if(l<=r)
			cout<<f[r]-f[l-1]<<endl;
		else cout<<0<<endl;
	}
}
```