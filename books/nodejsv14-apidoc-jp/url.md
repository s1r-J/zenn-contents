---
title: "URL"
---

# URL

> Stability: 2 - Stable

`url`モジュールはURLの結合とパースに関するユーティリティを提供します。
このモジュールは以下のようにして利用します:

```js
const url = require('url');
```

## URLストリングとURLオブジェクト

URLストリングは、複数の意味を持つコンポーネントを含む構造化された文字列です。
この文字列がパースされると、各コンポーネントのプロパティを持つURLオブジェクトが返却されます。

`url`モジュールはURLを操作するための2つのAPIを提供します: 
Node.js独自のレガシーAPIと、Webブラウザで使用される[WHATWG URL Standard][]と同じ実装をおこなった新しいAPIです。

WHATWG APIとレガシーAPIの比較を以下に示します。
`'http://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash'`というURLの上部に、レガシーAPI`url.parse()`から返却されたオブジェクトのプロパティを示します。URLの下部にはWHATWGの`URL`オブジェクトのプロパティを示しています。

WHATWG URLの`origin`プロパティには、`protocol`と`host`が含まれますが、`username`や`password`は含まれていません。

```text
┌────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                              href                                              │
├──────────┬──┬─────────────────────┬────────────────────────┬───────────────────────────┬───────┤
│ protocol │  │        auth         │          host          │           path            │ hash  │
│          │  │                     ├─────────────────┬──────┼──────────┬────────────────┤       │
│          │  │                     │    hostname     │ port │ pathname │     search     │       │
│          │  │                     │                 │      │          ├─┬──────────────┤       │
│          │  │                     │                 │      │          │ │    query     │       │
"  https:   //    user   :   pass   @ sub.example.com : 8080   /p/a/t/h  ?  query=string   #hash "
│          │  │          │          │    hostname     │ port │          │                │       │
│          │  │          │          ├─────────────────┴──────┤          │                │       │
│ protocol │  │ username │ password │          host          │          │                │       │
├──────────┴──┼──────────┴──────────┼────────────────────────┤          │                │       │
│   origin    │                     │         origin         │ pathname │     search     │ hash  │
├─────────────┴─────────────────────┴────────────────────────┴──────────┴────────────────┴───────┤
│                                              href                                              │
└────────────────────────────────────────────────────────────────────────────────────────────────┘
（""の行内の空白は全て無視してください。見やすくしているだけです。）
```

WHATWG APIを利用してURLストリングをパースするには以下のようにします:

```js
const myURL =
  new URL('https://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash');
```

レガシーAPIを利用してURLストリングをパースするには以下のようにします:

```js
const url = require('url');
const myURL =
  url.parse('https://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash');
```

## WHATWG URL API

### クラス: `URL`

WHATWG URL Standardに従って実装されたブラウザ互換`URL`クラスです。
[パースされたURLの例][]はこの標準仕様自体にあります。`URL`クラスはグローバルオブジェクトでも使用できます。

ブラウザの規則に従い、`URL`オブジェクトの全プロパティはこのオブジェクト自体のデータプロパティではなく、クラスプロトタイプのゲッターおよびセッターとして実装されています。
つまり、[レガシー`urlObject`][]とは異なり、`URL`オブジェクトのプロパティに対して`delete`演算子を利用（例 `delete myURL.protocol`や`delete myURL.pathname`等）しても効果がありませんが、`true`が返却されます。

#### `new URL(input[, base])`

* `input` {string} パースする絶対URLまたは相対URL。
  `input`が相対URLの場合、`base`が必須です。`input`が絶対URLの場合、`base`は無視されます。
* `base` {string|URL} `input`が絶対URLでない場合に結合するベースURL。

`base`に対する相対URLである`input`のパースによって新しい`URL`オブジェクトを生成します。`base`が文字列として受け渡された場合、`new URL(base)`と同じようにパースされます。

```js
const myURL = new URL('/foo', 'https://example.org/');
// https://example.org/foo
```

URLコンストラクタはグローバルオブジェクトのプロパティとしてアクセスできます。ビルトインのurlモジュールからインポートすることも可能です。

```js
console.log(URL === require('url').URL); // 出力 'true'.
```

`input`または`base`が正しいURLではなかった場合、`TypeError`がスローされます。与えられた値を文字列にどうにか変換しようとすることに注意してください。
以下に例を示します:

```js
const myURL = new URL({ toString: () => 'https://example.org/' });
// https://example.org/
```

`input`のホスト名に現れるUnicode文字は、[Punycode][]アルゴリズムによって自動的にASCIIコードに変換されます。

```js
const myURL = new URL('https://測試');
// https://xn--g6w251d/
```

この機能は、[ICU][]が有効化された状態で`node`実行可能ファイルにコンパイルされている場合のみ利用できます。無効化されている場合、ドメイン名は変更されずに受け渡されます。

`input`が絶対URLであり、`base`が渡されているかどうかが事前にわからない場合、`URL`オブジェクトの`origin`が期待通りであること検証することを推奨します。

```js
let myURL = new URL('http://Example.com/', 'https://example.org/');
// http://example.com/

myURL = new URL('https://Example.com/', 'https://example.org/');
// https://example.com/

myURL = new URL('foo://Example.com/', 'https://example.org/');
// foo://Example.com/

myURL = new URL('http:Example.com/', 'https://example.org/');
// http://example.com/

myURL = new URL('https:Example.com/', 'https://example.org/');
// https://example.org/Example.com/

myURL = new URL('foo:Example.com/', 'https://example.org/');
// foo:Example.com/
```

#### `url.hash`

* {string}

URLのフラグメント部分の取得と設定をおこないます。

```js
const myURL = new URL('https://example.org/foo#bar');
console.log(myURL.hash);
// 出力 #bar

myURL.hash = 'baz';
console.log(myURL.href);
// 出力 https://example.org/foo#baz
```

`hash`プロパティに設定される値に含まれる不正なURL文字は[パーセントエンコード][]されます。パーセントエンコードされる文字は、[`url.parse()`][]メソッドと[`url.format()`][]メソッドが生成するものとは多少異なる可能性があります。

#### `url.host`

* {string}

URLのホスト部分の取得と設定をおこないます。

```js
const myURL = new URL('https://example.org:81/foo');
console.log(myURL.host);
// 出力 example.org:81

myURL.host = 'example.com:82';
console.log(myURL.href);
// 出力 https://example.com:82/foo
```

`host`プロパティに与えられた不正な値は無視されます。

#### `url.hostname`

* {string}

URLのホスト名の取得と設定をおこないます。
`url.host`と`url.hostname`の主な違いは`url.hostname`はポート番号を**含まない**ことです。

```js
const myURL = new URL('https://example.org:81/foo');
console.log(myURL.hostname);
// 出力 example.org

// hostnameの設定ではポート番号は変更されない
myURL.hostname = 'example.com:82';
console.log(myURL.href);
// 出力 https://example.com:81/foo

// ホスト名およびポート番号を変更するためにはmyURL.hostを使う
myURL.host = 'example.org:82';
console.log(myURL.href);
// 出力 https://example.org:82/foo
```

`hostname`プロパティに与えられた不正な値は無視されます。

#### `url.href`

* {string}

シリアライズされたURLの取得と設定をおこないます。

```js
const myURL = new URL('https://example.org/foo');
console.log(myURL.href);
// 出力 https://example.org/foo

myURL.href = 'https://example.com/bar';
console.log(myURL.href);
// 出力 https://example.com/bar
```

`href`プロパティの値を取得することは[`url.toString()`][]を呼び出すことと同等です。

このプロパティの値を設定することは、[`new URL(value)`][`new URL()`]を使用して新たな`URL`オブジェクトを生成することと同等です。`URL`オブジェクトの各プロパティが変更されます。

不正なURLを`href`プロパティに値を設定する場合、`TypeError`がスローされます。

#### `url.origin`

* {string}

URLのオリジン部分の読み取り専用のシリアライズされた値の取得をおこないます。

```js
const myURL = new URL('https://example.org/foo/bar?baz');
console.log(myURL.origin);
// 出力 https://example.org
```

```js
const idnURL = new URL('https://測試');
console.log(idnURL.origin);
// 出力 https://xn--g6w251d

console.log(idnURL.hostname);
// 出力 xn--g6w251d
```

#### `url.password`

* {string}

URLのパスワード部分の取得と設定をおこないます。

```js
const myURL = new URL('https://abc:xyz@example.com');
console.log(myURL.password);
// 出力 xyz

myURL.password = '123';
console.log(myURL.href);
// 出力 https://abc:123@example.com
```

`password`プロパティに与えられた値に含まれる不正なURL文字は[パーセントエンコード][]されます。パーセントエンコードされる文字は、[`url.parse()`][]メソッドと[`url.format()`][]メソッドが生成するものとは多少異なる可能性があります。

#### `url.pathname`

* {string}

URLのパス部分の取得と設定をおこないます。

```js
const myURL = new URL('https://example.org/abc/xyz?123');
console.log(myURL.pathname);
// 出力 /abc/xyz

myURL.pathname = '/abcdef';
console.log(myURL.href);
// 出力 https://example.org/abcdef?123
```

`pathname`プロパティに与えられた値に含まれる不正なURL文字は[パーセントエンコード][]されます。パーセントエンコードされる文字は、[`url.parse()`][]メソッドと[`url.format()`][]メソッドが生成するものとは多少異なる可能性があります。

#### `url.port`

* {string}

URLのポート番号部分の取得と設定をおこないます。

ポートの値は、`0`から`65535`の範囲の数値または数値を含む文字列とします。以下の`protocol`の`URL`オブジェクトのデフォルトポート番号に対して値を設定すると、`port`値は空文字列（`''`）となります。

プロトコル/スキーマによってポート番号が決まっている場合、ポートの値は空文字列にできます。

| protocol | port |
| -------- | ---- |
| "ftp"    | 21   |
| "file"   |      |
| "gopher" | 70   |
| "http"   | 80   |
| "https"  | 443  |
| "ws"     | 80   |
| "wss"    | 443  |

ポートへの値を設定すると、値はまず最初に`.toString()`を利用して文字列に変換されます。

この文字列が不正だが数値から始まる場合、先頭の数値が`port`に設定されます。
数値が上述の範囲外だった場合、無視されます。

```js
const myURL = new URL('https://example.org:8888');
console.log(myURL.port);
// 出力 8888

// デフォルトポートは自動的に空文字列に変換されます
// （HTTPSプロトコルのデフォルトポートは443）
myURL.port = '443';
console.log(myURL.port);
// 出力 the empty string
console.log(myURL.href);
// 出力 https://example.org/

myURL.port = 1234;
console.log(myURL.port);
// 出力 1234
console.log(myURL.href);
// 出力 https://example.org:1234/

// portに対する完全に無効な文字列は無視されます
myURL.port = 'abcd';
console.log(myURL.port);
// 出力 1234

// 先頭の値がポート番号として利用できます
myURL.port = '5678abcd';
console.log(myURL.port);
// 出力 5678

// 整数ではないとき、切り捨てられます
myURL.port = 1234.5678;
console.log(myURL.port);
// 出力 1234

//  指数表記によって表示されるようなポート番号範囲外の数値は無視されます
myURL.port = 1e10; // 10000000000は前述の範囲判定に引っ掛かります
console.log(myURL.port);
// 出力 1234
```

浮動小数点数や指数表記の数のように小数点を含む数値もこのルールの例外ではありません。
小数点までの先頭の数値が適切であればURLのポート番号としてセットされます: 

```js
myURL.port = 4.567e21;
console.log(myURL.port);
// 出力 4 (because it is the leading number in the string '4.567e21')
```

#### `url.protocol`

* {string}

URLのプロトコル部分の取得と設定をおこないます。

```js
const myURL = new URL('https://example.org');
console.log(myURL.protocol);
// 出力 https:

myURL.protocol = 'ftp';
console.log(myURL.href);
// 出力 ftp://example.org/
```

`protocol`プロパティに与えられた不正な値は無視されます。

##### 特殊なスキーム

[WHATWG URL Standard][]は、パースやシリアライズをおこなう観点で_特別_として扱う少数のURLプロトコルスキームを考慮しています。これらのプロトコルの1つを使ってURLをパースするとき、`url.protocol`プロパティは別の特別なプロトコルに変更できますが、反対に特別ではないプロトコルへは変更できません。

例えば、`http`から`https`への変更は可能です:

```js
const u = new URL('http://example.org');
u.protocol = 'https';
console.log(u.href);
// https://example.org
```

しかし、`http`から架空の`fish`プロトコルへの変更はできません。なぜならば、この新しいプロトコルは特別ではないからです。

```js
const u = new URL('http://example.org');
u.protocol = 'fish';
console.log(u.href);
// http://example.org
```

同様に、特別ではないプロトコルから特別なプロトコルへの変更も許可されていません:

```js
const u = new URL('fish://example.org');
u.protocol = 'http';
console.log(u.href);
// fish://example.org
```

WHATWG URL Standardでは、特別なプロトコルスキームは、`ftp`、`file`、`gopher`、`http`、`https`、`ws`と`wss`です。

#### `url.search`

* {string}

URLのシリアライズされたクエリ部分の取得と設定をおこないます。

```js
const myURL = new URL('https://example.org/abc?123');
console.log(myURL.search);
// 出力 ?123

myURL.search = 'abc=xyz';
console.log(myURL.href);
// 出力 https://example.org/abc?abc=xyz
```

`search`プロパティに与えられた値に含まれる不正なURL文字は[パーセントエンコード][]されます。パーセントエンコードされる文字は、[`url.parse()`][]メソッドと[`url.format()`][]メソッドが生成するものとは多少異なる可能性があります。

#### `url.searchParams`

* {URLSearchParams}

URLのクエリパラメータを表す[`URLSearchParams`][]オブジェクトを取得します。
このプロパティは読み取り専用ですが、プロパティが提供する`URLSearchParams`オブジェクトを利用してURLインスタンスを変更することができます; URLのクエリパラメータ全てを変更するには[`url.search`][]セッターを利用します。詳細は[`URLSearchParams`][]ドキュメントを参照してください。

`URL`を変更するために`.searchParams`を利用するときは注意が必要です。WHATWG仕様に従い、`URLSearchParams`オブジェクトはパーセントエンコードする文字が異なります。例えば、`URL`オブジェクトはASCIIチルダ（`~`）をパーセントエンコードしませんが、`URLSearchParams`は常にエンコードします:

```js
const myUrl = new URL('https://example.org/abc?foo=~bar');

console.log(myUrl.search);  // 出力 ?foo=~bar

// searchParamsを使ってURLを変更します
myUrl.searchParams.sort();

console.log(myUrl.search);  // 出力 ?foo=%7Ebar
```

#### `url.username`

* {string}

URLのユーザ名部分の取得と設定をおこないます。

```js
const myURL = new URL('https://abc:xyz@example.com');
console.log(myURL.username);
// 出力 abc

myURL.username = '123';
console.log(myURL.href);
// 出力 https://123:xyz@example.com/
```

`username`プロパティに与えられた値に含まれる不正なURL文字は[パーセントエンコード][]されます。パーセントエンコードされる文字は、[`url.parse()`][]メソッドと[`url.format()`][]メソッドが生成するものとは多少異なる可能性があります。

#### `url.toString()`

* 戻り値: {string}

`URL`オブジェクトの`toString()`メソッドはシリアライズされたURLを返します。
返却された値は[`url.href`][]と[`url.toJSON()`][]と同等です。

標準に従う必要があるため、このメソッドはURLのシリアライズ処理のカスタマイズを許可していません。柔軟性を高くするには、[`require('url').format()`][]メソッドが参考になるかもしれません。

#### `url.toJSON()`

* 戻り値: {string}

`URL`オブジェクトの`toJSON()`メソッドはシリアライズされたURLを返却します。返却された値は[`url.href`][]および[`url.toString()`][]と同等です。

このメソッドは、[`JSON.stringify()`][]によって`URL`オブジェクトがシリアライズされるさいに自動的に呼び出されます。

```js
const myURLs = [
  new URL('https://www.example.com'),
  new URL('https://test.example.org')
];
console.log(JSON.stringify(myURLs));
// 出力 ["https://www.example.com/","https://test.example.org/"]
```

### クラス: `URLSearchParams`

`URLSearchParams`APIは`URL`のクエリへの読み取りと書き込みを提供します。
`URLSearchParams`クラスは以下の4つのコンストラクタから1つを選んで使い、単独で利用することもできます。`URLSearchParams`クラスもグローバルオブジェクトととして利用することができます。

WHATWGの`URLSearchParams`インタフェースと[`querystring`][]モジュールは似た目的を持っていますが、[`querystring`][]モジュールの目的はより汎用的で、区切り文字（`&`と`=`）のカスタマイズを許可しています。反対にこのAPIはURLクエリ文字列だけのために設計されています。

```js
const myURL = new URL('https://example.org/?abc=123');
console.log(myURL.searchParams.get('abc'));
// 出力 123

myURL.searchParams.append('abc', 'xyz');
console.log(myURL.href);
// 出力 https://example.org/?abc=123&abc=xyz

myURL.searchParams.delete('abc');
myURL.searchParams.set('a', 'b');
console.log(myURL.href);
// 出力 https://example.org/?a=b

const newSearchParams = new URLSearchParams(myURL.searchParams);
// これは以下と同等です
// const newSearchParams = new URLSearchParams(myURL.search);

newSearchParams.append('a', 'c');
console.log(myURL.href);
// 出力 https://example.org/?a=b
console.log(newSearchParams.toString());
// 出力 a=b&a=c

// newSearchParams.toString()が暗黙的に呼び出されています
myURL.search = newSearchParams;
console.log(myURL.href);
// 出力 https://example.org/?a=b&a=c
newSearchParams.delete('a');
console.log(myURL.href);
// 出力 https://example.org/?a=b&a=c
```

#### `new URLSearchParams()`

空の`URLSearchParams`オブジェクトを新しくインスタンス生成します。

#### `new URLSearchParams(string)`

* `string` {string} A query string

クエリ文字列として`string`をパースし、`URLSearchParams`オブジェクトを新しく生成するために使用します。
先頭の`'?'`が存在する場合、無視します。

```js
let params;

params = new URLSearchParams('user=abc&query=xyz');
console.log(params.get('user'));
// 出力 'abc'
console.log(params.toString());
// 出力 'user=abc&query=xyz'

params = new URLSearchParams('?user=abc&query=xyz');
console.log(params.toString());
// 出力 'user=abc&query=xyz'
```

#### `new URLSearchParams(obj)`

* `obj` {Object} キーとバリューのペアのコレクションとなっているオブジェクト

クエリのハッシュマップを使って`URLSearchParams`オブジェクトを新規に生成します。
`obj`の各プロパティのキーとバリューは常に文字列に強制的に変換します。

[`querystring`][]モジュールと異なり、配列値の形式としてキーを重複させることは許可されていません。配列は[`array.toString()`][]を使用し、配列の全要素をカンマで結合して文字列化されます。

```js
const params = new URLSearchParams({
  user: 'abc',
  query: ['first', 'second']
});
console.log(params.getAll('query'));
// 出力 [ 'first,second' ]
console.log(params.toString());
// 出力 'user=abc&query=first%2Csecond'
```

#### `new URLSearchParams(iterable)`

* `iterable` {Iterable} キーとバリューのペアとなっている要素をもつイテラブルなオブジェクト

[`Map`][]のコンストラクタと同様の方法でイテラブルなマップから`URLSearchParams`オブジェクトを新規生成します。
`iterable`は`Array`またはその他イテラブルなオブジェクトを渡すことができます。つまり、`iterable`が他の`URLSearchParams`の場合があります。この場合、このコンストラクタは与えられた`URLSearchParams`の単純なクローンを生成します。`iterable`の要素はキー-バリューのペアであり、それら自体もイテラブルなオブジェクトとすることができます。

キーの重複は許可されています。

```js
let params;

// 配列を用いる
params = new URLSearchParams([
  ['user', 'abc'],
  ['query', 'first'],
  ['query', 'second']
]);
console.log(params.toString());
// 出力 'user=abc&query=first&query=second'

// Mapオブジェクトを用いる
const map = new Map();
map.set('user', 'abc');
map.set('query', 'xyz');
params = new URLSearchParams(map);
console.log(params.toString());
// 出力 'user=abc&query=xyz'

// ジェネレータ関数を用いる
function* getQueryPairs() {
  yield ['user', 'abc'];
  yield ['query', 'first'];
  yield ['query', 'second'];
}
params = new URLSearchParams(getQueryPairs());
console.log(params.toString());
// 出力 'user=abc&query=first&query=second'

// 各キー-バリューペアは正確に2つの要素である必要があります
new URLSearchParams([
  ['user', 'abc', 'error']
]);
// Throws TypeError [ERR_INVALID_TUPLE]:
//        Each query pair must be an iterable [name, value] tuple
```

#### `urlSearchParams.append(name, value)`

* `name` {string}
* `value` {string}

新しいネーム-バリューのペアをクエリ文字列に追加します。

#### `urlSearchParams.delete(name)`

* `name` {string}

ネームが`name`と一致する全てのネーム-バリューのペアを削除します。

#### `urlSearchParams.entries()`

* 戻り値: {Iterator}

クエリ内の各ネーム-バリューペアを持つES6の`Iterator`を返却します。
イテレータの各要素はJavaScriptの`Array`となっています。`Array`の最初の要素は`name`であり、`Array`の2つ目の要素は`value`です。

[`urlSearchParams[@@iterator]()`][`urlSearchParams@@iterator()`]のエイリアスです。

#### `urlSearchParams.forEach(fn[, thisArg])`

* `fn` {Function} クエリのネーム-バリューペアごとに呼び出されます
* `thisArg` {Object} `fn`が呼び出されたときの`this`の値として利用されます

クエリ内のネーム-バリューの各ペアをイテレートし、渡された関数を呼び出します。

```js
const myURL = new URL('https://example.org/?a=b&c=d');
myURL.searchParams.forEach((value, name, searchParams) => {
  console.log(name, value, myURL.searchParams === searchParams);
});
// 出力:
//   a b true
//   c d true
```

#### `urlSearchParams.get(name)`

* `name` {string}
* 戻り値: {string}または与えられた`name`に一致するネーム-バリューのペアが存在しない場合には`null`

ネームが`name`と一致する最初のネーム-バリューペアの値を返します。
一致するペアが存在しない場合、`null`が返却されます。

#### `urlSearchParams.getAll(name)`

* `name` {string}
* 戻り値: {string[]}

ネームが`name`と一致する最初のネーム-バリューペアの値を返します。
一致するペアが存在しない場合、空配列が返却されます。

#### `urlSearchParams.has(name)`

* `name` {string}
* 戻り値: {boolean}

少なくとも1つのネームが`name`と一致する最初のネーム-バリューペアが存在する場合、`true`を返却します。

#### `urlSearchParams.keys()`

* 戻り値: {Iterator}

各ネーム-バリューペアのネームを返すES6 `Iterator`を返却します。

```js
const params = new URLSearchParams('foo=bar&foo=baz');
for (const name of params.keys()) {
  console.log(name);
}
// 出力:
//   foo
//   foo
```

#### `urlSearchParams.set(name, value)`

* `name` {string}
* `value` {string}

`URLSearchParams`オブジェクトに`name`と`value`を設定します。
`name`と一致するネーム-バリューのペアが既に存在している場合、その最初のペアのバリューを`value`に置換して他すべてのペアを削除します。存在しない場合、クエリ文字列にネーム-バリューのペアを追加します。

```js
const params = new URLSearchParams();
params.append('foo', 'bar');
params.append('foo', 'baz');
params.append('abc', 'def');
console.log(params.toString());
// 出力 foo=bar&foo=baz&abc=def

params.set('foo', 'def');
params.set('xyz', 'opq');
console.log(params.toString());
// 出力 foo=def&abc=def&xyz=opq
```

#### `urlSearchParams.sort()`

既存のネーム-バリューペアをそのネームに基づいてソートします。
ソートは[安定ソートアルゴリズム][]で実行されます。つまり、同じネームのネーム-バリューペア同士の相対的な順序は維持されます。

このメソッドは特にキャッシュヒットを増やすために使用できます。

```js
const params = new URLSearchParams('query[]=abc&type=search&query[]=123');
params.sort();
console.log(params.toString());
// 出力 query%5B%5D=abc&query%5B%5D=123&type=search
```

#### `urlSearchParams.toString()`

* 戻り値: {string}

必要であれば文字をパーセントエンコードし、文字列にシリアライズされた検索パラメータを返却します。

#### `urlSearchParams.values()`

* 戻り値: {Iterator}

各ネーム-バリューペアの値をES6 `Iterator`として返却する。

#### `urlSearchParams[Symbol.iterator]()`

* 戻り値: {Iterator}

クエリ文字列内の各ネーム-バリューペアをES6 `Iterator`として返却する。
イテレータの各要素はJavaScriptの`Array`となっています。`Array`の最初の要素は`name`であり、`Array`の2つ目の要素は`value`です。

[`urlSearchParams.entries()`][]のエイリアスです。

```js
const params = new URLSearchParams('foo=bar&xyz=baz');
for (const [name, value] of params) {
  console.log(name, value);
}
// 出力:
//   foo bar
//   xyz baz
```

### `url.domainToASCII(domain)`

* `domain` {string}
* 戻り値: {string}

`domain`を[Punycode][] ASCIIシリアライズ化して返却します。
`domain`が不正なドメインの場合、空文字列を返却します。

このメソッドは[`url.domainToUnicode()`][]の逆の処理を実行します。

```js
const url = require('url');
console.log(url.domainToASCII('español.com'));
// 出力 xn--espaol-zwa.com
console.log(url.domainToASCII('中文.com'));
// 出力 xn--fiq228c.com
console.log(url.domainToASCII('xn--iñvalid.com'));
// 出力 an empty string
```

### `url.domainToUnicode(domain)`
<!-- YAML
added:
  - v7.4.0
  - v6.13.0
-->

* `domain` {string}
* 戻り値: {string}

`domain`をUnicodeシリアライズ化して返却します。
`domain`が不正なドメインの場合、空文字列を返却します。

このメソッドは[`url.domainToASCII()`][]の逆の処理を実行します。

```js
const url = require('url');
console.log(url.domainToUnicode('xn--espaol-zwa.com'));
// 出力 español.com
console.log(url.domainToUnicode('xn--fiq228c.com'));
// 出力 中文.com
console.log(url.domainToUnicode('xn--iñvalid.com'));
// 出力 an empty string
```

### `url.fileURLToPath(url)`

* `url` {URL | string} パスに変換できるファイルのURL文字列またはURLオブジェクト
* 戻り値: {string} 完全に解決されたプラットフォームごとのNode.jsファイルパス

この関数はパーセントエンコードされた文字の正しいデコードを保証するだけでなく、クロスプラットフォームが有効な絶対パス文字列を保証します。

```js
new URL('file:///C:/path/').pathname;    // 誤: /C:/path/
fileURLToPath('file:///C:/path/');       // 正: C:\path\ (Windows)

new URL('file://nas/foo.txt').pathname;  // 誤: /foo.txt
fileURLToPath('file://nas/foo.txt');     // 正: \\nas\foo.txt (Windows)

new URL('file:///你好.txt').pathname;    // 誤: /%E4%BD%A0%E5%A5%BD.txt
fileURLToPath('file:///你好.txt');       // 正: /你好.txt (POSIX)

new URL('file:///hello world').pathname; // 誤: /hello%20world
fileURLToPath('file:///hello world');    // 正: /hello world (POSIX)
```

### `url.format(URL[, options])`

* `URL` {URL} [WHATWG URL][]オブジェクト
* `options` {Object}
  * `auth` {boolean} シリアライズされたURL文字列がユーザ名とパスワードを含める必要がある場合は`true`、そうでない場合は`false`
    **デフォルト:** `true`.
  * `fragment` {boolean} シリアライズされたURL文字列がフラグメントを含める必要がある場合は`true`、そうでない場合は`false`
    **デフォルト:** `true`.
  * `search` {boolean} シリアライズされたURL文字列が検索クエリを含める必要がある場合は`true`、そうでない場合は`false`
    **デフォルト:** `true`.
  * `unicode` {boolean} URL文字列のホスト名部分に現れるUnicode文字をPunycodeエンコードではなく、直接エンコードする必要がある場合は`true`
    **デフォルト:** `false`.
* 戻り値: {string}

[WHATWG URL][]オブジェクトのURL `String`表現のカスタマイズ可能なシリアライズされた結果を返却します。

URLオブジェクトは、URLのシリアライズされた文字列を返却する`toString()`メソッドと`href`プロパティの両方を有しています。しかし、これらはいかなる方法でもカスタマイズすることはできません。`url.format(URL[, options])`メソッドは出力の基本的なカスタマイズが可能です。

```js
const myURL = new URL('https://a:b@測試?abc#foo');

console.log(myURL.href);
// 出力 https://a:b@xn--g6w251d/?abc#foo

console.log(myURL.toString());
// 出力 https://a:b@xn--g6w251d/?abc#foo

console.log(url.format(myURL, { fragment: false, unicode: true, auth: false }));
// 出力 'https://測試/?abc'
```

### `url.pathToFileURL(path)`

* `path` {string} ファイルURLに変換するパス
* 戻り値: {URL} ファイルURLオブジェクト

この関数は`path`を絶対パスに解決し、ファイルURLに変換する際にURL操作文字を正しくエンコードすることを保証します。

```js
new URL(__filename);                // 誤: throws (POSIX)
new URL(__filename);                // 誤: C:\... (Windows)
pathToFileURL(__filename);          // 正: file:///... (POSIX)
pathToFileURL(__filename);          // 正: file:///C:/... (Windows)

new URL('/foo#1', 'file:');         // 誤: file:///foo#1
pathToFileURL('/foo#1');            // 正: file:///foo%231 (POSIX)

new URL('/some/path%.c', 'file:'); // 誤: file:///some/path%.c
pathToFileURL('/some/path%.c');    // 正: file:///some/path%25.c (POSIX)
```

## レガシーURL API

### レガシー`urlObject`

> Stability: 0 - 非推奨: WHATWG URL APIを代わりに利用してください。

The legacy `urlObject` (`require('url').Url`) is created and returned by the
`url.parse()` function.

#### `urlObject.auth`

The `auth` property is the username and password portion of the URL, also
referred to as _userinfo_. This string subset follows the `protocol` and
double slashes (if present) and precedes the `host` component, delimited by `@`.
The string is either the username, or it is the username and password separated
by `:`.

For example: `'user:pass'`.

#### `urlObject.hash`

The `hash` property is the fragment identifier portion of the URL including the
leading `#` character.

For example: `'#hash'`.

#### `urlObject.host`

The `host` property is the full lower-cased host portion of the URL, including
the `port` if specified.

For example: `'sub.example.com:8080'`.

#### `urlObject.hostname`

The `hostname` property is the lower-cased host name portion of the `host`
component *without* the `port` included.

For example: `'sub.example.com'`.

#### `urlObject.href`

The `href` property is the full URL string that was parsed with both the
`protocol` and `host` components converted to lower-case.

For example: `'http://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash'`.

#### `urlObject.path`

The `path` property is a concatenation of the `pathname` and `search`
components.

For example: `'/p/a/t/h?query=string'`.

No decoding of the `path` is performed.

#### `urlObject.pathname`

The `pathname` property consists of the entire path section of the URL. This
is everything following the `host` (including the `port`) and before the start
of the `query` or `hash` components, delimited by either the ASCII question
mark (`?`) or hash (`#`) characters.

For example: `'/p/a/t/h'`.

No decoding of the path string is performed.

#### `urlObject.port`

The `port` property is the numeric port portion of the `host` component.

For example: `'8080'`.

#### `urlObject.protocol`

The `protocol` property identifies the URL's lower-cased protocol scheme.

For example: `'http:'`.

#### `urlObject.query`

The `query` property is either the query string without the leading ASCII
question mark (`?`), or an object returned by the [`querystring`][] module's
`parse()` method. Whether the `query` property is a string or object is
determined by the `parseQueryString` argument passed to `url.parse()`.

For example: `'query=string'` or `{'query': 'string'}`.

If returned as a string, no decoding of the query string is performed. If
returned as an object, both keys and values are decoded.

#### `urlObject.search`

The `search` property consists of the entire "query string" portion of the
URL, including the leading ASCII question mark (`?`) character.

For example: `'?query=string'`.

No decoding of the query string is performed.

#### `urlObject.slashes`

The `slashes` property is a `boolean` with a value of `true` if two ASCII
forward-slash characters (`/`) are required following the colon in the
`protocol`.

### `url.format(urlObject)`

> Stability: 0 - Deprecated: Use the WHATWG URL API instead.

* `urlObject` {Object|string} A URL object (as returned by `url.parse()` or
  constructed otherwise). If a string, it is converted to an object by passing
  it to `url.parse()`.

The `url.format()` method returns a formatted URL string derived from
`urlObject`.

```js
url.format({
  protocol: 'https',
  hostname: 'example.com',
  pathname: '/some/path',
  query: {
    page: 1,
    format: 'json'
  }
});

// => 'https://example.com/some/path?page=1&format=json'
```

If `urlObject` is not an object or a string, `url.format()` will throw a
[`TypeError`][].

The formatting process operates as follows:

* A new empty string `result` is created.
* If `urlObject.protocol` is a string, it is appended as-is to `result`.
* Otherwise, if `urlObject.protocol` is not `undefined` and is not a string, an
  [`Error`][] is thrown.
* For all string values of `urlObject.protocol` that *do not end* with an ASCII
  colon (`:`) character, the literal string `:` will be appended to `result`.
* If either of the following conditions is true, then the literal string `//`
  will be appended to `result`:
  * `urlObject.slashes` property is true;
  * `urlObject.protocol` begins with `http`, `https`, `ftp`, `gopher`, or
    `file`;
* If the value of the `urlObject.auth` property is truthy, and either
  `urlObject.host` or `urlObject.hostname` are not `undefined`, the value of
  `urlObject.auth` will be coerced into a string and appended to `result`
   followed by the literal string `@`.
* If the `urlObject.host` property is `undefined` then:
  * If the `urlObject.hostname` is a string, it is appended to `result`.
  * Otherwise, if `urlObject.hostname` is not `undefined` and is not a string,
    an [`Error`][] is thrown.
  * If the `urlObject.port` property value is truthy, and `urlObject.hostname`
    is not `undefined`:
    * The literal string `:` is appended to `result`, and
    * The value of `urlObject.port` is coerced to a string and appended to
      `result`.
* Otherwise, if the `urlObject.host` property value is truthy, the value of
  `urlObject.host` is coerced to a string and appended to `result`.
* If the `urlObject.pathname` property is a string that is not an empty string:
  * If the `urlObject.pathname` *does not start* with an ASCII forward slash
    (`/`), then the literal string `'/'` is appended to `result`.
  * The value of `urlObject.pathname` is appended to `result`.
* Otherwise, if `urlObject.pathname` is not `undefined` and is not a string, an
  [`Error`][] is thrown.
* If the `urlObject.search` property is `undefined` and if the `urlObject.query`
  property is an `Object`, the literal string `?` is appended to `result`
  followed by the output of calling the [`querystring`][] module's `stringify()`
  method passing the value of `urlObject.query`.
* Otherwise, if `urlObject.search` is a string:
  * If the value of `urlObject.search` *does not start* with the ASCII question
    mark (`?`) character, the literal string `?` is appended to `result`.
  * The value of `urlObject.search` is appended to `result`.
* Otherwise, if `urlObject.search` is not `undefined` and is not a string, an
  [`Error`][] is thrown.
* If the `urlObject.hash` property is a string:
  * If the value of `urlObject.hash` *does not start* with the ASCII hash (`#`)
    character, the literal string `#` is appended to `result`.
  * The value of `urlObject.hash` is appended to `result`.
* Otherwise, if the `urlObject.hash` property is not `undefined` and is not a
  string, an [`Error`][] is thrown.
* `result` is returned.

### `url.parse(urlString[, parseQueryString[, slashesDenoteHost]])`

> Stability: 0 - 非推奨: WHATWG URL APIを代わりに利用してください。

* `urlString` {string} The URL string to parse.
* `parseQueryString` {boolean} If `true`, the `query` property will always
  be set to an object returned by the [`querystring`][] module's `parse()`
  method. If `false`, the `query` property on the returned URL object will be an
  unparsed, undecoded string. **Default:** `false`.
* `slashesDenoteHost` {boolean} If `true`, the first token after the literal
  string `//` and preceding the next `/` will be interpreted as the `host`.
  For instance, given `//foo/bar`, the result would be
  `{host: 'foo', pathname: '/bar'}` rather than `{pathname: '//foo/bar'}`.
  **Default:** `false`.

The `url.parse()` method takes a URL string, parses it, and returns a URL
object.

A `TypeError` is thrown if `urlString` is not a string.

A `URIError` is thrown if the `auth` property is present but cannot be decoded.

Use of the legacy `url.parse()` method is discouraged. Users should
use the WHATWG `URL` API. Because the `url.parse()` method uses a
lenient, non-standard algorithm for parsing URL strings, security
issues can be introduced. Specifically, issues with [host name spoofing][] and
incorrect handling of usernames and passwords have been identified.

### `url.resolve(from, to)`

> Stability: 0 - 非推奨: WHATWG URL APIを代わりに利用してください。

* `from` {string} The Base URL being resolved against.
* `to` {string} The HREF URL being resolved.

The `url.resolve()` method resolves a target URL relative to a base URL in a
manner similar to that of a Web browser resolving an anchor tag HREF.

```js
const url = require('url');
url.resolve('/one/two/three', 'four');         // '/one/two/four'
url.resolve('http://example.com/', '/one');    // 'http://example.com/one'
url.resolve('http://example.com/one', '/two'); // 'http://example.com/two'
```

<a id="whatwg-percent-encoding"></a>

## Percent-encoding in URLs

URLs are permitted to only contain a certain range of characters. Any character
falling outside of that range must be encoded. How such characters are encoded,
and which characters to encode depends entirely on where the character is
located within the structure of the URL.

### レガシーAPI

Within the Legacy API, spaces (`' '`) and the following characters will be
automatically escaped in the properties of URL objects:

```text
< > " ` \r \n \t { } | \ ^ '
```

For example, the ASCII space character (`' '`) is encoded as `%20`. The ASCII
forward slash (`/`) character is encoded as `%3C`.

### WHATWG API

The [WHATWG URL Standard][] uses a more selective and fine grained approach to
selecting encoded characters than that used by the Legacy API.

The WHATWG algorithm defines four "percent-encode sets" that describe ranges
of characters that must be percent-encoded:

* The *C0 control percent-encode set* includes code points in range U+0000 to
  U+001F (inclusive) and all code points greater than U+007E.

* The *fragment percent-encode set* includes the *C0 control percent-encode set*
  and code points U+0020, U+0022, U+003C, U+003E, and U+0060.

* The *path percent-encode set* includes the *C0 control percent-encode set*
  and code points U+0020, U+0022, U+0023, U+003C, U+003E, U+003F, U+0060,
  U+007B, and U+007D.

* The *userinfo encode set* includes the *path percent-encode set* and code
  points U+002F, U+003A, U+003B, U+003D, U+0040, U+005B, U+005C, U+005D,
  U+005E, and U+007C.

The *userinfo percent-encode set* is used exclusively for username and
passwords encoded within the URL. The *path percent-encode set* is used for the
path of most URLs. The *fragment percent-encode set* is used for URL fragments.
The *C0 control percent-encode set* is used for host and path under certain
specific conditions, in addition to all other cases.

When non-ASCII characters appear within a host name, the host name is encoded
using the [Punycode][] algorithm. Note, however, that a host name *may* contain
*both* Punycode encoded and percent-encoded characters:

```js
const myURL = new URL('https://%CF%80.example.com/foo');
console.log(myURL.href);
// 出力 https://xn--1xa.example.com/foo
console.log(myURL.origin);
// 出力 https://xn--1xa.example.com
```

[ICU]: intl.md#intl_options_for_building_node_js
[Punycode]: https://tools.ietf.org/html/rfc5891#section-4.4
[WHATWG URL Standard]: https://url.spec.whatwg.org/
[WHATWG URL]: #url_the_whatwg_url_api
[`Error`]: errors.md#errors_class_error
[`JSON.stringify()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify
[`Map`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map
[`TypeError`]: errors.md#errors_class_typeerror
[`URLSearchParams`]: #url_class_urlsearchparams
[`array.toString()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toString
[`new URL()`]: #url_new_url_input_base
[`querystring`]: querystring.md
[`require('url').format()`]: #url_url_format_url_options
[`url.domainToASCII()`]: #url_url_domaintoascii_domain
[`url.domainToUnicode()`]: #url_url_domaintounicode_domain
[`url.format()`]: #url_url_format_urlobject
[`url.href`]: #url_url_href
[`url.parse()`]: #url_url_parse_urlstring_parsequerystring_slashesdenotehost
[`url.search`]: #url_url_search
[`url.toJSON()`]: #url_url_tojson
[`url.toString()`]: #url_url_tostring
[`urlSearchParams.entries()`]: #url_urlsearchparams_entries
[`urlSearchParams@@iterator()`]: #url_urlsearchparams_symbol_iterator
[パースされたURLの例]: https://url.spec.whatwg.org/#example-url-parsing
[host name spoofing]: https://hackerone.com/reports/678487
[レガシー`urlObject`]: #url_legacy_urlobject
[パーセントエンコード]: #whatwg-percent-encoding
[安定ソートアルゴリズム]: https://en.wikipedia.org/wiki/Sorting_algorithm#Stability