# 네비게이션 가드

이름에서 알 수 있듯이 Vue 라우터에서 제공하는 내비게이션 가드는 주로 리디렉션하거나 취소하여 내비게이션을 보호하는 데 사용됩니다. 경로를 네비게이션 하는 과정을 후킹하는 것은 몇가지 방법이 있습니다.  전역, 경로 별, 컴포넌트 내부 구성

## 전역 선행(Before) 가드

`router.beforeEach` 사용하여 전역 선행(Before)가드를 등록할수  있습니다.

```js
const router = createRouter({ ... })

router.beforeEach((to, from) => {
  // ...
  // explicitly return false to cancel the navigation
  return false
})
```

전역 선행(Before)가드는 네비게이션이 요청되었을때 만들어진 순서대로 호출됩니다. 가드는 비동기적으로 해소(resolve) 될수 있기 때문에 모든 훅이 해소되기 전에는 네비게이션은 **대기(Pending)** 상태입니다.

모든 가드 함수는 두 개의 인수를받습니다.

- **`to`** : 탐색중인 [정규화 된 형식](../../api/#routelocationnormalized) 의 목적지  경로 위치.
- **`from`** : 네비게이션이 출발하는  [되는 정규화 된 형식](../../api/#routelocationnormalized) 의 현재 경로 위치.

선택적으로 다음 값 중 하나를 반환 할 수 있습니다.

- `false` : 현재 탐색을 취소합니다. 브라우저 URL이 변경된 경우 (사용자가 수동으로 또는 뒤로 버튼을 통해) `from` 경로의 URL로 재설정됩니다.
- [경로 위치](../../api/#routelocationraw): [`router.push()`](../../api/#push) 호출하는 것처럼, 새로운  경로 위치를 전달하여 다른 위치로 리디렉션합니다. `replace: true` 또는 `name: 'home'` 과 같은 옵션을 전달할 수 있습니다. 현재 실행중인 네비게이션을 버리고 `from` 을 새로 가지는 새로운 네비게이션이 만들어집니다.

예기치 않은 상황이 발생 `Error` 발생시킬 수도 있습니다. [`router.onError()`](../../api/#onerror) 를 통해 등록 된 콜백을 호출합니다.

`undefined` 또는 `true` 가 반환되지 않으면 **&nbsp;네비게이션에 대한 검증이 완료되고 ** 되고 다음 탐색 가드가 호출됩니다.

모든 동작은 **&nbsp;`async` 함수나 ** 프로마이즈(Promises)에서도 동일하게 동작합니다.

```js
router.beforeEach(async (to, from) => {
  // canUserAccess() returns `true` or `false`
  const canAccess = await canUserAccess(to)
  if (!canAccess) return '/login'
})
```

### 선택적 세 번째 인자:`next`

이전 버전의 Vue Router에서는 <code>next</code> <em>세 번째 인자</em> 를 사용할 수도있었습니다. 이것은 일반적인 실수의 원인이었으며 이를 제거하기 위해 [RFC를 거쳤습니다.](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0037-router-return-guards.md#motivation) 그러나 여전히 지원되므로 모든 내비게이션 가드에 세 번째 인수를 전달할 수 있습니다. 이 경우 내비게이션 가드를 통해 주어진 패스에서 **정확히 한 번 `next` 를 호출해야합니다.** 두 번 이상 나타날 수 있지만 논리적 경로가 겹치지 않는 경우에만 겹치지 않으면 후크가 해결되지 않거나 오류가 발생하지 않습니다. 다음은 사용자가 인증되지 않은 경우 <code>/login</code> <strong>으로 리디렉션하는 잘못된 예입니다.</strong>

```js
// BAD
router.beforeEach((to, from, next) => {
  if (to.name !== 'Login' && !isAuthenticated) next({ name: 'Login' })
  // if the user is not authenticated, `next` is called twice
  next()
})
```

올바른 버전은 다음과 같습니다.

```js
// GOOD
router.beforeEach((to, from, next) => {
  if (to.name !== 'Login' && !isAuthenticated) next({ name: 'Login' })
  else next()
})
```

## 전역 리졸빙 가드(Global Resolve Guards)

`router.beforeResolve` 로 글로벌 가드를 등록 할 수 있습니다. <strong>이는 모든 탐색</strong> 에서 트리거되기 때문에 <code>router.beforeEach</code> 와 유사하지만 탐색이 확인되기 직전에 **모든 구성 요소 가드 및 비동기 경로 구성 요소가 해결 된 후** 해결 가드가 호출됩니다. [다음은 사용자 지정 메타](./meta.md) 속성 `requiresCamera` 정의한 경로에 대해 사용자가 카메라에 대한 액세스 권한을 부여했는지 확인하는 예입니다.

```js
router.beforeResolve(async to => {
  if (to.meta.requiresCamera) {
    try {
      await askForCameraPermission()
    } catch (error) {
      if (error instanceof NotAllowedError) {
        // ... handle the error and then cancel the navigation
        return false
      } else {
        // unexpected error, cancel the navigation and pass the error to the global handler
        throw error
      }
    }
  }
})
```

`router.beforeResolve` 는 데이터를 가져 오거나 사용자가 페이지에 들어갈 수없는 경우 수행하지 않으려는 다른 작업을 수행하기에 이상적인 지점입니다.

<!-- TODO: how to combine with [`meta` fields](./meta.md) to create a [generic fetching mechanism](#TODO). -->

## Global After Hooks

You can also register global after hooks, however unlike guards, these hooks do not get a `next` function and cannot affect the navigation:

```js
router.afterEach((to, from) => {
  sendToAnalytics(to.fullPath)
})
```

<!-- TODO: maybe add links to examples -->

They are useful for analytics, changing the title of the page, accessibility features like announcing the page and many other things.

They also reflect [navigation failures](./navigation-failures.md) as the third argument:

```js
router.afterEach((to, from, failure) => {
  if (!failure) sendToAnalytics(to.fullPath)
})
```

Learn more about navigation failures on [its guide](./navigation-failures.md).

## Per-Route Guard

You can define `beforeEnter` guards directly on a route's configuration object:

```js
const routes = [
  {
    path: '/users/:id',
    component: UserDetails,
    beforeEnter: (to, from) => {
      // reject the navigation
      return false
    },
  },
]
```

`beforeEnter` guards **only trigger when entering the route**, they don't trigger when the `params`, `query` or `hash` change e.g. going from `/users/2` to `/users/3` or going from `/users/2#info` to `/users/2#projects`. They are only triggered when navigating **from a different** route.

You can also pass an array of functions to `beforeEnter`, this is useful when reusing guards for different routes:

```js
function removeQueryParams(to) {
  if (Object.keys(to.query).length)
    return { path: to.path, query: {}, hash: to.hash }
}

function removeHash(to) {
  if (to.hash) return { path: to.path, query: to.query, hash: '' }
}

const routes = [
  {
    path: '/users/:id',
    component: UserDetails,
    beforeEnter: [removeQueryParams, removeHash],
  },
  {
    path: '/about',
    component: UserDetails,
    beforeEnter: [removeQueryParams],
  },
]
```

Note it is possible to achieve a similar behavior by using [route meta fields](./meta.md) and [global navigation guards](#global-before-guards).

## In-Component Guards

Finally, you can directly define route navigation guards inside route components (the ones passed to the router configuration)

### Using the options API

You can add the following options to route components:

- `beforeRouteEnter`
- `beforeRouteUpdate`
- `beforeRouteLeave`

```js
const UserDetails = {
  template: `...`,
  beforeRouteEnter(to, from) {
    // called before the route that renders this component is confirmed.
    // does NOT have access to `this` component instance,
    // because it has not been created yet when this guard is called!
  },
  beforeRouteUpdate(to, from) {
    // called when the route that renders this component has changed,
    // but this component is reused in the new route.
    // For example, given a route with params `/users/:id`, when we
    // navigate between `/users/1` and `/users/2`, the same `UserDetails` component instance
    // will be reused, and this hook will be called when that happens.
    // Because the component is mounted while this happens, the navigation guard has access to `this` component instance.
  },
  beforeRouteLeave(to, from) {
    // called when the route that renders this component is about to
    // be navigated away from.
    // As with `beforeRouteUpdate`, it has access to `this` component instance.
  },
}
```

The `beforeRouteEnter` guard does **NOT** have access to `this`, because the guard is called before the navigation is confirmed, thus the new entering component has not even been created yet.

However, you can access the instance by passing a callback to `next`. The callback will be called when the navigation is confirmed, and the component instance will be passed to the callback as the argument:

```js
beforeRouteEnter (to, from, next) {
  next(vm => {
    // access to component public instance via `vm`
  })
}
```

Note that `beforeRouteEnter` is the only guard that supports passing a callback to `next`. For `beforeRouteUpdate` and `beforeRouteLeave`, `this` is already available, so passing a callback is unnecessary and therefore *not supported*:

```js
beforeRouteUpdate (to, from) {
  // just use `this`
  this.name = to.params.name
}
```

The **leave guard** is usually used to prevent the user from accidentally leaving the route with unsaved edits. The navigation can be canceled by returning `false`.

```js
beforeRouteLeave (to, from) {
  const answer = window.confirm('Do you really want to leave? you have unsaved changes!')
  if (!answer) return false
}
```

### Using the composition API

If you are writing your component using the [composition API and a `setup` function](https://v3.vuejs.org/guide/composition-api-setup.html#setup), you can add update and leave guards through `onBeforeRouteUpdate` and `onBeforeRouteLeave` respectively. Please refer to the [Composition API section](./composition-api.md#navigation-guards) for more details.

## The Full Navigation Resolution Flow

1. Navigation triggered.
2. Call `beforeRouteLeave` guards in deactivated components.
3. Call global `beforeEach` guards.
4. Call `beforeRouteUpdate` guards in reused components.
5. Call `beforeEnter` in route configs.
6. Resolve async route components.
7. Call `beforeRouteEnter` in activated components.
8. Call global `beforeResolve` guards.
9. Navigation is confirmed.
10. Call global `afterEach` hooks.
11. DOM updates triggered.
12. Call callbacks passed to `next` in `beforeRouteEnter` guards with instantiated instances.
