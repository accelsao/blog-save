
---
title: 當Github Pages遇上Hexo
date: 2018-06-23 22:39:51
categories:
- Environment Setup
tags:
- Hexo
- Github Pages
---
## 前言
做為Morris大大的小粉絲
早就想寫個博客 結果因為有點怠惰...
總之第一篇就來抱怨一下生這個Blog的過程

本來要使用jekyll 來玩 結果搞半天 jekyll serve 不能用中文 utf-8 error的甚麼鬼(雖然後來猜測是markdown的編碼錯誤)

找很久也解決不了就果斷放棄 後來問了morris 發現hexo這神器 至少不用裝ruby方便很多了

先說一下:
每個Theme需求都不同 所以最好以Theme的Readme.md為主


<!--more-->

## 預備
* Nodejs
* Git (我都用 git bash)


## Hexo 安裝
在Node.js的cmd 輸入
```
$ npm install hexo-cli -g
```
新建Blog
$ hexo init < hexo-folder>
(以下以blog為例)
```
$ hexo init blog
$ cd blog
$ npm install
```
## 其他指令
```
$ hexo g # 或者hexo generate 生成
$ hexo s # 或者hexo server 可以在http://localhost:4000/ 查看
$ hexo new <filename> # 新增 .md檔
$ hexo new page "pageName" # 新增頁面
```
## 常用縮寫
```
$ hexo n == hexo new
$ hexo g == hexo generate
$ hexo s == hexo server
$ hexo d == hexo deploy
```
## Hexo Theme
以下用 https://github.com/iissnan/hexo-theme-next 為例
可以參考他的 Readme

```
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
Theme用的字參考每個Theme的Readme
```
或下載 解壓縮到目錄
也是放在< hexo-file>/themes 裡面

修改blog目錄下的_config.yml的theme:

 .md or  .yml 可裝Notepad++使用

改成 theme: next
## 更新Theme
```
cd themes/next #除了landscape以外的那個就對了
$ git pull
$ hexo g 
$ hexo s 
```

## Github Pages 設置
Step 1:

直接New一個repo  名稱是 [username].github.io 就行了

Step 2:

**Deploy**到github上
一樣修改hexo目錄下的 _config.yml
```
deploy:
  type: git
  repo: git@github.com: [username]/ [username].github.io.git
  branch: master
```
repo 這裡可以從 Github repo頁面的 Clone or Download 那個綠色按鈕
clone with ssh 的那一串就是

Step 3:
```
$ hexo d 
```

## 注意
1. 如果出現 Error deployer not found:git 需要
 ```
$ npm install hexo-deployer-git --save
```
2. 如果有出現這個訊息
```
Permission denied (publickey).
fatal: Could not read from remote repository.
Please make sure you have the correct access rights
and the repository exists.
```
要設置public key [github help](https://help.github.com/articles/connecting-to-github-with-ssh/)

## Deploy 問題
有時候 `$ hexo d` 會出現問題
因此 使用git來完成deploy
```
cd <hexo-file>
到你的hexo file 路徑
$ git clone https://github.com/[name]/[name].github.io.git .deploy/[name].github.io
新建一個目錄叫做.deploy 放clone的code
```
以上第一次做就好
下面的每次deploy都要做
```
hexo generate
cp -R public/* .deploy/[name].github.io
cd .deploy/[name].github.io
git add .
git commit -m “update”
git push origin master
```

## 小筆記
* 用Notepad++ 寫md的時候 用TAB會出現紅字 可能會出錯 不知道原因
* `$ hexo s`變化很快
* `$ hexo d`需要點時間 不要誤以為是bug
* Notepad++ 編碼要先改成UTF-8 才不會出現亂碼
* 以後變動repo 直接 `$ hexo g && hexo s` 最後 `$ hexo d`上 github
* tags 冒號後面要空格

## 參考網站

* [# 手把手教你使用Hexo + Github Pages搭建个人独立博客](https://linghucong.js.org/2016/04/15/2016-04-15-hexo-github-pages-blog/)
* [Hexo Themes](https://hexo.io/themes/index.html)
* [$ hexo s 查看連結](http://localhost:4000/)
* [StackEdit In-browser Markdown editor](https://stackedit.io)

