# 시작하기

Vue + Vue Router 를 사용하여 싱글 페이지 애플리케이션(SPA) 만드는 것은 매우 자연스럽게 느껴집니다.  Vue.js로 애플리케이션을 만들때 우리는 이미 컴포넌트들로 구성해서 만들고 있습니다.  그래서  애플리케이션에  Vue Router를 추가할때 , 우리는 컴포넌트를 경로에 매핑하고 Vue Router 에게 렌더링할 위치를 알려주기만 하면 됩니다. <br>다음은 기본적인 예입니다.

## HTML

```html
<script src="https://unpkg.com/vue@3"></script>
<script src="https://unpkg.com/vue-router@4"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 경로 탐색을 위해 router-link 컴포넌트를 사용 -->
    <!-- 경로를 특정하기 위해 `to` 속성(prop)을 지정 -->
    <!-- `<router-link>`는 올바른 `href` 속성(attribute)이 있는 `<a>`태그를 렌더링합니다. -->
    <router-link to="/">Go to Home</router-link>
    <router-link to="/about">Go to About</router-link>
  </p>
  <!-- 라우트 출구 -->
  <!-- 경로와 일치하는 컴포넌트를 여기서 렌더링 합니다 -->
  <router-view></router-view>
</div>
```

### `router-link`

일반적인 `a` 태그를 사용하는 대신 커스텀 컴포넌트  `router-link` 를 사용하여 링크를 만든다는 것을 알아두셔야 합니다.  이를 통해 Vue Router 는 페이지를 다시 로드(reloading)하지 않고 URL 을 변경하고, 문자열 인코딩이 처리된 URL을 만들어 낼수 있습니다. 나중에 이러한 기능을 활용하는 방법을 살펴 보겠습니다.

### `router-view`

`router-view` 는 URL 에 일치하는 컴포넌트를 표시합니다. 레이아웃에 맞추어 어디에나 배치 할 수 있습니다.

## JavaScript

```js
// 1. 라우트 컴포넌트 정의(Define route components)
// These can be imported from other files
const Home = { template: '<div>Home</div>' }
const About = { template: '<div>About</div>' }

// 2. 경로 정의(Define some routes)
// 각 경로는 컴포넌트에 매핑되어야 합니다.
// 나중에 중첩된 경로(nested routes)에 대해 이야기하겠습니다.
const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
]

// 3. 라우터 인스턴스(router instance)를 생성하고 'routes' 옵션을 전달합니다.
// 추가 옵션 전달도 가능하나, 여기서는 단순하게 살펴보겠습니다.
const router = VueRouter.createRouter({
  // 4. 사용할 기록(history) 구현을 제공합니다. 여기서는 단순성을 위해 해시 기록(hash history)을 사용합니다.
  history: VueRouter.createWebHashHistory(),
  routes, // `routes: routes` 의 줄임
})

// 5. 루트 인스턴스(root instance)를 생성하고 마운트
const app = Vue.createApp({})
// 애플리케이션 API 인 use() 를 사용하여 라우터 인스턴스를 전달하여 전체 앱이 라우터를 인식하도록 합니다.
app.use(router)

app.mount('#app')

// 앱이 시작되었습니다!
```

`app.use(router)` 를 호출하면 `this.$router ` 로 접근할 수 있으며 현재 경로는 `this.$route` 로 접근할 수 있습니다.<br>모든 컴포넌트 내부 :

```js
// Home.vue
export default {
  computed: {
    username() {
      // 잠시 후 'params' 가 무엇인지 알아보겠습니다.
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

 `setup` 함수 안에서 라우터나 라우트에 접근하려면 `useRouter` 또는 `useRoute` 함수를 사용합니다.   [Composition API](./advanced/composition-api.md#accessing-the-router-and-current-route-inside-setup) 에서 자세히 알아볼 것입니다.

문서 전반에 걸쳐 `router` 인스턴스를 자주 사용합니다. `this.$router`는 `createRouter` 를 통해 생성된 `router` 인스턴스를 직접 사용하는 것과 동일합니다. `this.$router` 를 사용하는 이유는 라우팅 조작이 필요한 모든 컴포넌트에서 라우터를 import하고 싶지 않기  때문입니다.
