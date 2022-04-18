# picoCTF 2022 Writeup

## 目次

- Web
  - [Local Authority](#local-authority)
  - [Forbidden Paths](#forbidden-paths)
  - [SQL Direct](#sql-direct)
  - [Power Cookie](#power-cookie)
  - [Roboto Sans](#roboto-sans)
  - [Secrets](#secrets)
  - [SQLiLite](#sqlilite)
  - [Search source](#search-source)
- Binary(Pwn)
  - [flag leek](#flag-leek)
  - [File types](#file-types)
  - [RPS](#rps)
  - [buffer overflow 0](#buffer-overflow-0)
  - [buffer overflow 1](#buffer-overflow-1)
  - [buffer overflow 2](#buffer-overflow-2)
  - [x-sixty-what](#x-sixty-what)
- Rev
  - [Fresh Java](#fresh-java)
  - [Bbbbloat](#bbbbloat)
  - [unpackme](#unpackme)

## Forbidden Paths

相対パスを使う。
`../../../../flag.txt`

## SQL Direct

DBに接続して`\d`からの`select * from flags`

## Power Cookie

CookieのisAdminを1にして読み込む。

## Roboto Sans

robots.txtを見るとBase64エンコードされたパスがあるのでデコードしてそこにアクセス。

## Secrets

ソースコードを眺めるとsecretという謎のパスが一段入ってるものが見つかる。アクセスして順に同じことをやるとたどり着く。

## SQLiLite

シンプルなSQLインジェクション。

## Search source

CSSファイル内のコメントにフラグがある。

## flag leek

フォーマットストリング攻撃ができるので読み出す。  
`echo 'AABBCCDDEEFFGGHHIIJJKKLLMMNNOOPPQQRRSSTTUUVVWWXXYYZZ:%36$p' | ./vuln`

## File types

ファイルの中身を見るとshell archive(shar)というものらしい。shで解凍するとarという圧縮形式のファイルが出てくる。アーカイバをインストールして解凍すると次はcpioという形式のファイルが出てくる。bzip2、gzip、lgipと続いてそろそろ終わりかと思うがまだ終わらない。`ASCII text`と出てくることを願いながらfileコマンドを叩いて、アーカイバをインストールし解凍して、を繰り返すとLZ4、LZMA、lzop、lzip、XZで終わる。途中でやめようか迷った。

## RPS

ソースコードを読むと`srand(time(0))`があり、これはタイミング攻撃が可能。手元でも同じように`srand(time(0))`の`rand() % 3`を出力するCのコードを書いてコンパイルする。Pythonからその実行ファイルを呼び出して勝つ手を出すスクリプトを書いて実行してみるが絶妙にサーバー側と時間がズレていて何度も調整するがなかなか綺麗に合わせられない。ズレがあるとは言え1秒間の間では同じ手が出されるため連勝はしやすい。最終的に完璧な調整は諦めて運よく5連勝できることを待つという方法でフラグをゲット。

## buffer overflow 0

適当に長い文字列を入力に渡すとフラグが貰える。

## buffer overflow 1

win関数に飛ばせばフラグが貰える。`objdump -d vuln`などでwinのアドレスを確認し、バッファオーバーフローでvulnのリターンアドレスをwinのアドレスに書き換える。

## buffer overflow 2

これもwin関数に飛ばせばフラグが貰える。しかしwin関数に2つの引数(arg1とarg2)に正しい値を渡さないとフラグが貰えない。vulnのリターンアドレスをwinのアドレスに書き換えるとして、winのアドレスの次にarg1とarg2を置けばフラグが出力される。ここでリモートで試してみるがフラグが貰えない。GDBで試すとどうやらwinのあとにリターンアドレスでarg1を参照してセグフォで落ちている模様。0x804945あたりにpopを4回してretしているありがたいgadgetがあるので、popで引数2個分スタックポインタをずらして、mainのアドレスにリターンさせる。  
ペイロードはこんな感じになる。`b"A" * (buf_size + 8) + base_addr + win_addr + pop_addr + arg1 + arg2 + main_addr`

## x-sixty-what

64bitでのBoF。Pwn初心者なのであまり正しく理解できている自信はないが、Intel CETという機能があり、jmp/callでendbr32/64以外の場所に飛べなくする機構があるらしい(参考リンク: [【pwn 36.0】Intel CETが、みんなの恋人ROPを殺す](https://smallkirby.hatenablog.com/entry/2020/09/10/230629))。flag関数でフラグが貰えるのでBoFでflag関数に飛ばしたあとにmainに飛ばせばフラグゲット。

## Fresh Java

classファイルをデコンパイルするとフラグを1文字ずつ検証するコードが出てくる。ハードコードされているのでフラグゲット。

## Bbbbloat

特定の数値を渡すとフラグが貰える実行ファイルらしい。Ghidraでデコンパイルすると数値がわかる。

## unpackme

UPXでパックされているのでまずアンパックする。若干難読化されているがBbbbloatとほとんど変わらない動作をする実行ファイルが出てくる。Ghidraでデコンパイルしても見づらかったのでIDA Freeを使うと綺麗にデコンパイルされて数値がわかる。GhidraとIDA Freeを適材適所で使い分けられると便利かもしれない。