# パラメータを使った動的ルートマッチング

<VueSchoolLink 
  href="https://vueschool.io/lessons/dynamic-routes"
  title="Learn about dynamic route matching with params"
/>

とても多くの場合で、与えられたパターンのルートを同じコンポーネントにマッピングする必要があるでしょう。例えば、`User` コンポーネントがすべてのユーザに対してレンダリングされなければいけませんが、それぞれユーザ ID は異なります。Vue Router では、これを実現するためにパスの中に動的なセグメントを使うことができ、これを _パラメータ_ と呼びます:

```js
const User = {
  template: '<div>User</div>',
}

// these are passed to `createRouter`
const routes = [
  // dynamic segments start with a colon
  { path: '/users/:id', component: User },
]
```

これで `/users/johnny` と `/users/jolyne` のような URL は、どちらも同じルートにマッピングされます。

_パラメータ_ はコロン `:` で示されます。ルートがマッチすると、その _パラメータ_ の値はすべてのコンポーネントで `this.$route.params` として公開されます。したがって、`User` のテンプレートを `this.$route.params` に更新することで、現在のユーザ ID を表示することができます:

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>',
}
```

同じルートに複数の _パラメータ_ を設定することができ、それらは `$route.params` の対応するフィールドにマッピングされます。例えば:

| パターン                        | マッチしたパス             | \$route.params                           |
| ------------------------------ | ------------------------ | ---------------------------------------- |
| /users/:username               | /users/eduardo           | `{ username: 'eduardo' }`                |
| /users/:username/posts/:postId | /users/eduardo/posts/123 | `{ username: 'eduardo', postId: '123' }` |

`$route.params` に加えて、`$route` オブジェクトは `$route.query`（URL にクエリがある場合）、`$route.hash` など、その他の有用な情報も公開します。すべての詳細は [API リファレンス](../../api/#routelocationnormalized) で確認できます。

この例の動作デモは [こちら](https://codesandbox.io/s/route-params-vue-router-examples-mlb14?from-embed&initialpath=%2Fusers%2Feduardo%2Fposts%2F1) にあります。

<!-- <iframe
  src="https://codesandbox.io/embed//route-params-vue-router-examples-mlb14?fontsize=14&theme=light&view=preview&initialpath=%2Fusers%2Feduardo%2Fposts%2F1"
  style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
  title="Route Params example"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe> -->

## Params の変更検知

<VueSchoolLink 
  href="https://vueschool.io/lessons/reacting-to-param-changes"
  title="Learn how to react to param changes"
/>

ルートのパラメータを使う際に特筆すべき点は、ユーザが `/users/johnny` から `/users/jolyne` へ遷移するときに **同じコンポーネントインスタンスが再利用される** ということです。両方のルートが同じコンポーネントをレンダリングするため、古いインスタンスを破棄して新しいものを作成するよりも効率的です。**しかしながら、これはコンポーネントのライフサイクルフックが呼ばれないことを意味しています**。

同じコンポーネント内のパラメータの変更を検知するには、単純に `$route` オブジェクトの何かを監視することでできます。このシナリオでは、`$route.params` を監視します:

```js
const User = {
  template: '...',
  created() {
    this.$watch(
      () => this.$route.params,
      (toParams, previousParams) => {
        // react to route changes...
      }
    )
  },
}
```

または、ナビゲーションをキャンセルすることもできる `beforeRouteUpdate` [ナビゲーションガード](../advanced/navigation-guards.md) を使ってください:

```js
const User = {
  template: '...',
  async beforeRouteUpdate(to, from) {
    // react to route changes...
    this.userData = await fetchUser(to.params.id)
  },
}
```

## すべてをキャッチ／404 Not found ルート

<VueSchoolLink 
  href="https://vueschool.io/lessons/404-not-found-page"
  title="Learn how to make a catch all/404 not found route"
/>

通常のパラメータは `/` で区切られた URL フラグメントの間の文字にのみマッチします。**何にでも** マッチさせたい場合、_パラメータ_ の直後のカッコ内に正規表現を追加することで、カスタム _パラメータ_ 正規表現を使うことができます:

```js
const routes = [
  // will match everything and put it under `$route.params.pathMatch`
  { path: '/:pathMatch(.*)*', name: 'NotFound', component: NotFound },
  // will match anything starting with `/user-` and put it under `$route.params.afterUser`
  { path: '/user-:afterUser(.*)', component: UserGeneric },
]
```

このシナリオでは、カッコの間に [カスタム正規表現](./route-matching-syntax.md#custom-regexp-in-params) を使い、`pathMatch` パラメータを [任意で繰り返し可能](./route-matching-syntax.md#optional-parameters) としてマークしています。これにより `path` を配列に分割することで、必要に応じてルートに直接ナビゲートすることができます:

```js
this.$router.push({
  name: 'NotFound',
  // preserve current path and remove the first char to avoid the target URL starting with `//`
  params: { pathMatch: this.$route.path.substring(1).split('/') },
  // preserve existing query and hash if any
  query: this.$route.query,
  hash: this.$route.hash,
})
```

詳しくは [繰り返し可能なパラメータ](./route-matching-syntax.md#repeatable-params) セクションを参照してください。

[ヒストリーモード](./history-mode.md) を使っている場合は、必ず指示に従ってサーバも正しく設定してください。

## 高度なマッチングパターン

Vue Router は、`express` で使われているものに触発された独自のパスマッチング構文を使っているため、省略可能なパラメータ、0個以上／1個以上の要件、さらにはカスタム正規表現パターンなど、多くの高度なマッチングパターンをサポートしています。それらを調べるには [高度なマッチングパターン](./route-matching-syntax.md) ドキュメントを確認してください。
