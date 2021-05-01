---
title: "Query strings"
---

# Query string

> Stability: 2 - Stable

`querystring`モジュールはURLクエリ文字列のパースと整形のためのユーティリティを提供します。
以下のように使用することができます:

```js
const querystring = require('querystring');
```

## `querystring.decode()`

`querystring.decode()`関数は`querystring.parse()`のエイリアスです。

## `querystring.encode()`

`querystring.encode()`関数は`querystring.stringify()`のエイリアスです。

## `querystring.escape(str)`

* `str` {string}

`querystring.escape()`メソッドは、URLクエリ文字列の特定の要件に最適化された方法で、与えられた`str`に対してURLパーセントエンコーディングをおこないます。

`querystring.escape()`メソッドは`querystring.stringify()`から呼び出され、基本的に直接呼び出されることは想定されていません。
このメソッドがエクスポートされている主な理由は、必要に応じてアプリケーションコードが`querystring.escape`の代替となる関数を受け渡してパーセントエンコーディング実装を置換できるようにするためです。

## `querystring.parse(str[, sep[, eq[, options]]])`

* `str` {string} 変換対象のURLクエリ文字列
* `sep` {string} クエリ文字列内でキーバリューペアを区切る文字列 **デフォルト:** `'&'`
* `eq` {string} クエリ文字列内でキーとバリューを区切る文字列 **デフォルト:** `'='`
* `options` {Object}
  * `decodeURIComponent` {Function} クエリ文字列でパーセントエンコードされた文字をデコードする際に利用される関数 **デフォルト:**
    `querystring.unescape()`
  * `maxKeys` {number} 解析するキーとの最大数を指定します。`0`を指定した場合、キーカウントの制限を排除します。 **デフォルト:** `1000`

`querystring.parse()`メソッドはURLクエリ文字列（`str`）をキーバリューペアのコレクションに変換します。

例えば、クエリ文字列`'foo=bar&abc=xyz&abc=123'`は以下のように変換されます：

<!-- eslint-skip -->
```js
{
  foo: 'bar',
  abc: ['xyz', '123']
}
```

`querystring.parse()`によって変換されたオブジェクトは、JavaScript `Object`からプロトタイプ継承を _していません_ 。
これは、 `obj.toString()`や`obj.hasOwnProperty()`のような一般的な`Object`のメソッドが定義されておらず、**動作しない**ことを意味します。

デフォルトではクエリ文字列内のパーセントエンコードされた文字はUTF-8エンコードが使われていると想定します。
他の文字エンコードが利用されている場合、`decodeURIComponent`オプションに代替となる関数を指定する必要があります：

```js
// Assuming gbkDecodeURIComponent function already exists...

querystring.parse('w=%D6%D0%CE%C4&foo=bar', null, null,
                  { decodeURIComponent: gbkDecodeURIComponent });
```

## `querystring.stringify(obj[, sep[, eq[, options]]])`

* `obj` {Object} URLクエリ文字列にシリアライズされるオブジェクト
* `sep` {string} クエリ文字列内でキーバリューペアを区切る文字列 **デフォルト:** `'&'`
* `eq` {string} クエリ文字列内でキーとバリューを区切る文字列 **デフォルト:** `'='`
* `options`
  * `encodeURIComponent` {Function} クエリ文字列内のURLアンセーフ文字をパーセントエンコーディングに変換する際に利用される関数 **デフォルト:** `querystring.escape()`.

`querystring.stringify()`メソッドは、オブジェクトの「直接のプロパティ（own properties）」をイテレートする方法で、引数の`obj`からURLクエリ文字列を生成します。

`obj`に渡された以下の型の値をシリアライズ化します：
{string|number|boolean|string\[\]|number\[\]|boolean\[\]}  
他の値は強制的に空文字列に変換します。

```js
querystring.stringify({ foo: 'bar', baz: ['qux', 'quux'], corge: '' });
// Returns 'foo=bar&baz=qux&baz=quux&corge='

querystring.stringify({ foo: 'bar', baz: 'qux' }, ';', ':');
// Returns 'foo:bar;baz:qux'
```

デフォルトでは、クエリ文字列内のパーセントエンコーディングが必要な文字はUTF-8としてエンコードされます。
他の文字エンコードが利用されている場合、`encodeURIComponent`オプションに代替となる関数を指定する必要があります：

```js
// Assuming gbkEncodeURIComponent function already exists,

querystring.stringify({ w: '中文', foo: 'bar' }, null, null,
                      { encodeURIComponent: gbkEncodeURIComponent });
```

## `querystring.unescape(str)`

* `str` {string}

`querystring.unescape()`メソッドは、引数`str`のURLパーセントエンコーディングされた文字をでデコーディングします。

`querystring.unescape()`メソッドは、`querystring.parse()`から呼び出され、基本的に直接呼び出されることは想定されていません。
このメソッドがエクスポートされている主な理由は、必要に応じてアプリケーションコードが`querystring.unescape`の代替となる関数を受け渡してデコーディング実装を置換できるようにするためです。

デフォルトでは、デコードには`querystring.unescape()`メソッドはJavaScriptビルトインの`decodeURIComponent()`メソッドを利用します。
失敗した場合、不正な形式のURLをスローしないようにするためのより安全な同等物が利用されます。
