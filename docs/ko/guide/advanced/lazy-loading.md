# 경로 지연 로딩

<VueSchoolLink 
  href="https://vueschool.io/lessons/lazy-loading-routes-vue-cli-only"
  title="Learn about lazy loading routes"
/>

번들러를 사용하여 앱을 빌드하면 JavaScript 번들이 상당히 커져 페이지 로드 시간에 영향을 미칠 수 있습니다. 각 경로의 컴포넌트를 별도의 청크로 분할하고 경로를 방문 할 때 만 로드 할 수 있다면 더 효율적일 것입니다.

Vue Router는 기본적으로 [동적 임포트(Dynamic import)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#Dynamic_Imports) 지원하므로 정적 임포트(Static Import)를 동적 임포트(Dynamic import)로 바꿀 수 있습니다.

```js
// replace
// import UserDetails from './views/UserDetails'
// with
const UserDetails = () => import('./views/UserDetails')

const router = createRouter({
  // ...
  routes: [{ path: '/users/:id', component: UserDetails }],
})
```

`component` (및 `components` ) 옵션은 컴포넌트의 약속을 반환하는 함수를 허용하며 Vue Router **는 처음 페이지에 들어갈 때만 가져온** 다음 캐시 된 버전을 사용합니다. 즉, Promise를 반환하는 한 더 복잡한 함수를 가질 수도 있습니다.

```js
const UserDetails = () =>
  Promise.resolve({
    /* component definition */
  })
```

일반적으로 모든 경로에 대해 **항상 동적 임포트(Dynamic import)** 를 사용하는 것이 좋습니다.

::: tip Router 설정에서는  [비동기 컴포넌트](https://v3.vuejs.org/guide/component-dynamic-async.html#async-components)를 **사용하지** 마십시오. 경로 컴포넌트 내에서는 여전히 비동기 컴포넌트를 사용해도 되지만, 경로 컴포넌트 자체는 정적 임포트를 해야 합니다  :::

웹팩과 같은 번 들러를 사용하면 [코드 분할의 이점이 자동으로 제공됩니다.](https://webpack.js.org/guides/code-splitting/)

Babel을 사용할 때 Babel이 구문을 올바르게 구문 분석 할 수 있도록 [syntax-dynamic-import 플러그인을 추가해야합니다.](https://babeljs.io/docs/plugins/syntax-dynamic-import/)

## 동일한 청크에서 컴포넌트 그룹화

때로는 동일한 경로에 중첩 된 모든 컴포넌트를 동일한 비동기 청크로 그룹화 할 수 있습니다. 이를 위해서는 특별한 주석 구문을 사용하여 청크 이름을 제공하여 [명명 된 청크를 사용해야합니다 (webpack&gt; 2.4 필요).](https://webpack.js.org/guides/code-splitting/#dynamic-imports)

```js
const UserDetails = () =>
  import(/* webpackChunkName: "group-user" */ './UserDetails.vue')
const UserDashboard = () =>
  import(/* webpackChunkName: "group-user" */ './UserDashboard.vue')
const UserProfileEdit = () =>
  import(/* webpackChunkName: "group-user" */ './UserProfileEdit.vue')
```

webpack은 동일한 청크 이름을 가진 모든 비동기 모듈을 동일한 비동기 청크로 그룹화합니다.
