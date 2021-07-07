---
sidebar: auto
---

# API 레퍼런스(API Reference)

## `<router-link>` Props

### to

- **Type**: [`RouteLocationRaw`](#routelocationraw)

- **상세**:

    링크의 대상 라우트를 나타냅니다. 클릭하면 `to` prop 의 값이 내부적으로 `router.push()` 에 전달되므로 `문자열(string)` 또는 [route location object](#routelocationraw) 일 수 있습니다.

```html
<!-- 리터럴 문자열 -->
<router-link to="/home">Home</router-link>
<!-- /home 로 이동 -->
<a href="/home">Home</a>

<!-- `v-bind`를 사용한 자바 스크립트 표현식 -->
<router-link :to="'/home'">Home</router-link>

<!-- 위와 동일 -->
<router-link :to="{ path: '/home' }">Home</router-link>

<!-- 명명된 라우트(named route) -->
<router-link :to="{ name: 'user', params: { userId: '123' }}">User</router-link>

<!-- 쿼리를 사용하면 `/register?plan=private` 이 됩니다. -->
<router-link :to="{ path: '/register', query: { plan: 'private' }}">
  Register
</router-link>
```

### replace

- **Type**: `boolean`

- **기본값**: `false`

- **상세**:

    `replace` prop 을 설정하면 클릭 시 `router.push()` 대신 `router.replace()` 가 ​​호출되므로 탐색이 기록 레코드를 남기지 않습니다.

```html
<router-link to="/abc" replace></router-link>
```

### active-class

- **Type**: `string`

- **기본값**: `"router-link-active"` (또는 전역 [`linkActiveClass`](#linkactiveclass))

- **상세**:

    링크가 활성 상태 일 때 렌더링 된 `<a>` 에 적용 할 CSS클래스 입니다.

### aria-current-value

- **Type**: `'page' | 'step' | 'location' | 'date' | 'time' | 'true' | 'false'` (`string`)

- **기본값**: `"page"`

- **상세**:

    링크가 정확히 활성 상태 일 때 `aria-current` 속성에 전달되는 값입니다.

### custom

- **Type**: `boolean`

- **기본값**: `false`

- **상세**:

    `<router-link>` 가 컨텐츠를 `<a>` 요소로 래핑하지 않아야 하는지 여부입니다. [`v-slot`](#router-link-s-v-slot) 을 사용하여 커스텀 RouterLink 를 만들 때 유용합니다. 기본적으로 `<router-link>` 는 `v-slot` 을 사용하는 경우에도 `<a>` 요소에 래핑 된 컨텐츠를 렌더링합니다. `custom` prop 을 전달하면 해당 동작이 제거됩니다.

- **예제**:

    ```html
    <router-link to="/home" custom v-slot="{ navigate, href, route }">
      <a :href="href" @click="navigate">{{ route.fullPath }}</a>
    </router-link>
    ```

    `<a href="/home">/home</a>` 생성.

    ```html
    <router-link to="/home" custom v-slot="{ route }">
      <span>{{ route.fullPath }}</span>
    </router-link>
    ```

    `<a href="/home"><span>/home</span></a>` 생성.

### exact-active-class

- **Type**: `string`

- **기본값**: `"router-link-exact-active"` (또는 전역 [`linkExactActiveClass`](#linkexactactiveclass))

- **상세**:

    링크가 정확히 활성 상태 일 때 렌더링 된 `<a>`에 적용 할 CSS클래스 입니다.

## `<router-link>` 의 `v-slot`

`<router-link>` 는 [scoped slot](https://v3.vuejs.org/guide/component-slots.html#scoped-slots) 을 통해 낮은 수준의 맞춤 설정을 노출합니다. 이 API는 주로 라이브러리 작성자를 대상으로 하지만 *NavLink* 등의 맞춤 컴포넌트를 빌드하는데 개발자에게도 유용할 수 있는 고급 API입니다.

::: 팁 `custom` 옵션을 `<router-link>` 에 전달하여 `<a>` 요소 내부의 컨텐츠를 래핑하지 않도록해야합니다. :::

```html
<router-link
  to="/about"
  custom
  v-slot="{ href, route, navigate, isActive, isExactActive }"
>
  <NavLink :active="isActive" :href="href" @click="navigate">
    {{ route.fullPath }}
  </NavLink>
</router-link>
```

- `href` : 해결된 URL. 이는 `<a>` 요소의 `href` 속성입니다. 제공된 경우 `base` 를 포함합니다.
- `route`: 확인된 표준화 된 위치.
- `navigate` : 탐색을 트리거 하는 함수입니다. **필요한 경우 자동으로 이벤트를 방지합니다.** 예를 들어 `router-link` 와 같은 방식으로 `ctrl` 또는 `cmd` + 클릭은 `navigate` 에서 여전히 무시됩니다.
- `isActive` : [active class](#active-class) 를 적용해야 하는 경우 `true` 입니다. 임의의 CSS 클래스를 적용 할 수 있습니다.
- `isExactActive` : [exact active class](#exact-active-class) 를 적용 해야하는 경우 `true` 입니다. 임의의 CSS 클래스를 적용 할 수 있습니다.

### 예 : 외부 요소에 활성 클래스 적용

때로는 활성 클래스가 `<a>` 요소 자체가 아닌 외부 요소에 적용되기를 원할 수 있습니다. 이 경우 해당 요소를 `router-link` 안에 래핑 할 수 있습니다. 그리고 `v-slot` 속성을 사용하여 링크를 만듭니다.

```html
<router-link
  to="/foo"
  custom
  v-slot="{ href, route, navigate, isActive, isExactActive }"
>
  <li
    :class="[isActive && 'router-link-active', isExactActive && 'router-link-exact-active']"
  >
    <a :href="href" @click="navigate">{{ route.fullPath }}</a>
  </li>
</router-link>
```

::: 팁 `target="_ blank"` 를 `a` 요소에 추가하는 경우 `@click="navigate"` 핸들러를 생략 해야합니다. :::

## `<router-view>` Props

### name

- **Type**: `string`

- **기본값**: `"default"`

- **상세**:

    `<router-view>` 에 `name` 이 있으면 일치하는 라우트 레코드의 `components` 옵션에서 해당 이름으로 컴포넌를 렌더링합니다.

- **참고**: [명명된 뷰(Named Views)](../guide/essentials/named-views.md)

### route

- **Type**: [`RouteLocationNormalized`](#routelocationnormalized)

- **상세**:

    표시 할 수 있도록 모든 컴포넌트가 확인 된 라우트 위치입니다 (지연로드 된 경우).

## `<router-view>` 의 `v-slot`

`<router-view>` 는 주로 `<transition>` 및 `<keep-alive>` 컴포넌트 로 라우트 컴포넌트를 래핑하기 위해 `v-slot` API가 노출되어있습니다.

```html
<Suspense>
  <template #default>
    <router-view v-slot="{ Component, route }">
      <transition :name="route.meta.transition || 'fade'" mode="out-in">
        <keep-alive>
          <component
            :is="Component"
            :key="route.meta.usePathKey ? route.path : undefined"
          />
        </keep-alive>
      </transition>
    </router-view>
  </template>
  <template #fallback> Loading... </template>
</Suspense>
```

- `Component`: `<component>` 의 `is` prop 에 전달 될 VNode.
- `route`: 표준화 된 [라우트 위치(route location)](#routelocationnormalized).

## createRouter

Vue 앱에서 사용할 수있는 라우터 인스턴스를 만듭니다. 전달할 수있는 모든 속성 목록은 [`라우터 옵션(RouterOptions)`](#routeroptions) 을 확인하세요.

**형식(Signature):**

```typescript
export declare function createRouter(options: RouterOptions): Router
```

### 매개변수(Parameters)

Parameter | Type | Description
--- | --- | ---
options | [RouterOptions](#routeroptions) | 라우터 초기화 옵션

## createWebHistory

HTML5 history 를 생성합니다. 싱글 페이지 애플리케이션에 대한 가장 일반적인 기록. 애플리케이션은 http 프로토콜을 통해 제공되어야합니다.

**형식(Signature):**

```typescript
export declare function createWebHistory(base?: string): RouterHistory
```

### 매개변수(Parameters)

Parameter | Type | Description
--- | --- | ---
base | `string` | 제공 할 선택 베이스(optional base). 애플리케이션이 `https://example.com/folder/` 와 같은 폴더 내부에서 호스팅 될 때 유용합니다.

### 예제

```js
createWebHistory() // 베이스가 없습니다. 앱은 도메인 'https://example.com' 의 루트에서 호스팅됩니다.
createWebHistory('/folder/') // 'https://example.com/folder/' URL을 제공합니다.
```

## createWebHashHistory

해시 히스토리(hash history)를 생성합니다. 호스트가 없는 웹 애플리케이션 (예 : `file://`) 또는 URL 을 처리하도록 서버를 구성하는 것이 옵션이 아닌 경우에 유용합니다. **SEO 가 중요한 경우[`createWebHistory`](#createwebhistory) 를 사용해야합니다.**

**형식(Signature):**

```typescript
export declare function createWebHashHistory(base?: string): RouterHistory
```

### 매개변수(Parameters)

Parameter | Type | Description
--- | --- | ---
base | `string` | optional base to provide. Defaults to `location.pathname + location.search`. If there is a `<base>` tag in the `head`, its value will be ignored in favor of this parameter **but note it affects all the history.pushState() calls**, meaning that if you use a `<base>` tag, its `href` value **has to match this parameter** (ignoring anything after the `#`)

### 예제

```js
// at https://example.com/folder
createWebHashHistory() // gives a url of `https://example.com/folder#`
createWebHashHistory('/folder/') // gives a url of `https://example.com/folder/#`
// if the `#` is provided in the base, it won't be added by `createWebHashHistory`
createWebHashHistory('/folder/#/app/') // gives a url of `https://example.com/folder/#/app/`
// you should avoid doing this because it changes the original url and breaks copying urls
createWebHashHistory('/other-folder/') // gives a url of `https://example.com/other-folder/#`

// at file:///usr/etc/folder/index.html
// for locations with no `host`, the base is ignored
createWebHashHistory('/iAmIgnored') // gives a url of `file:///usr/etc/folder/index.html#`
```

## createMemoryHistory

인 메모리 기반 히스토리(in-memory based history)를 생성합니다. 이 히스토리의 주요 목적은 SSR 을 처리하는 것입니다. 그것은 어디에도없는 특별한 위치에서 시작됩니다. 사용자가 브라우저 컨텍스트에없는 경우 `router.push()` 또는 `router.replace()` 를 호출하여 해당 위치를 시작 위치로 바꾸어야 합니다.

**형식(Signature):**

```typescript
export declare function createMemoryHistory(base?: string): RouterHistory
```

### 매개변수(Parameters)

Parameter | Type | Description
--- | --- | ---
base | `string` | 모든 URL 에 적용되는 베이스, 기본값은 '/'

### 반환값(Returns)

A history object that can be passed to the router constructor

## 탐색 실패 유형(NavigationFailureType)

탐색 실패에 대해 가능한 모든 유형이있는 열거. 특정 실패를 확인하기 위해 [isNavigationFailure](#isnavigationfailure) 에 전달할 수 있습니다. **숫자 값을 사용하지 마세요**. 항상 `NavigationFailureType.aborted` 와 같은 변수를 사용하세요.

**형식(Signature):**

```typescript
export declare enum NavigationFailureType
```

### 구성(Members)

Member | Value | Description
--- | --- | ---
aborted | 4 | 중단된(aborted) 탐색은 네비게이션 가드가 `false` 를 반환했거나 `next(false)` 를 호출하여 실패한 탐색입니다.
cancelled | 8 | 취소된(cancelled) 탐색은 최근에 완료된 탐색이 시작되었기 때문에 실패한 탐색입니다.(완료된 것은 아님)
duplicated | 16 | 중복(duplicated) 탐색은 이미 정확히 동일한 위치에있는 동안 시작 되었기 때문에 실패한 탐색입니다.

## START_LOCATION

- **Type**: [`RouteLocationNormalized`](#routelocationnormalized)

- **상세**:

    라우터가 있는 초기 라우트 위치입니다. 네비게이션 가드에서 초기 네비게이션을 구분하는 데 사용할 수 있습니다.

    ```js
    import { START_LOCATION } from 'vue-router'

    router.beforeEach((to, from) => {
      if (from === START_LOCATION) {
        // 초기 탐색
      }
    })
    ```

## 컴포지션 API(Composition API)

### onBeforeRouteLeave

현재 위치에 대한 컴포넌트가 남아있을 때마다 트리거되는 네비게이션 가드를 추가합니다. `beforeRouteLeave` 와 유사하지만 모든 컴포넌트에서 사용할 수 있습니다. 컴포넌트가 제거되면(unmounted) 가드가 제거됩니다.

**형식(Signature):**

```typescript
export declare function onBeforeRouteLeave(leaveGuard: NavigationGuard): void
```

#### 매개변수(Parameters)

Parameter | Type | Description
--- | --- | ---
leaveGuard | [`NavigationGuard`](#navigationguard) | 추가 할 네비게이션 가드

### onBeforeRouteUpdate

현재 위치가 업데이트 될 때마다 트리거되는 네비게이션 가드를 추가합니다. `beforeRouteUpdate` 와 유사하지만 모든 컴포넌트에서 사용할 수 있습니다. 컴포넌트가 제거되면(unmounted) 가드가 제거됩니다.

**형식(Signature):**

```typescript
export declare function onBeforeRouteUpdate(updateGuard: NavigationGuard): void
```

#### 매개변수(Parameters)

Parameter | Type | Description
--- | --- | ---
updateGuard | [`NavigationGuard`](#navigationguard) | Navigation guard to add

### useLink

[`v-slot` API](#router-link-s-v-slot) 에 의해 노출 된 모든 것을 반환합니다.

**형식(Signature):**

```typescript
export declare function useLink(props: RouterLinkOptions): {
  route: ComputedRef<RouteLocationNormalized & { href: string }>,
  href: ComputedRef<string>,
  isActive: ComputedRef<boolean>,
  isExactActive: ComputedRef<boolean>,
  navigate: (event?: MouseEvent) => Promise(NavigationFailure | void),
}
```

#### 매개변수(Parameters)

Parameter | Type | Description
--- | --- | ---
props | `RouterLinkOptions` | `<router-link>` 에 전달할 수 있는 props 객체 입니다. `Ref` 및 `ComputedRef` 허용

### useRoute

현재 라우트 위치를 반환합니다. 템플릿 내에서 `$route` 를 사용하는 것과 같습니다. `setup()` 내에서 호출해야합니다.

**형식(Signature):**

```typescript
export declare function useRoute(): RouteLocationNormalized
```

### useRouter

[router](#router-properties) 인스턴스를 반환합니다. 템플릿 내에서 `$router` 를 사용하는 것과 같습니다. `setup()` 내에서 호출해야합니다.

**형식(Signature):**

```typescript
export declare function useRouter(): Router
```

## 타입스크립트

다음은 Vue Router 에서 사용하는 인터페이스 및 타입 중 일부입니다. 문서는 이를 참조하여 객체의 기존 속성에 대한 아이디어를 제공합니다.

## 라우터 속성(Router Properties)

### currentRoute

- **Type**: [`Ref<RouteLocationNormalized>`](#routelocationnormalized)

- **상세**:

    현재 라우트 위치. 읽기 전용.

### options

- **Type**: [`RouterOptions`](#routeroptions)

- **상세**:

    라우터 생성을 위해 전달 된 원본 옵션 객체입니다. 읽기 전용.

## 라우터 메서드(Router Methods)

### addRoute

새 [라우트 레코드(Route Record)](#routerecordraw)를 기존 라우트의 하위로 추가합니다. 라우트에 `name` 이 있고 동일한 경로를 가진 기존 라우트가 이미있는 경우 먼저 제거합니다.

**형식(Signature):**

```typescript
addRoute(parentName: string | symbol, route: RouteRecordRaw): () => void
```

*매개변수(Parameters)*

Parameter | Type | Description
--- | --- | ---
parentName | `string | symbol` | `route` 를 추가해야하는 상위 라우트 레코드
route | [`RouteRecordRaw`](#routerecordraw) | 추가 할 라우트 레코드

### addRoute

새 [라우트 레코드(Route Record)](#routerecordraw)를 기존 라우트의 하위로 추가합니다. 라우트에 `name` 이 있고 동일한 경로를 가진 기존 라우트가 이미있는 경우 먼저 제거합니다.

**형식(Signature):**

```typescript
addRoute(route: RouteRecordRaw): () => void
```

*매개변수(Parameters)*

Parameter | Type | Description
--- | --- | ---
route | [`RouteRecordRaw`](#routerecordraw) | 추가 할 라우트 레코드

::: 팁 라우트를 추가해도 새 탐색이 트리거되지 않습니다. 즉, 새 탐색이 트리거되지 않으면 추가 된 라우트가 표시되지 않습니다. :::

### afterEach

모든 탐색 후에 실행되는 탐색 훅를 추가하십시오. 등록 된 훅을 제거하는 함수를 반환합니다.

**형식(Signature):**

```typescript
afterEach(guard: NavigationHookAfter): () => void
```

*매개변수(Parameters)*

Parameter | Type | Description
--- | --- | ---
guard | `NavigationHookAfter` | 추가 할 탐색 훅

#### 예제

```js
router.afterEach((to, from, failure) => {
  if (isNavigationFailure(failure)) {
    console.log('failed navigation', failure)
  }
})
```

### back

Go back in history if possible by calling `history.back()`. Equivalent to `router.go(-1)`.

**형식(Signature):**

```typescript
back(): void
```

### beforeEach

Add a navigation guard that executes before any navigation. Returns a function that removes the registered guard.

**형식(Signature):**

```typescript
beforeEach(guard: NavigationGuard): () => void
```

*매개변수(Parameters)*

Parameter | Type | Description
--- | --- | ---
guard | [`NavigationGuard`](#navigationguard) | navigation guard to add

### beforeResolve

Add a navigation guard that executes before navigation is about to be resolved. At this state all component have been fetched and other navigation guards have been successful. Returns a function that removes the registered guard.

**형식(Signature):**

```typescript
beforeResolve(guard: NavigationGuard): () => void
```

*매개변수(Parameters)*

Parameter | Type | Description
--- | --- | ---
guard | [`NavigationGuard`](#navigationguard) | navigation guard to add

#### 예제

```js
router.beforeResolve(to => {
  if (to.meta.requiresAuth && !isAuthenticated) return false
})
```

### forward

Go forward in history if possible by calling `history.forward()`. Equivalent to `router.go(1)`.

**Signature:**

```typescript
forward(): void
```

### getRoutes

Get a full list of all the [route records](#routerecord).

**Signature:**

```typescript
getRoutes(): RouteRecord[]
```

### go

Allows you to move forward or backward through the history.

**Signature:**

```typescript
go(delta: number): void
```

*Parameters*

Parameter | Type | Description
--- | --- | ---
delta | `number` | The position in the history to which you want to move, relative to the current page

### hasRoute

Checks if a route with a given name exists

**Signature:**

```typescript
hasRoute(name: string | symbol): boolean
```

*Parameters*

Parameter | Type | Description
--- | --- | ---
name | `string | symbol` | Name of the route to check

### isReady

Returns a Promise that resolves when the router has completed the initial navigation, which means it has resolved all async enter hooks and async components that are associated with the initial route. If the initial navigation already happened, the promise resolves immediately.This is useful in server-side rendering to ensure consistent output on both the server and the client. Note that on server side, you need to manually push the initial location while on client side, the router automatically picks it up from the URL.

**Signature:**

```typescript
isReady(): Promise<void>
```

### onError

Adds an error handler that is called every time a non caught error happens during navigation. This includes errors thrown synchronously and asynchronously, errors returned or passed to `next` in any navigation guard, and errors occurred when trying to resolve an async component that is required to render a route.

**Signature:**

```typescript
onError(handler: (error: any) => any): () => void
```

*Parameters*

Parameter | Type | Description
--- | --- | ---
handler | `(error: any) => any` | error handler to register

### push

Programmatically navigate to a new URL by pushing an entry in the history stack.

**Signature:**

```typescript
push(to: RouteLocationRaw): Promise<NavigationFailure | void | undefined>
```

*Parameters*

Parameter | Type | Description
--- | --- | ---
to | [`RouteLocationRaw`](#routelocationraw) | Route location to navigate to

### removeRoute

Remove an existing route by its name.

**Signature:**

```typescript
removeRoute(name: string | symbol): void
```

*Parameters*

Parameter | Type | Description
--- | --- | ---
name | `string | symbol` | Name of the route to remove

### replace

Programmatically navigate to a new URL by replacing the current entry in the history stack.

**Signature:**

```typescript
replace(to: RouteLocationRaw): Promise<NavigationFailure | void | undefined>
```

*Parameters*

Parameter | Type | Description
--- | --- | ---
to | [`RouteLocationRaw`](#routelocationraw) | Route location to navigate to

### resolve

Returns the [normalized version](#routelocation) of a [route location](#routelocationraw). Also includes an `href` property that includes any existing `base`.

**Signature:**

```typescript
resolve(to: RouteLocationRaw): RouteLocation & {
  href: string
}
```

*Parameters*

Parameter | Type | Description
--- | --- | ---
to | [`RouteLocationRaw`](#routelocationraw) | Raw route location to resolve

## 라우터 옵션(RouterOptions)

### history

라우터에서 사용하는 히스토리 구현입니다. 대부분의 웹 애플리케이션은 `createWebHistory` 를 사용해야하지만 서버를 올바르게 구성해야합니다. `createWebHashHistory`와 함께 *hash* 기반 히스토리를 사용할 수도 있습니다. `createWebHashHistory`는 서버에서 구성이 필요하지 않지만 검색 엔진에서 전혀 처리하지 않고 SEO 에서 제대로 작동하지 않습니다.

**형식(Signature):**

```typescript
history: RouterHistory
```

#### 예제

```js
createRouter({
  history: createWebHistory(),
  // 다른 옵션들...
})
```

### linkActiveClass

활성 [RouterLink](#router-link-props) 에 적용된 기본 CSS클래스입니다. 아무것도 제공되지 않으면 `router-link-active` 가 ​​적용됩니다.

**형식(Signature):**

```typescript
linkActiveClass?: string
```

### linkExactActiveClass

정확한 활성 [RouterLink](#router-link-props) 에 적용되는 기본 CSS클래스입니다. 아무것도 제공되지 않으면 `router-link-exact-active` 가 ​​적용됩니다.

**형식(Signature):**

```typescript
linkExactActiveClass?: string
```

### parseQuery

쿼리를 구문 분석하기위한 사용자 지정 구현입니다. 쿼리 키와 값을 디코딩해야합니다. 상응하는 [stringifyQuery](#stringifyquery) 를 참조하세요.

**형식(Signature):**

```typescript
parseQuery?: (searchQuery: string) => Record<string, (string | null)[] | string | null>
```

#### 예제

[qs](https://github.com/ljharb/qs) 패키지를 사용하여 쿼리를 파싱하려는 경우 `parseQuery` 및 `stringifyQuery` 를 모두 제공 할 수 있습니다.

```js
import qs from 'qs'

createRouter({
  // other options...
  parseQuery: qs.parse,
  stringifyQuery: qs.stringify,
})
```

### routes

라우터에 추가해야하는 라우트 초기 목록입니다.

**형식(Signature):**

```typescript
routes: RouteRecordRaw[]
```

### scrollBehavior

페이지 사이를 이동할 때 스크롤을 제어하는 ​​기능입니다. 스크롤이 발생할 때 지연 프로미스를 반환 할 수 있습니다. 자세한 내용은 [스크롤 동작(Scroll Behaviour)](../guide/advanced/scroll-behavior.md) 을 참조하세요.

**형식(Signature):**

```typescript
scrollBehavior?: RouterScrollBehavior
```

#### 예제

```js
function scrollBehavior(to, from, savedPosition) {
  // `to` and `from` 은 모두 라우트 위치(route locations)
  // `savedPosition` 이 없으면 null 일 수 있습니다.
}
```

### stringifyQuery

쿼리 객체를 문자열로 변환하는 사용자 지정 구현입니다. 앞에 `?` 를 붙이지 말아야 합니다. 쿼리 키와 값을 올바르게 인코딩해야합니다. 쿼리 파싱을 처리하기위한 [parseQuery](#parsequery) 대응

**형식(Signature):**

```typescript
stringifyQuery?: (
  query: Record<
    string | number,
    string | number | null | undefined | (string | number | null | undefined)[]
  >
) => string
```

## 원시 라우트 레코드(RouteRecordRaw)

[`routes` 옵션](#routeroptions) 또는 [`router.addRoute()`](#addroute-2) 를 통해 라우트를 추가 할 때 사용자가 제공 할 수있는 라우트 레코드. 세 가지 종류의 라우트 레코드가 있습니다.

- 단일 뷰 레코드: `component` 옵션이 있습니다.
- 다중 뷰 레코드([명명된 뷰(named views)](../guide/essentials/named-views.md)): `components` 옵션이 있습니다.
- 리다이렉트 레코드: 리다이렉트 레코드에 도달하지 않으므로 `component` 또는 `components` 옵션을 가질 수 없습니다.

### path

- **Type**: `string`

- **상세**:

    레코드 경로(path)입니다. 레코드가 다른 레코드의 하위가 아니면 `/` 로 시작해야합니다. 매개 변수를 정의 할 수 있습니다. `/users/:id` 은 `/users/1 ` 및 `/users/posva` 와 일치합니다.

- **참고**: [동적 라우트 매칭(Dynamic Route Matching)](../guide/essentials/dynamic-matching.md)

### redirect

- **Type**: `RouteLocationRaw | (to: RouteLocationNormalized) => RouteLocationRaw` (선택적)

- **상세**:

    라우트가 직접 일치하는 경우 리다이렉트 할 위치입니다. 리다이렉트는 네비게이션 가드보다 먼저 발생하고 새 대상 위치로 새 탐색을 트리거합니다. 대상 라우트 위치를 수신하고 리다이렉트 해야하는 위치를 반환하는 함수일 수도 있습니다.

### children

- **Type**: Array of [`RouteRecordRaw`](#routerecordraw) (선택적)

- **상세**:

    현재 레코드의 중첩된 라우트.

- **참고**: [중첩된 라우트(Nested Routes)](../guide/essentials/nested-routes.md)

### alias

- **Type**: `string | string[]` (선택적)

- **상세**:

    라우트의 별칭입니다. 레코드 사본처럼 작동하는 추가 경로를 정의 할 수 있습니다. 이렇게하면 `/users/:id` 및 `/u/:id ` 와 같은 경로 축약형을 사용할 수 있습니다. **모든 `alias` 및 `path` 값은 동일한 매개 변수를 공유해야합니다.**

### name

- **Type**: `string | symbol` (선택적)

- **상세**:

    라우트 레코드의 고유 이름.

### beforeEnter

- **Type**: [`NavigationGuard | NavigationGuard[]`](#navigationguard) (선택적)

- **상세**:

    이 레코드에 특정한 가드를 입력하기 전에. 레코드에 `redirect` 속성이있는 경우 `beforeEnter` 효과는 없습니다.

### props

- **Type**: `boolean | Record<string, any> | (to: RouteLocationNormalized) => Record<string, any>` (선택적)

- **상세**:

    `router-view` 에서 렌더링 한 컴포넌트에 매개 변수를 props 로 전달할 수 있습니다. *다중 뷰 레코드* 에 전달되면 각 컴포넌트에 적용 할 `components` 또는 `boolean` 과 동일한 키를 가진 객체 여야합니다. 타겟 위치.

- **참고**: [Route 컴포넌트에 Prop 전달하기(Passing props to Route Components)](../guide/essentials/passing-props.md)

### meta

- **Type**: [`RouteMeta`](#routemeta) (선택적)

- **상세**:

    레코드에 첨부 된 사용자 지정 데이터입니다.

- **참고**: [Meta fields](../guide/advanced/meta.md)

::: 팁 함수형 컴포넌트(functional component)를 사용하려면 `displayName` 을 추가해야합니다.

예제:

```js
const HomeView = () => h('div', 'HomePage')
// TypeScript 에서는 FunctionalComponent 타입을 사용해야합니다.
HomeView.displayName = 'HomeView'
const routes = [{ path: '/', component: HomeView }]
```

:::

## 라우트 레코드 표준화(RouteRecordNormalized)

[라우트 레코드(Route Record)](#routerecordraw)의 표준화 된 버전

### aliasOf

- **Type**: `RouteRecordNormalized | undefined`

- **상세**:

    이 레코드가 다른 레코드의 별칭인지 정의합니다. 레코드가 원래 레코드인 경우 이 속성은 `undefined` 입니다.

### beforeEnter

- **Type**: [`NavigationGuard`](#navigationguard)

- **상세**:

    다른 곳에서 이 레코드를 입력 할 때 네비게이션 가드가 적용됩니다.

- **참고**: [네비게이션 가드(Navigation guards)](../guide/advanced/navigation-guards.md)

### children

- **Type**: 표준화 된 [라우트 레코드(route records)](#routerecordnormalized) 의 배열

- **상세**:

    현재 라우트의 하위 경로 기록. 없는 경우 빈 배열.

### components

- **Type**: `Record<string, Component>`

- **상세**:

    명명된 뷰(named views) 사전(없는 경우)에는 `default` 키가 있는 객체가 포함됩니다.

### meta

- **Type**: `RouteMeta`

- **상세**:

    레코드에 첨부 된 임의의 데이터.

- **참고**: [Meta fields](../guide/advanced/meta.md)

### name

- **Type**: `string | symbol | undefined`

- **상세**:

    라우트 레코드의 이름입니다. 제공되지 않은 경우 `undefined` 입니다.

### path

- **Type**: `string`

- **상세**:

    레코드의 표준화 된 경로입니다. 상위의 `경로(path)` 를 포함합니다.

### props

- **Type**: `Record<string, boolean | Function | Record<string, any>>`

- **상세**:

    각 명명 된 뷰(named view)에 대한 [`props` 옵션](#props)의 사전입니다. 없는 경우 `default` 속성 하나만 포함됩니다.

### redirect

- **Type**: [`RouteLocationRaw`](#routelocationraw)

- **상세**:

    라우트가 직접 일치하는 경우 리다이렉트 할 위치입니다. 리다이렉트는 네비게이션 가드보다 먼저 발생하며 새 대상 위치로 새 탐색을 트리거합니다.

## 원시 라우트 위치(RouteLocationRaw)

`router.push()`, `redirect` 에 전달되고 [Navigation Guards](../guide/advanced/navigation-guards.md) 에서 반환 될 수 있는 사용자 수준 라우트 위치입니다.

원시 위치는 `/users/posva#bio` 와 같은 `string` 또는 객체 일 수 있습니다.

```js
// 이 3가지 형태는 동일함
router.push('/users/posva#bio')
router.push({ path: '/users/posva', hash: '#bio' })
router.push({ name: 'users', params: { username: 'posva' }, hash: '#bio' })
// 해시만 변경
router.push({ hash: '#bio' })
// 쿼리만 변경
router.push({ query: { page: '2' } })
// 하나의 매개변수 변경
router.push({ params: { username: 'jolyne' } })
```

참고 `path` 는 인코딩 된 상태로 제공해야 합니다.<br>(예 : `phantom blood` 는 `phantom%20blood` 가 됨) `params`, `query` 및 `hash` 는 라우터에서 인코딩하면 안됩니다.

원시 라우트 위치는 네비게이션 가드에서 `router.push()` 대신 `router.replace()` 를 호출하는 추가 옵션 `replace` 도 지원합니다. `router.push() ` 를 호출하는 경우에도 내부적으로 `router.replace()` 를 호출합니다.

```js
router.push({ hash: '#bio', replace: true })
// 동일함
router.replace({ hash: '#bio' })
```

## 라우트 위치(RouteLocation)

[redirect records](#routerecordraw) 를 포함 할 수 있는 [RouteLocationRaw](#routelocationraw) 가 해결되었습니다. 그 외에도 [RouteLocationNormalized](#routelocationnormalized) 와 동일한 속성을 가집니다.

## 라우트 위치 표준화(RouteLocationNormalized)

표준화 된 라우트 위치. [redirect records](#routerecordraw) 는 없습니다. 네비게이션 가드에서 `to` 및 `from` 은 항상 이 유형입니다.

### fullPath

- **Type**: `string`

- **상세**:

    라우트 위치에 연결된 인코딩 된 URL입니다.`path`, `query` 와 `hash` 를 포함

### hash

- **Type**: `string`

- **상세**:

    URL의 `hash` 부분이 디코딩 되었습니다. 항상 `#` 으로 시작합니다. URL에 `hash` 가 없으면 빈 문자열입니다.

### query

- **Type**: `Record<string, string | string[]>`

- **상세**:

    URL의 `search` 브분에서 추출 된 디코딩 된 쿼리 매개변수의 사전입니다.

### matched

- **Type**: [`RouteRecordNormalized[]`](#routerecordnormalized)

- **상세**:

    지정된 라우트 위치와 일치하는 [normalized route records](#routerecord)의 배열입니다.

### meta

- **Type**: `RouteMeta`

- **상세**:

    일치하는 모든 레코드에 첨부 된 임의 데이터가 부모에서 자식으로 병합(비재귀적 : non recursively)됩니다.

- **참고**: [Meta fields](../guide/advanced/meta.md)

### name

- **Type**: `string | symbol | undefined | null`

- **상세**:

    라우트 레코드의 이름입니다. 제공되지 않은 경우 `undefined`.

### params

- **Type**: `Record<string, string | string[]>`

- **상세**:

    `path`에서 추출 된 디코딩 된 매개변수의 사전입니다.

### path

- **Type**: `string`

- **상세**:

    라우트 위치에 연결된 URL의 인코딩 된 `pathname` 부분입니다.

### redirectedFrom

- **Type**: [`RouteLocation`](#routelocation)

- **상세**:

    `redirect` 옵션이 발견되거나 라우트 위치가 있는 `next()` 라는 내비게이션 가드가 발견되었을 때 현재 위치에 도달하기 전에 처음 접근 하려고 했던 라우트 위치입니다. 리다이렉트가 없는 경우 `undefined` 입니다.

## 탐색실패(NavigationFailure)

### from

- **Type**: [`RouteLocationNormalized`](#routelocationnormalized)

- **상세**:

    현재 라우트 경로

### to

- **Type**: [`RouteLocationNormalized`](#routelocationnormalized)

- **상세**:

    탐색할 라우트 위치

### type

- **Type**: [`NavigationFailureType`](#navigationfailuretype)

- **상세**:

    탐색실패(navigation failure) 유형(NavigationFailureType.redirected, NavigationFailureType.cancelled 등)

- **참고**: [탐색실패(Navigation Failures)](../guide/advanced/navigation-failures.md)

## 네비게이션 가드(NavigationGuard)

- **인자**:

    - [`RouteLocationNormalized`](#routelocationnormalized) to - 탐색할 라우트 위치
    - [`RouteLocationNormalized`](#routelocationnormalized) from - 현재 라우트 위치
    - `Function` next (Optional) - 탐색을 승인하기 위한 콜백

- **상세**:

    라우터 탐색을 제어하기 위해 전달할 수 있는 기능입니다. 대신 값 (또는 Promise)을 반환하는 경우 `next` 콜백을 생략 할 수 있습니다. 가능한 반환 값 (및 `next`의 매개 변수)은 다음과 같습니다.

    - `undefined | void | true`: 탐색을 승인합니다.
    - `false`: 탐색을 취소합니다.
    - [`RouteLocationRaw`](#routelocationraw): 다른 위치로 리다이렉트 합니다.
    - `(vm: ComponentPublicInstance) => any` **`beforeRouteEnter` 만 해당**: 탐색이 완료되면 실행할 콜백입니다. 라우트 컴포넌트 인스턴스를 매개변수로 받습니다.

- **참고**: [네비게이션 가드(Navigation Guards)](../guide/advanced/navigation-guards.md)

## 컴포넌트 주입

### 컴포넌트 주입 속성

이러한 속성은 `app.use(router)` 를 호출하여 모든 하위 컴포넌트에 주입됩니다.

- **this.$router**

    라우터 인스턴스

- **this.$route**

    현재 활성 [route location](#routelocationnormalized) 속성 입니다. 이 속성은 읽기 전용이며 속성은 변경할 수 없지만 관찰 할 수 있습니다.

### 컴포넌트 활성화 옵션

- **beforeRouteEnter**
- **beforeRouteUpdate**
- **beforeRouteLeave**

[In Component Guards](../guide/advanced/navigation-guards.md#in-component-guards) 참고
