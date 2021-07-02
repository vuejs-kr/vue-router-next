---
sidebarDepth: '0'
---

# 프로그래밍 방식 네비게이션

`<router-link>` 를 사용하는 것 외에도 라우터의 인스턴스 메서드를 사용하여 프로그래밍 방식으로 이를 수행 할 수 있습니다.

## 다른 위치로 이동

**Note: Vue 인스턴스 안에서는, `$router` 로 라우터 인스턴스에 액세스 할 수 있습니다. 따라서 다음처럼`this.$router.push`를 호출 할 수 있습니다.**

다른 URL로 이동하려면 `router.push` 사용하십시오. 이 방법은 새 항목을 히스토리 스택에 푸시하므로 사용자가 브라우저 뒤로 버튼을 클릭하면 이전 URL로 이동합니다.

`<router-link>` 를 클릭 할 때 내부적으로 호출되는 메서드이므로 `<router-link :to="...">` `router.push(...)` 를 호출하는 것과 같습니다.

선언적 | 프로그래밍 방식
--- | ---
`<router-link :to="...">` | `router.push(...)`

인수는 문자열 경로 또는 위치 설명자 객체 일 수 있습니다. 예 :

```js
// 리터럴 문자열 경로
router.push('/users/eduardo')

// 경로가 있는 개체
router.push({ path: '/users/eduardo' })

// 라우터가 URL을 빌드 할 수 있도록 매개 변수가있는 경로 지정
router.push({ name: 'user', params: { username: 'eduardo' } })

// with query, resulting in /register?plan=private
router.push({ path: '/register', query: { plan: 'private' } })

// with hash, resulting in /about#team
router.push({ path: '/about', hash: '#team' })
```

**참고** :  위의 예제에서처럼 `path`가 주어지면`query`가 아닐 경우 `params` 이 무시됩니다.  대신  route의 `name` 을 제공하거나 매개 변수를 `path` 를 수동으로 지정해야합니다.

```js
const username = 'eduardo'
// URL을 수동으로 만들 수 있지만 인코딩은 직접 처리해야합니다.
router.push(`/user/${username}`) // -> /user/eduardo
// 동일
router.push({ path: `/user/${username}` }) // -> /user/eduardo
// 가능하면`name`과`params`를 사용하여 자동 URL 인코딩의 이점을 얻습니다.
router.push({ name: 'user', params: { username } }) // -> /user/eduardo
//`params`는`path`와 함께 사용할 수 없습니다.
router.push({ path: '/user', params: { username } }) // -> /user
```

Since the prop `to` accepts the same kind of object as `router.push`, the exact same rules apply to both of them.

`router.push` 및 다른 모든 탐색 메서드는 탐색이 완료 될 때까지 기다릴 수 있고 성공 또는 실패 여부를 알 수 *있는 Promise를 반환합니다.* [이에 대해서는 탐색 처리](../advanced/navigation-failures.md) 에서 자세히 설명하겠습니다.

## 현재 위치 치환하기

`router.push` 처럼 작동합니다. 유일한 차이점은 이름에서 알 수 있듯이 새 기록 항목을 푸시하지 않고 탐색한다는 것입니다. 현재 항목을 대체합니다.

선언적 | 프로그래밍 방식
--- | ---
`<router-link :to="..." replace>` | `router.replace(...)`

`routeLocation` 에 전달되는 `router.push`에  `replace: true` 속성을 직접 추가 할 수도 있습니다.

```js
router.push({ path: '/home', replace: true })
// 동일함
router.replace({ path: '/home' })
```

## 히스토리 이동하기

`window.history.go(n)` 과 유사하게 히스토리 스택에서 앞으로 또는 뒤로 이동할 단계 수를 나타내는 매개 변수로 단일 정수를 사용합니다.

예

```js
// router.forward ()와 동일하게 하나의 레코드 앞으로 이동
router.go (1)

// router.back ()과 동일하게 하나의 레코드로 돌아갑니다.
router.go (-1)

// 3 개 레코드 앞으로 이동
router.go (3)

// 레코드가 많지 않으면 조용히 실패합니다.
router.go (-100)
router.go (100)
```

## 히스토리 조작

`router.push` , `router.replace` 및 `router.go` [`window.history.pushState` , `window.history.replaceState` 및 `window.history.go`](https://developer.mozilla.org/en-US/docs/Web/API/History)에 상응합니다. `window.history` API를 모방한다는 것을 알 수 있습니다.

따라서 이미 [Browser History API에](https://developer.mozilla.org/en-US/docs/Web/API/History_API) 익숙하다면 Vue Router를 사용할 때 기록을 조작하는 것이 익숙 할 것입니다.

`push` , `replace` , `go` 은 라우터 인스턴스를 만들 때 전달되는 [`history` 옵션](../../api/#history) 의 종류에 관계없이 일관되게 작동한다는 점을 언급 할 가치가 있습니다.
