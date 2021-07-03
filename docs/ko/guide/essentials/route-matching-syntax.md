# 경로 매칭 문법

대부분의 애플리케이션은 <a>Dynamic Route Matching</a> 에서 방금 본 `/about` 과 <code>/users/:userId</code> 와 같은 동적 경로를 사용하지만 Vue Router는 훨씬 더 많은 것을 제공합니다!

:::tip 
단순성을 위해 모든 경로 레코드 <code>path</code> 값에 초점을 맞추기 <strong><code data-md-type="codespan">component</code> 속성을 생략합니다.</strong> 
:::

## params에 정규 표현식 사용하기

`:userId` 와 같은 매개 변수를 정의 할 때 내부적으로 다음 정규식 `([^/]+)` `/` 가 아닌 하나 이상의 문자)을 사용하여 URL에서 매개 변수를 추출합니다. 매개 변수 내용을 기반으로 두 경로를 구별 할 필요가없는 한 이것은 잘 작동합니다. 두 경로 `/:orderId` 및 `/:productName` 상상해보십시오. 둘 다 정확히 동일한 URL과 일치하므로이를 구분할 방법이 필요합니다. 가장 쉬운 방법은 경로를 구분하는 정적 섹션을 추가하는 것입니다.

```js
const routes = [
  // matches /o/3549
  { path: '/o/:orderId' },
  // matches /p/books
  { path: '/p/:productName' },
]
```

그러나 일부 시나리오에서는 정적 섹션 `/o` / `p` 를 추가하고 싶지 않습니다. 그러나 `orderId` 는 항상 숫자이고 `productName` 은 무엇이든 될 수 있으므로 괄호 안에 매개 변수에 대한 사용자 정의 정규식을 지정할 수 있습니다.

```js
const routes = [
  // /:orderId -> 숫자만 일치
  { path: '/:orderId(\\d+)' },
  // /:productName -> 아무거나 일치
  { path: '/:productName' },
]
```

이제 `/25` 하면 `/:orderId` 와 일치하고 다른 항목으로 이동하면 `/:productName` 과 일치합니다. `routes` 배열의 순서는 중요하지 않습니다!

:::tip 
실제로 JavaScript에서 백 슬래시 문자를 문자열로 전달하기 위해 <code>\d</code> ( `\d` )에서했던 것처럼 <strong>백 슬래시 ( <code data-md-type="codespan">\</code> )를 이스케이프해야합니다.</strong> 
:::

## 반복되는 params

`/first/second/third` 와 같은 여러 섹션이있는 경로를 일치시켜야하는 경우 `*` (0 이상) 및 `+` (1 이상)를 사용하여 매개 변수를 반복 가능한 것으로 표시해야합니다.

```js
const routes = [
  // /:chapters -> matches /one, /one/two, /one/two/three, etc
  { path: '/:chapters+' },
  // /:chapters -> matches /, /one, /one/two, /one/two/three, etc
  { path: '/:chapters*' },
]
```

이렇게하면 문자열 대신 params 배열이 제공되며 명명 된 경로를 사용할 때 배열을 전달해야합니다.

```js
// 다음의 경우 { path: '/:chapters*', name: 'chapters' },
router.resolve({ name: 'chapters', params: { chapters: [] } }).href
// produces /
router.resolve({ name: 'chapters', params: { chapters: ['a', 'b'] } }).href
// produces /a/b

// 다음의 경우 { path: '/:chapters+', name: 'chapters' },
router.resolve({ name: 'chapters', params: { chapters: [] } }).href
//`chapters`가 비어 있기 때문에 오류가 발생합니다.
```

**닫는 괄호 뒤에** 추가하여 사용자 지정 Regexp와 결합 할 수도 있습니다.

```js
const routes = [
  // only match numbers
  // matches /1, /1/2, etc
  { path: '/:chapters(\\d+)+' },
  // matches /, /1, /1/2, etc
  { path: '/:chapters(\\d+)*' },
]
```

## 옵션 파라메터

`?` 를 사용하여 매개 변수를 선택 사항으로 표시 할 수도 있습니다. 수정 자 (0 또는 1) :

```js
const routes = [
  // will match /users and /users/posva
  { path: '/users/:userId?' },
  // will match /users and /users/42
  { path: '/users/:userId(\\d+)?' },
]
```

`*` 기술적으로도 매개 변수를 선택 사항으로 표시하지만 `?` 매개 변수는 반복 할 수 없습니다.

## 디버깅

경로가 일치하지 않는 이유를 이해하기 위해 경로가 Regexp로 변환되는 방법을 파헤쳐 야하거나 버그를보고하기 위해 [경로 순위 지정 도구를](https://paths.esm.dev/?p=AAMeJSyAwR4UbFDAFxAcAGAIJXMAAA..#) 사용할 수 있습니다. URL을 통한 경로 공유를 지원합니다.
