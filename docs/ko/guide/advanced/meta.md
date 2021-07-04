# 경로(Route) 메타 필드

때로는 트랜지션 이름, 경로에 접근 허용된 사용자등과 같은 임의의 정보를 경로(Route)에 첨부 할 수 있습니다. 경로 설정 객체에 `meta`  속성을 추가하면, 경로 위치나 네비게이션 가드에서 사용할수 있습니다. 다음 예제 처럼  `meta` 속성을 정의 할 수 있습니다.

```js
const routes = [
  {
    path: '/posts',
    component: PostsLayout,
    children: [
      {
        path: 'new',
        component: PostsNew,
        // 권한이 있는 사용자만 Post를 만들수 있음
        meta: { requiresAuth: true }
      },
      {
        path: ':id',
        component: PostsDetail
        // 아무나 읽을수 있음
        meta: { requiresAuth: false }
      }
    ]
  }
]
```

그렇다면 이 `meta` 필드에 어떻게 접근 할까요?

<!-- TODO: the explanation about route records should be explained before and things should be moved here -->

일단 알아두어야 할것은, `routes` 구성의 각 경로 개체를 **경로 레코드(Route record)** 라고합니다. 경로 레코드는 중첩 될 수 있습니다. 따라서 경로가 일치하면 잠재적으로 하나 이상의 경로 레코드와 일치 할 수 있습니다.

예를 들어 위의 경로 구성에서 URL `/posts/new` 는 상위 경로 레코드 ( `path: '/posts'` ) 및 하위 경로 레코드 ( `path: 'new'` )와 모두 일치합니다.

경로와 일치하는 모든 경로 레코드는 `$route`의 `$route.matched` 배열  개체로 에 노출됩니다. (내비게이션 가드에 주어지는 경로(Route) 개체에도 동일하게 노출됩니다). `meta` 필드를 확인하기 위해 해당 배열을 순회 할 수 있지만 Vue Router는 부모와 자식 경로의 <strong>&nbsp;모든 <code>meta</code></strong> 필드를 `$route.meta` 로 병합해서 제공합니다. <br>다음 예제 처럼 간단히 쓸 수 있다는 의미입니다

```js
router.beforeEach((to, from) => {
  // 경로가 접근 권한을 필요로 하는지 모든 매칭된 경로를 뒤지지 않고
  if (to.meta.requiresAuth && !auth.isLoggedIn()) {
    // 이 경로는 접근 권한을 필요로 하기 때문에 로그인 여부를 체크함
    // 권한이 없다면 로그인 페이지로 리다이렉트
    return {
      path: '/login',
      // 어느 페이지에서 왔는지 저장해둠
      query: { redirect: to.fullPath },
    }
  }
})
```

## TypeScript

`RouteMeta` 인터페이스를 확장하여 메타 필드를 입력 할 수 있습니다.

```ts
// typings.d.ts or router.ts
import 'vue-router'

declare module 'vue-router' {
  interface RouteMeta {
    // is optional
    isAdmin?: boolean
    // must be declared by every route
    requiresAuth: boolean
  }
}
```
