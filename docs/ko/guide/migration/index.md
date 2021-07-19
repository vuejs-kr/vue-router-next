# Vue 2에서 마이그레이션

Vue 라우터 API의 대부분은 v3(Vue 2의 경우)에서 v4(Vue 3의 경우)로 재작성하는 동안 변경되지 않았지만, 여전히 애플리케이션을 마이그레이션하는 동안 발생할 수 있는 몇 가지 주요 변경 사항이 있습니다. 이 가이드는 이러한 변경 사항이 발생한 이유와 Vue Router 4에서 작동하도록 애플리케이션을 조정하는 방법을 이해하는 데 도움이 됩니다.

## 주요 변경 사항

변경 사항은 용도에 따라 정렬됩니다. 따라서 이 목록을 순서대로 따르는 것이 좋습니다.

### new Router는 createRouter가 됩니다.

Vue Router는 더 이상 클래스가 아니라 기능의 집합입니다. `new Router()` 를 작성하는 대신 `createRouter` 를 호출해야 합니다.

```js
// previously was
// import Router from 'vue-router'
import { createRouter } from 'vue-router'

const router = createRouter({
  // ...
})
```

### `mode` 를 대체하는 새로운 `history` 옵션

`mode: 'history'`옵션dl `history` 라는 유연한 옵션으로 대체되었습니다. 사용 중인 모드에 따라 적절한 기능으로 교체해야 합니다.

- `"history"`: `createWebHistory()`
- `"hash"`: `createWebHashHistory()`
- `"abstract"`: `createMemoryHistory()`

전체 코드는 다음과 같습니다.

```js
import { createRouter, createWebHistory } from 'vue-router'
// createWebHashHistory와 createMemoryHistory도 있습니다.

createRouter({
  history: createWebHistory(),
  routes: [],
})
```

SSR에서는 적절한 history를 수동으로 전달해야 합니다.

```js
// router.js
let history = isServer ? createMemoryHistory() : createWebHistory()
let router = createRouter({ routes, history })
// server-entry.js 안의 어딘가
router.push(req.url) // request url
router.isReady().then(() => {
  // request를 resolve
})
```

**이유** : 기본 솔루션과 같은 고급 사용 사례에 대한 커스텀 history 구현뿐만 아니라 사용되지 않은 history의 트리 쉐이킹을 활성화합니다.

### `base` 옵션을 이동했습니다.

`base` 옵션은 이제 `createWebHistory` (및 기타 history)에 대한 첫 번째 인수로 전달됩니다.

```js
import { createRouter, createWebHistory } from 'vue-router'
createRouter({
  history: createWebHistory('/base-directory/'),
  routes: [],
})
```

### 제거된 `*` (별표 표시 또는 모두 표시) 경로

이제 모든 경로( `*` , `/*` )를 사용자 지정 정규식과 함께 매개변수를 사용하여 정의해야 합니다.

```js
const routes = [
  // pathMatch는 파라미터명입니다. 예를들어 /not/found로 이동합니다
  // { params: { pathMatch: ['not', 'found'] }}
  // 이 것은 반복되는 매개변수를 의미하는 마지막 * 덕분이며, 이름을 사용하여 찾을 수 없는 경로로 직접 탐색하는 경우 필요합니다.
  { path: '/:pathMatch(.*)*', name: 'not-found', component: NotFound },
  // 마지막 `*`를 생력하면, resolve하거나 push할 때, `/`문자가 인코딩됩니다.
  { path: '/:pathMatch(.*)', name: 'bad-not-found', component: NotFound },
]
// named routes를 사용하는 경우의 나쁜 예:
router.resolve({
  name: 'bad-not-found',
  params: { pathMatch: 'not/found' },
}).href // '/not%2Ffound'
// 좋은 예:
router.resolve({
  name: 'not-found',
  params: { pathMatch: ['not', 'found'] },
}).href // '/not/found'
```

:::tip 이름을 사용하여 찾을 수 없는 경로로 직접 푸시하지 않으려면 반복되는 매개변수에 `*`를 추가할 필요가 없습니다. `router.push('/not/found/url')` 를 호출하면, 올바른 `pathMatch` 매개변수를 제공됩니다. :::

**이유** : Vue Router는 `path-to-regexp` 사용하지 않고, 대신 경로 순위를 부여하고 동적 라우팅을 가능하게 하는 자체 구문 분석 시스템을 구현합니다. 일반적으로 프로젝트당 하나의 포괄(catch-all) 경로를 추가하기 때문에 `*` 대한 특수 구문을 지원하는 데 큰 이점이 없습니다. params의 인코딩은 예측하기 쉽게 하기 위해 예외 없이 경로 전체를 인코딩하는 것입니다.

### `onReady` 를 `isReady` 로 대체

기존 `router.onReady()` 함수는 인수를 취하지 않고 Promise를 반환하는 `router.isReady()` 로 대체되었습니다.

```js
// 이전
router.onReady(onSuccess, onError)
// 이후
router.isReady().then(onSuccess).catch(onError)
// 또는 await 사용하기:
try {
  await router.isReady()
  // 성공
} catch (err) {
  // 에러
}
```

### `scrollBehavior` 변경 사항

`scrollBehavior` 에서 반환된 객체 [`ScrollToOptions`](https://developer.mozilla.org/en-US/docs/Web/API/ScrollToOptions) 와 유사합니다. `x` 는 `left` 으로 이름이 바뀌고 `y` `top` 으로 이름이 바뀝니다. [RFC를](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0035-router-scroll-position.md) 참조하십시오.

**이유** `ScrollToOptions` 와 유사한 객체를 만들어 네이티브 JS API에 더 친숙하게 만들고 향후 새로운 옵션을 잠재적으로 활성화할 수 있습니다.

### `<router-view>` , `<keep-alive>` 및 `<transition>`

`transition` 및 `keep-alive`는 `v-slot` API를 통해 `RouterView` **내부** 에서 사용해야 합니다.

```vue
<router-view v-slot="{ Component }">
  <transition>
    <keep-alive>
      <component :is="Component" />
    </keep-alive>
  </transition>
</router-view>
```

**이유** : 이것은 필요한 변경이었습니다. [관련 RFC를](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0034-router-view-keep-alive-transitions.md) 참조하십시오.

### `<router-link>` 에서 `append` prop 제거

`append` prop이 `<router-link>` 에서 제거되었습니다. 대신 기존의 `path` 값을 수동으로 연결할 수 있습니다.

```html
// 이전
<router-link to="child-route" append>to relative child</router-link>
// 이후
<router-link :to="append($route.path, 'child-route')">
  to relative child
</router-link>
```

<em>App</em> 인스턴스에서 전역 <code>append</code>함수를 정의해야 합니다.

```js
app.config.globalProperties.append = (path, pathToAppend) =>
  path + (path.endsWith('/') ? '' : '/') + pathToAppend
```

**이유** : `append` 는 자주 사용되지 않으며, 사용자 영역에서 복제하기 쉽습니다.

### `<router-link>` 에서 `event` 및 `tag` prop 제거

`event` 및 `tag` props가 모두 `<router-link>` 에서 제거되었습니다. [`v-slot` API](../../api/#router-link-s-v-slot) `<router-link>` 를 완전히 커스텀할 수 있습니다.

```html
// 이전
<router-link to="/about" tag="span" event="dblclick">About Us</router-link>
// 이후
<router-link to="/about" custom v-slot="{ navigate }">
  <span @click="navigate" @keypress.enter="navigate" role="link">About Us</span>
</router-link>
```

**이유**: 이러한 props는 종종 `<a>` 태그와 다른 것을 사용하기 위해 함께 사용되었지만, `v-slot` API 이전에 도입되었으며, 모든 사람들을 위해 번들 크기에 추가하는 것을 정당화할 만큼 충분히 사용되지 않았습니다.

### `<router-link>` 에서 `exact` prop 제거

`exact` prop는 수정 중이었던 경고가 더 이상 존재하지 않으므로 안전하게 제거할 수 있기 때문에 제거되었습니다. 그러나 다음 두 가지 사항을 알고 있어야 합니다.

- 이제 Routes는 생성된 route 위치 객체와 `path` , `query` 및 `hash` 속성 대신 나타내는 route 레코드를 기반으로 활성화됩니다.
- `path` 섹션만 일치하고, `query` 및 `hash` 는 더 이상 고려되지 않습니다.

이 동작을 커스텀하려면, 예를들어 `hash` 섹션을 고려하여 [`v-slot` API](https://next.router.vuejs.org/api/#router-link-s-v-slot) 를 사용하여 `<router-link>` 를 확장해야 합니다.

**이유** : 자세한 내용은 [active matching에 대한 RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0028-router-active-link.md#summary) 변경 사항을 참조하십시오.

### mixins의 navigation guards는 무시됩니다.

현재 mixins의 navigation guards는 지원되지 않습니다. [vue-router#454](https://github.com/vuejs/vue-router-next/issues/454) 에서 지원을 추적할 수 있습니다.

### `router.match` 제거 및 `router.resolve` 변경

`router.match` 와 `router.resolve` 는 모두 약간 다른 시그니쳐를 가진 `router.resolve` 로 병합되었습니다. 자세한 내용은 [API를 참조](../../api/#resolve)하세요.

**이유** : 같은 목적으로 사용되던 여러 방법을 통합.

### `router.getMatchedComponents()` 제거

`router.getMatchedComponents` 메소드는 이제 `router.currentRoute.value.matched` 에서 일치하는 컴포넌트를 검색할 수 있으므로 제거됩니다 :

```js
router.currentRoute.value.matched.flatMap(record =>
  Object.values(record.components)
)
```

**이유** : 이 방법은 SSR 동안에만 사용되었으며 사용자가 수행할 수 있는 짧은 농담입니다.

### **모든** 탐색은 이제 항상 비동기식입니다.

첫 번째 탐색을 포함한 모든 탐색은 이제 비동기식입니다. 즉, `transition` 을 사용하는 경우 앱을 마운트하기 전에 라우터가 *준비* 될 때까지 기다려야 할 수 있습니다.

```js
app.use(router)
// 참고: 서버 사이트에서는 초기 위치를 수동으로 푸시해야합니다.
router.isReady().then(() => app.mount('#app'))
```

그렇지 않으면 라우터가 초기 위치(아무것도 않음)를 표시한 다음 첫 번째 위치를 표시하기 때문에 `appear` prop를 `transition` 에 제공한 것처럼 초기 전환이 발생합니다.

**초기 탐색 시 navigation guards가 있는 경우** 서버 측 렌더링(SSR)을 수행하지 않는 한 해결될 때까지 앱 렌더링을 차단하고 싶지 않을 수 있습니다. 이 시나리오에서 라우터가 앱을 탑재할 준비가 될 때까지 기다리지 않으면 Vue 2에서와 동일한 결과를 얻을 수 있습니다.

### `router.app` 제거

`router.app`는 라우터를 주입한 마지막 루트 구성 요소(Vue 인스턴스)를 나타내는 데 사용됩니다. Vue Router는 이제 여러 Vue 애플리케이션에서 동시에 안전하게 사용할 수 있습니다. 라우터를 사용할 때 계속 추가할 수 있습니다.

```js
app.use(router)
router.app = app
```

`Router` 인터페이스의 TypeScript 정의를 확장하여 `app` 속성을 추가할 수도 있습니다.

**이유** : Vue 3 애플리케이션은 Vue 2에 존재하지 않으며, 이제는 동일한 라우터 인스턴스를 사용하는 여러 애플리케이션을 제대로 지원하므로 `app` 속성이 있으면 루트 인스턴스 대신 애플리케이션이 되었기 때문에 오해의 소지가 있었습니다.

### route 컴포넌트의 `<slot>`에 컨텐츠 전달

Before you could directly pass a template to be rendered by a route components' `<slot>` by nesting it under a `<router-view>` component:

```html
<router-view>
  <p>In Vue Router 3, I render inside the route component</p>
</router-view>
```

`<router-view>`에 대한 `v-slot` API의 도입으로 인해 `v-slot` API를 사용하여 `<component>` 에 전달해야 합니다.

```html
<router-view v-slot="{ Component }">
  <component :is="Component">
    <p>In Vue Router 3, I render inside the route component</p>
  </component>
</router-view>
```

### route 위치에서 `parent` 제거

The `parent` property has been removed from normalized route locations (`this.$route` and object returned by `router.resolve`). You can still access it via the `matched` array:

```js
const parent = this.$route.matched[this.$route.matched.length - 2]
```

**이유** : `parent` 와 `children`이 있으면 불필요한 순환 참조가 생성되지만, 속성은 이미 `matched` 통해 검색될 수 있습니다.

### `pathToRegexpOptions` 제거

경로 레코드의 `pathToRegexpOptions` 및 `caseSensitive` 속성은 `createRouter()` 대한 `sensitive`와 `strict` 옵션으로 대체되었습니다. 이제 `createRouter()`로 라우터를 생성할 때 직접 전달할 수도 있습니다. `path-to-regexp` 관련된 다른 모든 옵션 `path-to-regexp` 가 더 이상 경로를 파싱하는 데 사용되지 않으므로 제거되었습니다.

### 이름 없는 매개변수 제거

`path-to-regexp` 제거로 인해 이름 없는 매개변수는 더 이상 지원되지 않습니다.

- `/foo(/foo)?/suffix` 는 `/foo/:_(foo)?/suffix` 가 됩니다.
- `/foo(foo)?` 는 `/foo:_(foo)?` 가 됩니다.
- `/foo/(.*)` 는 `/foo/:_(.*)` 가 됩니다.

:::tip Note 매개변수 `_` 대신 아무 이름이나 사용할 수 있습니다. 제공하는 것이 포인트입니다. :::

### `history.state` 의 사용법

Vue Router는 `history.state` 에 대한 정보를 저장합니다. `history.pushState()` 수동으로 호출하는 코드가 있는 경우, 이를 피하거나 일반 `router.push()` 및 `history.replaceState()`로 리팩토링해야 합니다.

```js
// 이전
history.pushState(myState, '', url)
// 이후
await router.push(url)
history.replaceState({ ...history.state, ...myState }, '')
```

마찬가지로 현재 상태를 유지하지 않고 `history.replaceState()` 를 호출했다면 `history.state` 를 전달해야 합니다.

```js
// 이전
history.replaceState({}, '', url)
// 이후
history.replaceState(history.state, '', url)
```

**이유** : 스크롤 위치, 이전 위치 등과 같은 탐색 정보를 저장하기 위해 history state를 사용합니다.

### `routes` option is required in `options`

이제 `options`에 `routes` 속성이 필요합니다.

```js
createRouter({ routes: [] })
```

**이유** : router는 나중에 추가할 수 있지만 route와 함께 생성되도록 설계되었습니다. 대부분의 시나리오에서 하나 이상의 route가 필요하며 일반적으로 앱당 한 번 작성됩니다.

### 존재하지 않는 명명된 경로

존재하지 않는 명명된 경로를 push하거나 resolve하면 오류가 발생합니다.

```js
// 이름을 잘못 입력했습니다.
router.push({ name: 'homee' }) // throws
router.resolve({ name: 'homee' }) // throws
```

**이유** : 이전에는 라우터가 `/` 이동했지만 아무 것도 표시하지 않았습니다(home page 대신). 탐색할 유효한 URL을 생성할 수 없기 때문에 오류를 던지는 것이 더 합리적입니다.

### 명명된 경로에 필수 `params` 누락

필수 매개변수 없이 명명된 경로를 push하거나 resolve하면 오류가 발생합니다.

```js
// 주어진 route:
const routes = [{ path: '/users/:id', name: 'user', component: UserDetails }]

// `id` 매개변수가 없으면 실패합니다.
router.push({ name: 'user' })
router.resolve({ name: 'user' })
```

**이유** : 위와 동일.

### 빈 `path` 가 있는 명명된 하위 경로는 더 이상 슬래시가 추가되지 않습니다.

빈 `path` 가 있는 중첩된 명명된 경로가 있는 경우:

```js
const routes = [
  {
    path: '/dashboard',
    name: 'dashboard-parent',
    component: DashboardParent,
    children: [
      { path: '', name: 'dashboard', component: DashboardDefault },
      {
        path: 'settings',
        name: 'dashboard-settings',
        component: DashboardSettings,
      },
    ],
  },
]
```

명명된 경로 `dashboard` 를 navigate하거나 resolve 하면 이제 **후행 슬래시가 없는** URL이 생성됩니다.

```js
router.resolve({ name: 'dashboard' }).href // '/dashboard'
```

이것은 다음과 같은 자식 `redirect` 레코드에 대한 중요한 사이드 이펙트가 있습니다.

```js
const routes = [
  {
    path: '/parent',
    component: Parent,
    children: [
      // 이제 `/parent/home` 대신 `/home`으로 리다이렉트됩니다.
      { path: '', redirect: 'home' },
      { path: 'home', component: Home },
    ],
  },
]
```

`path`가 `/parent/`에 대한 상대 위치 `home`이 실제로는 `/parent/home`이지만, `/parent`에 대한 `home`의 상대 위치가 `/home`인 경우 작동합니다.

<!-- Learn more about relative links [in the cookbook](../../cookbook/relative-links.md). -->

**이유** : 이것은 후행 슬래시 동작을 일관되게 만들기 위한 것입니다. 기본적으로 모든 경로는 후행 슬래시를 허용합니다. `strict` 옵션을 사용하고 경로에 슬래시를 수동으로 추가(또는 추가하지 않음)하여 비활성화할 수 있습니다.

<!-- TODO: maybe a cookbook entry -->

### `$route` 속성 인코딩

`params` , `query` 및 `hash`의 디코딩된 값은 이제 탐색이 시작된 위치에 관계없이 일관됩니다(이전 브라우저는 여전히 인코딩되지 않은 `path` 및 `fullPath` 생성함). 초기 탐색은 인앱 탐색과 동일한 결과를 산출해야 합니다.

[표준화 된 경로 위치가](../../api/#routelocationnormalized) 주어지면 :

- `path` , `fullPath`의 값은 더 이상 디코딩되지 않습니다. 브라우저에서 제공한 대로 나타납니다(대부분의 브라우저는 인코딩된 것을 제공합니다). 예를 들어 주소 표시줄에 `https://example.com/hello world`로 직접 작성하면 인코딩된 버전이 `https://example.com/hello%20world`가 되고 `path` 와 `fullPath` 는 모두 `/hello%20world`가 됩니다.
- `hash`가 이제 디코딩되어 `router.push({ hash: $route.hash })` 통해 복사하고 [scrollBehavior](../../api/#scrollbehavior) 의 `el` 옵션에서 직접 사용할 수 있습니다.
- `push` , `resolve` 및 `replace`를 사용하고 객체에 `string` 위치 또는 `path` 속성을 제공할 때 이전 버전과 같이 **인코딩해야 합니다.** 반면에 `params` , `query` 및 `hash` 는 인코딩되지 않은 버전으로 제공되어야 합니다.
- 슬래시 문자( `/`는 이제 `params` 내에서 올바르게 디코딩되는 동시에  URL: `%2F` 에서 인코딩된 버전을 계속 생성합니다.

**이유**: 이를 통해 `router.push()` 및 `router.resolve()`를 호출할 때, 위치의 기존 속성을 쉽게 복사하고 결과 경로 위치를 브라우저 간에 일관되게 만들 수 있습니다. `router.push()` 는 이제 멱등적(연산을 여러 번 적용하더라도 결과가 달라지지 않음)입니다. 즉, `router.push(route.fullPath)` , `router.push({ hash: route.hash })` , `router.push({ query: route.query })` 및 `router.push({ params: route.params })` 는 추가 인코딩을 생성하지 않습니다.

### TypeScript 변경 사항

타이핑을 보다 일관되고 표현력 있게 만들기 위해 일부 타입의 이름이 변경되었습니다.

`vue-router@3` | `vue-router@4`
--- | ---
RouteConfig | RouteRecordRaw
Location | RouteLocation
Route | RouteLocationNormalized

## 새로운 기능

Vue Router 4에서 주목해야 할 몇 가지 새로운 기능은 다음과 같습니다.

- [Dynamic Routing](../advanced/dynamic-routing.md)
- [Composition API](../advanced/composition-api.md)

<!-- - Custom History implementation -->
