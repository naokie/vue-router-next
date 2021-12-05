# ネストしたルート

<VueSchoolLink 
  href="https://vueschool.io/lessons/nested-routes"
  title="Learn about nested routes"
/>

アプリケーションの UI は、複数レベルの深さでネストしたコンポーネントで構成されているものがあります。この場合、URL のセグメントがネストしたコンポーネントの構造に対応していることは、とてもよくあることです。例えば:

```
/user/johnny/profile                     /user/johnny/posts
+------------------+                  +-----------------+
| User             |                  | User            |
| +--------------+ |                  | +-------------+ |
| | Profile      | |  +------------>  | | Posts       | |
| |              | |                  | |             | |
| +--------------+ |                  | +-------------+ |
+------------------+                  +-----------------+
```

Vue Router ではネストしたルート構成を使ってこの関係を表現することができます。

前章で作成したアプリがあるとします:

```html
<div id="app">
  <router-view></router-view>
</div>
```

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>',
}

// these are passed to `createRouter`
const routes = [{ path: '/user/:id', component: User }]
```

ここでの `<router-view>` はトップレベルの `router-view` です。トップレベルのルートにマッチしたコンポーネントをレンダリングします。同じように、レンダリングされたコンポーネントは、それ自身のネストした `<router-view>` を含むことができます。例えば、`User` コンポーネントのテンプレートの中に1つ追加するとします:

```js
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `,
}
```

このネストした `router-view` にコンポーネントをレンダリングするには、いずれかのルートで `children` オプションを使う必要があります:

```js
const routes = [
  {
    path: '/user/:id',
    component: User,
    children: [
      {
        // UserProfile will be rendered inside User's <router-view>
        // when /user/:id/profile is matched
        path: 'profile',
        component: UserProfile,
      },
      {
        // UserPosts will be rendered inside User's <router-view>
        // when /user/:id/posts is matched
        path: 'posts',
        component: UserPosts,
      },
    ],
  },
]
```

**`/` ではじまるネストしたパスは、ルートパスとして扱われます。これにより、ネストした URL を使うことなく、コンポーネントのネストを活用することができます。**

ここまで見てきたように、この `children` オプションは `routes` 自身と同じようにルートの配列でしかありません。そのため、必要なだけビューをネストすることができます。

この時点では、上記の構成で、`/user/eduardo` にアクセスしても、`User` の `router-view` にはネストしたルートがマッチしないため、なにもレンダリングされません。もしかすると、そこでなにかをレンダリングしたいかもしれません。そのような場合には、空のネストしたパスを提供することができます:

```js
const routes = [
  {
    path: '/user/:id',
    component: User,
    children: [
      // UserHome will be rendered inside User's <router-view>
      // when /user/:id is matched
      { path: '', component: UserHome },

      // ...other sub routes
    ],
  },
]
```

この例の動作デモは [こちら](https://codesandbox.io/s/nested-views-vue-router-4-examples-hl326?initialpath=%2Fusers%2Feduardo) にあります。
