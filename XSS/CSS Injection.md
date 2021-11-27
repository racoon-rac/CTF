# CSS Injection

- [\# CSS Injection とは](#CSS-Injection-とは)
- [\# Attribute Selector について](#Attribute-Selector-について)
- [\# CSS での HTTP Request について](#CSS-での-HTTP-Request-について)
- [\# Injection](#Injection)


## CSS Injection とは

HTML に対して CSS を挿入することで Web ページ上の機密情報の取得を狙う攻撃のこと。

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

Attribute Selector と URL 関数を利用することで、ユーザの入力した値を取得することができる。
簡潔に言うと、総当りする。

```css
input[value=]
```