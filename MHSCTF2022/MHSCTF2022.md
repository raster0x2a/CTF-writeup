# MHSCTF2022 簡易Writeup

## 目次

- [Peanuts](Peanuts)
- [Perspective](Perspective)
- [Where the Wild Cards Are 2](Where-the-Wild-Cards-Are 2)
- [Cloudy w/ a Chance of Rain](Cloudy-w/-a-Chance-of-Rain)
- [Reconstruction](Reconstruction)
- [Blank Slate 2](Blank-Slate-2)
- [Blank Slate 3](Blank-Slate-3)
- [Blank Slate 4](Blank-Slate-4)
- [Piece It Together](Piece-It-Together)

## Peanuts

ググると[Pigpen Cipher Font](https://fontmeme.com/jfont/pigpen-cipher-font/)というものが見つかる。ファベットとの対応が3パターンほど見つかるという罠がある。私は3つ目で正解でした。

## Perspective

stlファイルを開けるアプリで開いて真上から確認する。上から見てもわかりずらい。

## Where the Wild Cards Are 2

`[a-z](?=[A-Z]{2,}\d{3,})`  
`\w`  

1つ目は肯定先読みを使う。2つ目は\wというメタ文字があるのでそれが最短。

## Cloudy w/ a Chance of Rain

```py
n = int(input())
for _ in range(n):
  lst = list(map(int, input().split()))

  ans = 1
  for n in lst:
    ans *= 100 - n
  ans = 100 - ans / (100 ** (len(lst) - 1))
  print(int(ans))
```

試していないのでわからないがループ毎で除算を行ってしまうと誤差で通らないようになってそう。

## Reconstruction

行が";"区切り、列が","区切りになっているので、分割して3つずつRGBに変換。

## Blank Slate 2

ファイルデータは.gitの中のobjectsに入っているので、Pythonでobjects以下のファイルをzlib形式で解凍するスクリプトを書く。

## Blank Slate 3

RGBが(0,0,0)と(0,1,0)の部分があるので、(0,1,0)部分を(0,255,0)にした画像を作成。

## Blank Slate 4

先頭にBOMがあり、以降にゼロ幅スペースと半角スペースが並んでいるので、それぞれ0と1に置き換えて二進数から文字コードで変換。

## Piece It Together

二重のHTMLエスケープで難読化されているので変換する。最後のscriptタグの中にcheckpwd()という関数があって怪しいので、関数の中の条件式で使われている文字列を出力するとフラグ。
