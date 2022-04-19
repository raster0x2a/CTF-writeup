---
title: "CakeCTF 2021 Writeup"
draft: false
---

# CakeCTF 2021 Writeup

注意: 書きかけです

## 目次

- Pwn
  - [No Step On Snek](#no-step-on-snek)
  - [Jellyspotters](#jellyspotters)
  - [MDL Considered Harmful](#mdl-considered-harmful)

## MofuMofu Diary - 110

### 問題

[Would you like to see some mofu-mofu pictures?](http://web.cakectf.com:8003/)  
\* The flag is located at `/flag.txt`  
[mofumofu_diary_82c31e779086e3079e0ac62643e14fed.tar.gz(を解凍したもの)](https://github.com/raster0x2a/CTF-writeup/blob/master/CakeCTF2021/mofumofu_diary_82c31e779086e3079e0ac62643e14fed)  
  
author: ptr-yudai

### 解説

index.phpとutil.phpを読むと、だいたいの動きとしてCookieの`cache`中に`data`と`expiry`があり、`data`の中には画像ファイル名である`name`、画像の説明である`description`があり、`expiry`の日付が一週間以上前の場合は再度Cookie情報からファイルの再読み込みをして表示しています。  
重要な部分はCookie情報からファイルの再読み込みをしていることで、Cookieを編集することで自由なファイルを読み込ませることができる点です。  
再読み込みさせるために`expiry`を適当に1週間以上前の値にし、`name`が`/flag.txt`になるデータを作ります。例えばChromeの開発者モードのApplicationタブで以下のようなものをパーセントエンコーディングして`cache`に入れます。  
`{"data":[{"name":"\/flag.txt","description":"flag"}],"expiry":1530722670}`  
リロードするとbase64エンコードされたフラグが手に入るのでデコードして終了。

## UAF4b - 113

### 問題

warmuppwn
You don't dare to try learning Use-after-Free?  
`nc pwn.cakectf.com 9001`  

author: ptr-yudai  

### 解説



