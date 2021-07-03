# Route 컴포넌트에 Prop 전달하기


`$route` 를 사용하면 특정 URL에서만 사용할 수 있으므로 컴포넌트의 유연성을 제한하며,  경로와 긴밀하게 결합됩니다. `props` 옵션을 사용하여 이 동작을 분리 할 수 있습니다.

다음을

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
const routes = [{ path: '/user/:id', component: User }]
```

이렇게 바꿀수 있습니다.

```js
const User = {
  // route param과 똑같은 이름의 prop을 추가해야합니다.
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const routes = [{ path: '/user/:id', component: User, props: true }]
```

이를 통해 어디서나 컴포넌트를 사용할 수 있으므로 컴포넌트를 더 쉽게 재사용하고 테스트 할 수 있습니다.

## 부울 모드

`props` `true` 로 설정되면 `route.params` 가 컴포넌트 props로 설정됩니다.

## 명명 된 뷰

명명 된 뷰가 있는 경로의 경우 명명 된 각 뷰에 대해 `props`를 정의 할수 있습니다.

```js
const routes = [
  {
    path: '/user/:id',
    components: { default: User, sidebar: Sidebar },
    props: { default: true, sidebar: false }
  }
]
```

## 객체 모드

`props` 가 객체 일 때 이것은 컴포넌트 props로 설정됩니다. props가  static 일 때 유용합니다.

```js
const routes = [
  {
    path: '/promotion/from-newsletter',
    component: Promotion,
    props: { newsletterPopup: false }
  }
]
```

## 함수 모드

props을 반환하는 함수를 만들 수 있습니다. 이를 통해 매개 변수를 다른 유형으로 캐스트하고 정적 값을 경로 기반 값과 결합하는 등의 작업을 수행 할 수 있습니다.

```js
const routes = [
  {
    path: '/search',
    component: SearchUser,
    props: route => ({ query: route.query.q })
  }
]
```

URL `/search?q=vue` 는 `{query: 'vue'}` `SearchUser` 컴포넌트에 props으로 전달합니다.

`props` 함수는 경로 변경시에만 평가되므로 상태없는(Stateless) 함수여야 합니다.  props를  정의하기 위해 상태가 필요한 경우 래퍼 컴포넌트를 사용하면 vue가 상태 변경에 반응 할 수 있습니다.

고급 사용법은 [예제를](https://github.com/vuejs/vue-router/blob/dev/examples/route-props/app.js) 확인하십시오.
