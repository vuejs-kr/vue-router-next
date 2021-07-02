# 리디렉션 및 별칭

## 리디렉션

`routes` 구성에서도 리디렉션이 수행됩니다. `/a` 에서 `/b`로  리디렉션하려면 :

```js
const routes = [{ path: '/home', redirect: '/' }]
```

리디렉션은 명명 된 경로를 대상으로 할 수도 있습니다.

```js
const routes = [{ path: '/home', redirect: { name: 'homepage' } }]
```

또는 함수를 이용해 동적으로  리디렉션을 사용할 수도 있습니다.

```js
const routes = [
  {
    // /search/screens -> /search?q=screens
    path: '/search/:searchText',
    redirect: to => {
      //  함수는 대상 경로를 인수로받습니다.
      // 여기에 리디렉션 경로 / 위치를 반환합니다.
      return { path: '/search', query: { q: to.params.searchText } }
    },
  },
  {
    path: '/search',
    // ...
  },
]
```

**[내비게이션 가드](../advanced/navigation-guards.md) 는 리디렉션하는 경로에 적용되지 않고 대상에만 적용됩니다** . 예를 들어 아래 예에서 `beforeEnter` 가드를 `/home` 경로에 추가해도 아무런 효과가 없습니다.

`redirect` 작성할 때  렌더링 할 컴포넌트가 없기 때문에 `component` 옵션을 생략 할 수 있습니다. 유일한 예외는 [중첩 된 경로입니다](./nested-routes.md) . 경로 레코드에 `children` 과 `redirect` 속성이있는 경우 `component` 속성도 있어야합니다.

### 상대 리디렉션

상대 위치로 리디렉션 할 수도 있습니다.

```js
const routes = [
  {
    path: '/users/:id/posts',
    redirect: to => {
      // 함수는 대상 경로를 인수로받습니다.
      // 여기에 리디렉션 경로 / 위치를 반환합니다.
    },
  },
]
```

## 별칭(Alias)

리디렉션 이란 사용자가 방문 `/home`에 방문했을떄 `/`로 URL이 이동해서 결과적으로 `/`가 됩니다. 그럼  별칭이란 무엇일까요?

**`/`의 별칭으로 `/home`을 등록해두고 `/home` 방문 하면 URL은 여전히  `/home`에 있지만, 내부적으로는 `/`을  방문하는 것처럼 일치된다는 것입니다.**

위의 내용은 경로 구성에서 다음과 같이 표현할 수 있습니다.

```js
const routes = [{ path: '/', component: Homepage, alias: '/home' }]
```

별칭을 사용하면 구성의 중첩 구조에 의해 제한되는 대신 UI 구조를 임의의 URL에 자유롭게 매핑 할 수 있습니다. 중첩 된 경로에서 절대 경로를 만들려면 별칭을 `/` 둘을 결합하고 배열에 여러 별칭을 제공 할 수도 있습니다.

```js
const routes = [
  {
    path: '/users',
    component: UsersLayout,
    children: [
      // this will render the UserList for these 3 URLs
      // - /users
      // - /users/list
      // - /people
      { path: '', component: UserList, alias: ['/people', 'list'] },
    ],
  },
]
```

경로에 매개 변수가있는 경우 절대 별칭에 매개 변수를 포함해야합니다.

```js
const routes = [
  {
    path: '/users/:id',
    component: UsersByIdLayout,
    children: [
      // this will render the UserDetails for these 3 URLs
      // - /users/24
      // - /users/24/profile
      // - /24
      { path: 'profile', component: UserDetails, alias: ['/:id', ''] },
    ],
  },
]
```

**SEO에 대한 참고 사항** : 별칭을 사용할 때 [표준 링크(Canonical links)](https://support.google.com/webmasters/answer/139066?hl=en) 를 정의해야합니다.
