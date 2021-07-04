# 탐색 결과를 기다리기

`router-link` 사용할 때 Vue Router는 `router.push` 를 호출하여 탐색을 트리거합니다. 대부분의 링크에서 예상되는 동작은 사용자를 새 페이지로 이동하는 것이지만 사용자가 동일한 페이지에 남아있어야 하는 몇 가지 상황이 있습니다.

- 사용자가  탐색하려는 페이지에 이미 있습니다.
- [네비게이션 가드](./navigation-guards.md)함수가   `false`를 반환하여 탐색을 중단합니다.
- 이전 가드가 완료되지 않은 동안 새로운 내비게이션 가드가 발생합니다.
- [내비게이션 가드](./navigation-guards.md)가  새로운  위치를 반환하여 다른 곳으로 리디렉션합니다 (예 : `return '/login'` ).
- [내비게이션 가드](./navigation-guards.md)가  `Error` 를 던집니다.

네비게이션이 끝난후에 추가적으로 무언가를 하고 싶다면 `router.push` 를 호출 한 후 기다릴 방법이 필요합니다. 다른 페이지로 이동할 수있는 모바일 메뉴가 있고 새 페이지로 이동 한 후에 만 메뉴를 숨기고 싶다고 가정 해 보겠습니다. 다음과 같이 할 수 있습니다.

```js
router.push('/my-profile')
this.isMenuOpen = false
```

그러나 이것은 **네비게이션은 비동기적으로 수행되기 때문에 ** 때문에 즉시 메뉴를 닫을 것입니다. 기다리기 위해서는 `router.push` 에서  반환 된 promise를 `await` 해야 합니다.

```js
await router.push('/my-profile')
this.isMenuOpen = false
```

이제 네비게이션이 완료되면 메뉴가 닫힙니다. 하지만 이 코드는  네비게이션이 취소 된 경우에도 메뉴를 닫게 됩니다. 현재 페이지가 페이지를 실제로 변경되었는지 감지하는 방법이 필요합니다.

## 네비게이션 실패 감지하기

네비게이션이 취소되어 사용자가 동일한 페이지에 머무르게 된다면, `router.push`에서 반환한  `Promise` *의 resolve 된 값은 Navigation Failure*가 됩니다. 그렇지 않다면  *falsy* 값 (일반적으로 `undefined` )이됩니다. 이를 통해 네비게이션이 성공하는 경우를 구별 할 수 있습니다.

```js
const navigationResult = await router.push('/my-profile')

if (navigationResult) {
  // 네비게이션이 방지됨
} else {
  // 네비게이션이 성공함(리다이렉트 포함)
  this.isMenuOpen = false
}
```

*네비게이션 실패(Navigation Failure)* 는 `Error`의 인스턴스로, 여러 추가 정보를 제공합니다.  네비게이션 취소 여부와 그 이유를 담고 있습니다. 이렇게 반환된 네비게이션 결과를 확인 하기 위해서  `isNavigationFailure` 함수를 사용할수 있습니다.

```js
import { NavigationFailureType, isNavigationFailure } from 'vue-router'

// 페이지 수정중에 저장없이 떠나도 되는지 시도해봄
const failure = await router.push('/articles/2')

if (isNavigationFailure(failure, NavigationFailureType.aborted)) {
  // 사용자에게 알림을 보여줌
  showToast('You have unsaved changes, discard and leave anyway?')
}
```

::: tip `isNavigationFailure(failure)`  함수를 호출할떄 두 번째 매개 변수 `failure`를 생략하면 *탐색 실패* 인지 여부 만 확인합니다. :::

## 네비게이션 실패 구분하기

처음에 언급했듯이 탐색을 중단하는 여러 상황이 있으며 모두 다른 *탐색 실패를* 초래합니다. `isNavigationFailure` 및 `NavigationFailureType` 사용하여 구분할 수 있습니다. 세 가지 유형이 있습니다.

- `aborted` : 내비게이션이 발생했을때  내비게이션 가드 내에서  `false`를 반환함
- `cancelled` : 현재 네비게이션이 완료되기 전에 새 네비게이션이 수행되었음. 예를 들어, 내비게이션 가드안에서 `router.push`가 호출됨(eg: 권한이 없어서 로그인 페이지로 이동시키는등)
- `duplicated` : 이미 대상 위치에 있기 때문에 네비게이션이 차단되었습니다.

## *네비게이션  실패(Natvigation Failure)* 속성

모든 탐색 실패는 `to` 와 `from`속성을 제공하며, 이를 통해 현재 위치와 가고자 했던 위치를 알수 있습니다.

```js
// trying to access the admin page
router.push('/admin').then(failure => {
  if (isNavigationFailure(failure, NavigationFailureType.redirected)) {
    failure.to.path // '/admin'
    failure.from.path // '/'
  }
})
```

모든 경우에, `to` 와 `from` 는 정규화 된 경로 위치입니다.

## 리디렉션 감지

내비게이션 가드 내부에 새 위치를 반환 할 때 진행중인 위치를 무시하는 새 내비게이션을 트리거합니다. 다른 반환 값과 달리 리디렉션은 탐색을 방해하지 않고 새 값 **을 만듭니다** . 이를 알기 위해  Route Location에서 `redirectedFrom` 속성을 읽을수 있습니다.

```js
await router.push('/my-profile')
if (router.currentRoute.value.redirectedFrom) {
  // redirectedFrom is resolved route location like to and from in navigation
  // guards
}
```
