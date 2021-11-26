## CSPとは
CSP(Content Security Policy)

CSPはXSSなどの*Content Injection攻撃*に対するリスクを軽減させるセキュリティレイヤーのこと。

具体的には

- ホスト名
	- スクリプトの配信元として信頼するサーバーのホスト名を指定する。スクリプトを読み込む際に参照されて信頼できるものとして含まれていない場合は実行しない。
- ハッシュ値
	- 実効するスクリプトのハッシュ値を先に計算しておき、そのハッシュ値と同じ場合のみ実行を許可する。ハッシュ値衝突の危険性はある。
- 乱数
	- HTMLをレスポンス時に使い捨ての乱数スクリプトに付与し、付与時の乱数と等しい場合のみ実行する。

## 見分け方

CSPが設定されているかは以下の2パターンで確認できる。

**HTTPヘッダに含まれる場合**
```http
HTTP/1.1 200 OK
Host: example.com
...
Content-Security-Policy: script-src 'self'
...
```

**meta要素で記述する場合**
```html
<head>
	<meta 
		http-equiv="Content-Security-Policy" 
		content="script-src 'self'"
	>
</head>
```


## ディレクティブ
ディレクティブとは、CSPによる制限を定義するための命令記法のこと。
*\<directive-name\> \<directive-value\> ...* の順で書かれる
For exsample)
```directive
script-src 'self'
script-src 'self' 'unsafe-inline'
```


**script-srs** : スクリプトに関する制限を定義するディレクティブ。

|keyword|説明|
|---|---|
|'none'|あらゆるスクリプト|
|'self'|参照している HTML と Same-Origin であるスクリプトの実行を許可する|
|host-source: |ホスト名や IP アドレス、URLによって指定されたサーバーから配信されるスクリプトの実行を許可する。|
|'host-souce' 'nonce-<base64-value\>'



**connect-src** : script interfaces API (XMLHttpRequest や Fetch API のこと) を介して通信可能なURLの制限を定義するディレクティブ。connect-src ディレクティブの値として指定されたURL以外のアクセスは禁止される。

**default-src** : Fetch Directives に属するディレクティブのフォールバックとして機能する。
[developer.mozilla.org CSP: default-src](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Content-Security-Policy/default-src)

**base-uri** : base要素で可能なドキュメントのそうたいURLの起点を制限するディレクティブ。**none** なら安全

## Bypassing

### 

### ディレクティブの設定不備により脆弱性が生まれるパターン

**base-uri** : base要素で可能なドキュメントのそうたいURLの起点を制限するディレクティブ。