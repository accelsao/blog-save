---
title: CF 1016C - Vasya And The Mushrooms
tags:
- DP
categories:
- ACM
date: 2018-08-11 19:42:33
---
未醒...
<!--more-->
# Problem
給你兩行數字代表蘑菇的重量
行走各個位置**剛好一次**
每次只能行進相鄰一格,剛好一次且不能走出去,也不能停下來
從0開始 找出能夠獲得最大權重的走法

# Input
* length of the glade \\(1 \le n \le 3 \cdot 10^5\\)
* growth rate \\(1\le gr\le 10^6\\)

# Solution

走法可能
![](http://codeforces.com/predownloaded/22/b3/22b3554d78d55966d7f18638de45e29d5ee2d754.png)
求出向右走的\\(\sum_{k=i}^n a_i \cdot k\\)
求出向左走的\\(\sum_{k=i}^n a_i \cdot (n-k+1)\\)
還有suffix\\(\sum_{k=i}^n a_i\\)

# Code
```cpp
LL a[2][N];
int n;
LL f[2][N];//front
LL b[2][N];//back
LL p[2][N];//suffix

int main(){Accel
	cin>>n;
	for(int i=0;i<n;i++)
		cin>>a[0][i];
	for(int i=0;i<n;i++)
		cin>>a[1][i];
	LL lt=0;
	for(int i=0;i<2;i++){
		for(int j=n-1;j>=0;j--){
			f[i][j]=f[i][j+1]+(j+1)*a[i][j];
			b[i][j]=b[i][j+1]+(n-j)*a[i][j];
			p[i][j]=p[i][j+1]+a[i][j];
			lt+=a[i][j];
		}
		
	}
	
	LL tmp=0,ans=0;
	for(int i=0,j=0;j<n;j++,i^=1){
		LL cur=tmp;
		cur+=f[i][j]+j*p[i][j];
		cur+=b[i^1][j]+(j+n)*p[i^1][j];
		ans=max(ans,cur);
		//每輪依序加上(1,2),(3,4),..(n-1,n)
		tmp+=a[i][j]*(j+j+1);
		tmp+=a[i^1][j]*(j+j+2);
		
	}
	//因為前面1-index處理 所以再扣掉多的
	cout<<ans-lt<<endl;
	
}
```
