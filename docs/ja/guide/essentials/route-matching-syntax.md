# ルートのマッチング構文

ほとんどのアプリケーションでは、`/about` のような静的ルートや、さきほどの [動的ルートマッチング](./dynamic-matching.md) で見た `/users/:userId` のような動的ルートを使いますが、Vue Router にはもっと多くの機能があります。

:::tip
簡略化のため、すべてのルートレコードでは **`component` プロパティを省略して** `path` の値に焦点を当てています。
:::

## パラメータのカスタム正規表現

`:userId` のようなパラメータを定義する場合、内部的には次のような正規表現 `([^/]+)`（スラッシュ `/` 以外の少なくとも1文字）を使って、URL からパラメータを抽出しています。これはパラメータの内容に基づいて2つのルートを区別する必要がある場合を除いて、うまく機能します。2つのルート `/:orderId` と `/:productName` を想像してみてください。どちらもまったく同じ URL にマッチするので、それらを区別する方法が必要です。もっとも簡単な方法は、それらを区別する静的セクションをパスに追加することです:

```js
const routes = [
  // matches /o/3549
  { path: '/o/:orderId' },
  // matches /p/books
  { path: '/p/:productName' },
]
```

しかし、いくつかのシナリオでは、その静的セクション `/o` や `/p` を追加したくありません。`orderId` は常に数字であるのに対して、`productName` は何でもいいので、カッコの中のパラメータにカスタム正規表現を指定することができます:

```js
const routes = [
  // /:orderId -> matches only numbers
  { path: '/:orderId(\\d+)' },
  // /:productName -> matches anything else
  { path: '/:productName' },
]
```

これで `/25` に遷移すると `/:orderId` にマッチして、それ以外では `/:productName` にマッチするようになります。`routes` 配列の順番は問題ではありません。

:::tip
JavaScript で文字列中のバックスラッシュを実際に渡すには `\d`（`\\d` になる）のように、必ず **バックスラッシュ（`\`）をエスケープ** してください。
:::

## 繰り返し可能なパラメータ

もし `/first/second/third` のように複数のセクションを持つルートにマッチさせる必要がある場合は、`*`（0個以上）と `+`（1個以上）でパラメータを繰り返し可能とマークしてください:

```js
const routes = [
  // /:chapters -> matches /one, /one/two, /one/two/three, etc
  { path: '/:chapters+' },
  // /:chapters -> matches /, /one, /one/two, /one/two/three, etc
  { path: '/:chapters*' },
]
```

これにより、文字列の代わりにパラメータの配列が得られます。また、名前付きルートを使う場合は、配列を渡す必要があります:

```js
// given { path: '/:chapters*', name: 'chapters' },
router.resolve({ name: 'chapters', params: { chapters: [] } }).href
// produces /
router.resolve({ name: 'chapters', params: { chapters: ['a', 'b'] } }).href
// produces /a/b

// given { path: '/:chapters+', name: 'chapters' },
router.resolve({ name: 'chapters', params: { chapters: [] } }).href
// throws an Error because `chapters` is empty
```

これらはカスタム正規表現と組み合わせることも可能で、**閉じたカッコの後** に追加します:

```js
const routes = [
  // only match numbers
  // matches /1, /1/2, etc
  { path: '/:chapters(\\d+)+' },
  // matches /, /1, /1/2, etc
  { path: '/:chapters(\\d+)*' },
]
```

## 省略可能なパラメータ

`?` 修飾子（0または1）を使って、パラメータをオプションとしてマークすることもできます:

```js
const routes = [
  // will match /users and /users/posva
  { path: '/users/:userId?' },
  // will match /users and /users/42
  { path: '/users/:userId(\\d+)?' },
]
```

技術的には `*` もパラメータを省略可能とマークしますが、`?` パラメータは繰り返すことができません。

## デバッグ

どうしてルートがマッチしないかを理解するために、ルートがどのように正規表現に変換されているかを調べる必要がある場合や、バグを報告する場合は [Path Ranker ツール](https://paths.esm.dev/?p=AAMeJSyAwR4UbFDAFxAcAGAIJXMAAA..#) を使うことができます。このツールは URL を介したルートの共有をサポートしています。
