# はじめよう

<VueMasteryVideo
  title="Get Started with Vue Router"
  url="https://player.vimeo.com/video/548250062"
  img="/Vue_Router_-_Getting_Started.jpeg"
/>

<script setup>
  import VueMasteryVideo from '../../.vitepress/components/VueMasteryVideo.vue'
  </script>

Vue と Vue Router でシングルページアプリケーション作ることは驚くほど簡単です。わたしたちはすでに Vue.js のコンポーネントでアプリケーションを構成しています。Vue Router と組み合わせるときに必要なのは、コンポーネントをルートにマッピングして、どこでレンダリングするかを Vue Router に知らせることだけです。これが基本的な例です:

## HTML

```html
<script src="https://unpkg.com/vue@3"></script>
<script src="https://unpkg.com/vue-router@4"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- use the router-link component for navigation. -->
    <!-- specify the link by passing the `to` prop. -->
    <!-- `<router-link>` will render an `<a>` tag with the correct `href` attribute -->
    <router-link to="/">Go to Home</router-link>
    <router-link to="/about">Go to About</router-link>
  </p>
  <!-- route outlet -->
  <!-- component matched by the route will render here -->
  <router-view></router-view>
</div>
```

### `router-link`

通常の `a` タグを使う代わりに、カスタムコンポーネントの `router-link` でリンクを作成していることに注意してください。これにより Vue Router はページをリロードせずに URL を変更したり、URL の生成やエンコーディングを処理することができます。これらの機能をどのように活用するかは後述します。

### `router-view`

`router-view` は、URL に対応するコンポーネントを表示します。これはどこにでも置いて、レイアウトに合わせることができます。

## JavaScript

```js
// 1. Define route components.
// These can be imported from other files
const Home = { template: '<div>Home</div>' }
const About = { template: '<div>About</div>' }

// 2. Define some routes
// Each route should map to a component.
// We'll talk about nested routes later.
const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
]

// 3. Create the router instance and pass the `routes` option
// You can pass in additional options here, but let's
// keep it simple for now.
const router = VueRouter.createRouter({
  // 4. Provide the history implementation to use. We are using the hash history for simplicity here.
  history: VueRouter.createWebHashHistory(),
  routes, // short for `routes: routes`
})

// 5. Create and mount the root instance.
const app = Vue.createApp({})
// Make sure to _use_ the router instance to make the
// whole app router-aware.
app.use(router)

app.mount('#app')

// Now the app has started!
```

`app.use(router)` を呼び出すことで、どのコンポーネントの中でも `this.$router` として、現在のルートにアクセスできるようになります:

```js
// Home.vue
export default {
  computed: {
    username() {
      // We will see what `params` is shortly
      return this.$route.params.username
    },
  },
  methods: {
    goToDashboard() {
      if (isAuthenticated) {
        this.$router.push('/dashboard')
      } else {
        this.$router.push('/login')
      }
    },
  },
}
```

`setup` 関数の内部でルータやルートにアクセスするには、`useRouter` 関数や `useRoute` 関数を呼び出します。これについては [Composition API](./advanced/composition-api.md#accessing-the-router-and-current-route-inside-setup) で詳しく説明します。

このドキュメントの中では、しばしば `router` インスタンスを使います。`this.$router` は `createRouter` で作成した `router` インスタンスを直接使うのとまったく同じであることを覚えておいてください。`this.$router` を使う理由は、ルーティングを操作する必要のあるすべての単体コンポーネントで、ルータをインポートしたくないからです。
