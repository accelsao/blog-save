---
title: Hexo note
date: 2018-06-25 20:54:00
comment: true
categories:
- Environment Setup
- Talks
tags:
- Hexo
- Mathjax
- Git
---
## 前言

終於...換了4個theme 一段時間都會用Material theme了吧...
建議想用hexo的新手 不要從很少人的theme開始..會問到一大堆問題 (M屬性例外

<!--more-->

我本來要更改圖片
單純hexo g s d 沒有變化

所以就用以下方法
如果沒clone過先clone
```
$ git clone https://github.com/[name]/[name].github.io.git .deploy/[name].github.io
$ cd .deploy/accelsao.github.io
$ hexo g d
```
為甚麼呢... 阿災



## Hexo with Mathjax

分享一下這篇[如何在 hexo 中支持 Mathjax？](https://ranmaosong.github.io/2017/11/29/hexo-support-mathjax/)
基本上我後面都在搞數學式子...最後 直接強制更換markdown 就可以直接用Mathjax 輕鬆多了 感謝大大!!!

## Hexo 備份

這也是挺重要的 因為github上沒有源碼 源碼消失 全部歸零 除非有辦法html轉md?

[Hexo 博客备份](https://blog.itswincer.com/posts/7efd2818/)
跟著這個準沒錯了

## 日常修改

```
$ git add .
$ git commit -m "upd"
$ git push origin hexo
$ hexo g -d # 原來有這種縮寫 我還一直用 && ...
```

## 補充 (2018/7/7)
今天在hexo d時居然卡超久 一開始還以為是太久沒deploy 結果 他就...一直那樣了
沒辦法只好google 結果google半天甚麼都找不到
心一橫 直接砍掉重練

最後又過了一天了..不過還好明天不用再用了

### 紀錄
不用.deploy那個爛方法 git push origin hexo 會瘋狂error
但以上的方法又解決了我一堆圖片沒辦法部署的問題 但最後大砍後也不需要.deploy
有需要還是得了解他到底幹了甚麼
以後要重練
直接把舊的檔案(還在的話) 備份下來
有6個必需的
1. package
2. _config
3. gitnore
4. source
5. themes
6. scafflods
7. node_modules(可不用 但因為我用mathjax有更動 所以我需要)

接著按照此篇 [Hexo 博客备份](https://blog.itswincer.com/posts/7efd2818/)
這篇寫得真的不錯
基本上就可以了

如果有像我圖片還是不行的
目前我還沒有結論
try again and again吧

注意新增repo後 有.git檔 這個不用刪掉

早知道我一開始就全砍掉處理了..浪費一推時間還不知道到底為甚麼會卡住...

注: 後來發現[Hexo 博客备份]的方法 themes/config 不會上傳github...要手動喔
