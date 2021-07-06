# 트랜지션(Transitions)

<vueschoollink href="https://vueschool.io/lessons/route-transitions" title="Learn about route transitions"></vueschoollink>

route 컴포넌트에서 트랜지션을 사용하고 탐색을 애니메이션 처리 하려면 [v-slot API](../../api/#router-view-s-v-slot) 를 사용합니다.

```html
<router-view v-slot="{ Component }">
  <transition name="fade">
    <component :is="Component" />
  </transition>
</router-view>
```

[All transition APIs](https://v3.vuejs.org/guide/transitions-enterleave.html) work the same here.

## 경로별 트랜지션(Per-Route Transition)

The above usage will apply the same transition for all routes. If you want each route's component to have different transitions, you can instead combine [meta fields](./meta.md) and a dynamic `name` on `<transition>`:

```js
const routes = [
  {
    path: '/custom-transition',
    component: PanelLeft,
    meta: { transition: 'slide-left' },
  },
  {
    path: '/other-transition',
    component: PanelRight,
    meta: { transition: 'slide-right' },
  },
]
```

```html
<router-view v-slot="{ Component, route }">
  <!-- 사용자지정 트랜지션 및 대체 트랜지션은 `fade` 사용 -->
  <transition :name="route.meta.transition || 'fade'">
    <component :is="Component" />
  </transition>
</router-view>
```

## 경로 기반 동적 트랜지션(Route-Based Dynamic Transition)

대상 경로와 현재 경로 사이의 관계에 따라 동적으로 사용할 트랜지션을 결정할 수도 있습니다. <br>직전과 매우 유사한 예시 사용 :

```html
<!-- 동적 트랜지션 이름을 사용 -->
<router-view v-slot="{ Component, route }">
  <transition :name="route.meta.transitionName">
    <component :is="Component" />
  </transition>
</router-view>
```

경로의 깊이에 따라 `meta` 필드에 정보를 동적으로 추가하기 위해 [after navigation hook](./navigation-guards.md#global-after-hooks) 을 추가 할 수 있습니다.

```js
router.afterEach((to, from) => {
  const toDepth = to.path.split('/').length
  const fromDepth = from.path.split('/').length
  to.meta.transitionName = toDepth < fromDepth ? 'slide-right' : 'slide-left'
})
```

<!-- TODO: interactive example -->

<!-- See full example [here](https://github.com/vuejs/vue-router/blob/dev/examples/transitions/app.js). -->
