# 매개변수(Params)를 활용한 동적 라우트 매칭(Dynamic Route Matching)


매우 자주 주어진 패턴의 경로를 동일한 컴포넌트에 매핑해야합니다. 예를 들어 모든 사용자에 대해 렌더링되어야하지만 사용자 ID가 다른 `User` 컴포넌트가 있을 수 있습니다. Vue Router 에서는 이를 달성하기 위해 경로의 동적 세그먼트를 사용할 수 있으며 이를 *param* 이라고합니다.

```js
const User = {
  template: '<div>User</div>',
}

// 이것은 `createRouter` 로 전달됩니다.
const routes = [
  // 동적 세그먼트는 콜론(:)으로 시작합니다.
  { path: '/users/:id', component: User },
]
```

이제 `/users/johnny` 및 `/users/jolyne` 과 같은 URL 은 모두 동일한 경로에 매핑됩니다.

*param* 은(는) 콜론 `:` (으)로 표시됩니다. 경로가 일치하면 해당 *params* 의 값이 모든 컴포넌트에서 `this.$route.params` 로 노출됩니다. 따라서 `User` 의 템플릿을 다음과 같이 업데이트하여 현재 사용자 ID를 렌더링 할 수 있습니다.

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>',
}
```

동일한 경로에 여러 *params* 가 있을 수 있으며 `$route.params` 의 해당 필드에 매핑됩니다. <br>예 :

pattern | matched path | $route.params
--- | --- | ---
/users/:username | /users/eduardo | `{ username: 'eduardo' }`
/users/:username/posts/:postId | /users/eduardo/posts/123 | `{ username: 'eduardo', postId: '123' }`

`$route.params` 외에도 `$route` 객체는 `$route.query` 와 같은 기타 유용한 정보도 노출합니다.(URL 에 query 가 있는 경우) `$route.hash` 기타 등 [API Reference](../../api/#routelocationnormalized) 에서 자세한 내용을 확인할 수 있습니다.

이 예제의 작동 데모는 [여기](https://codesandbox.io/s/route-params-vue-router-examples-mlb14?from-embed&initialpath=%2Fusers%2Feduardo%2Fposts%2F1) 에서 찾을 수 있습니다.

<!-- <iframe
  src="https://codesandbox.io/embed//route-params-vue-router-examples-mlb14?fontsize=14&theme=light&view=preview&initialpath=%2Fusers%2Feduardo%2Fposts%2F1"
  style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
  title="Route Params example"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe> -->

## 매개변수(Params) 변경에 대한 반응


매개 변수(params)와 함께 경로를 사용할 때 주의해야 할 점은 사용자가 `/users/johnny` 에서 `/users/jolyne ` 으로 이동 할 때 **&nbsp;동일한 컴포넌트 인스턴스가 재사용 된다는 것입니다. **두 경로 모두 동일한 컴포넌트를 렌더링하므로 이전 인스턴스를 삭제 한 다음 새 인스턴스를 만드는 것보다 더 효율적입니다. **그러나 이는 컴포넌트 수명주기 훅(hook)이 호출되지 않음을 의미하기도 합니다.**

동일한 컴포넌트의 매개 변수(params) 변경에 대응하려면 `$route` 객체를 감시합니다. 이 시나리오에서는 `$route.params` 의 모든 항목을 관찰하면됩니다.

```js
const User = {
  template: '...',
  created() {
    this.$watch(
      () => this.$route.params,
      (toParams, previousParams) => {
        // 라우트 변경에 대응...
      }
    )
  },
}
```

또는 `beforeRouteUpdate`, [navigation guard](../advanced/navigation-guards.md) 를 사용하여 탐색을 취소 할 수도 있습니다.

```js
const User = {
  template: '...',
  async beforeRouteUpdate(to, from) {
    // 라우트 변경에 대응...
    this.userData = await fetchUser(to.params.id)
  },
}
```

## 모두 캐치(Catch all) / 404 Not fount Route


일반 매개 변수(params)는 `/` 로 구분된 URL 조각 사이의 문자만 일치합니다. **무엇이든** 일치 시키려면 *param* 바로 뒤에 괄호 안에 정규식을 추가하여 맞춤 *param* 정규식을 사용할 수 있습니다.

```js
const routes = [
  // 모든 것을 일치시키고`$ route.params.pathMatch` 아래에 넣습니다.
  { path: '/:pathMatch(.*)*', name: 'NotFound', component: NotFound },
  // `/ user-`로 시작하는 모든 항목과 일치하고`$ route.params.afterUser` 아래에 배치합니다.
  { path: '/user-:afterUser(.*)', component: UserGeneric },
]
```

이 특정 시나리오에서는 괄호 사이에 [커스텀 정규식(custom regexp)](./route-matching-syntax.md#custom-regexp-in-params) 을 사용하고 `pathMatch` 매개 변수를 [선택적으로 반복 가능(optionally repeatable)](./route-matching-syntax.md#optional-parameters) 으로 표시합니다. 이를 통해 필요한 경우 `path` 를 배열로 분할하여 경로로 직접 이동할 수 있습니다.

```js
this.$router.push({
  name: 'NotFound',
  // 현재 경로를 유지하고 `//` 로 시작하는 대상 URL 을 피하기 위해 첫 번째 문자를 제거
  params: { pathMatch: this.$route.path.substring(1).split('/') },
  // 기존 쿼리 및 해시(있는 경우) 유지
  query: this.$route.query,
  hash: this.$route.hash,
})
```

[반복 매개변수(repeated params)](./route-matching-syntax.md#repeatable-params) 부분에서 자세히 알아보세요.

[History mode](./history-mode.md) 를 사용하는 경우 지침에 따라 서버도 올바르게 구성해야합니다.

## 고급 매칭 패턴

Vue Router 는 `express` 에서 사용하는 것에서 영감을 얻은 자체 경로 일치 구문을 사용하므로 선택적 매개 변수, 0 개 이상 또는 하나 이상의 요구 사항, 심지어 사용자 정의 정규식 패턴과 같은 많은 고급 매칭 패턴을 지원합니다. [고급 매칭(Advanced Matching)](./route-matching-syntax.md) 문서를 확인하여 살펴보세요.
