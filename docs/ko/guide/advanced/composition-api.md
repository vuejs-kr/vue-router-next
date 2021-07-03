# Vue Router 와 컴포지션  API

Vue에서 `setup` 및  [Composition API](https://v3.vuejs.org/guide/composition-api-introduction.html) 이 도입되어서 새로운 가능성이 열렸습니다.  vue 라우터에서 이를 최대한 활용하기 위해서 `this` 를 대신하여 컴포넌트간의 네비게이션을 수행하는 새로운 함수를 사용해야 합니다.

## `setup` 안에서 라우터와 라우트(경로)에 접근하기

컴포지션 API 기반의 `setup` 함수 안에서는  `this`를 사용하면 않됩니다. 따라서  `this.$router` 나  `this.$route`를 사용할수 없습니다.  그대신 `useRouter` 함수를 사용해야 합니다.

```js
import { useRouter, useRoute } from 'vue-router'

export default {
  setup() {
    const router = useRouter()
    const route = useRoute()

    function pushWithQuery(query) {
      router.push({
        name: 'search',
        query: {
          ...route.query,
        },
      })
    }
  },
}
```

`route` 객체는 반응형 객체입니다. 라우트 객체의 속성을 watch하는 것은 좋지만, <strong data-md-type="double_emphasis">`route` 자체를 watch 하는 것은 피하는게 좋습니다.</strong> 대부분의 시나리오에서는 변경될 파라메터를 직접 watch할수 있습니다.

```js
import { useRoute } from 'vue-router'
import { ref } from 'vue'

export default {
  setup() {
    const route = useRoute()
    const userData = ref()

    // fetch the user information when params change
    watch(
      () => route.params.id,
      async newId => {
        userData.value = await fetchUser(newId)
      }
    )
  },
}
```

Note we still have access to `$router` and `$route` in templates, so there is no need to return `router` or `route` inside of `setup`.

## 네비게이션 가드

예전 방식대로 컴포넌트 내 네비게이션 가드 선언 방식을 사용할수도 있습니다. 하지만 `setup` 내에서  컴포지션 API 함수를 이용해 네비게이션 가드를 이용할수 있습니다.

```js
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'
import { ref } from 'vue'

export default {
  setup() {
    // `this` 업이도 beforeRouteLeave 옵션을 사용할수 있습니다.
    onBeforeRouteLeave((to, from) => {
      const answer = window.confirm(
        'Do you really want to leave? you have unsaved changes!'
      )
      // 네비게이션을 취소하고 현재 페이지에 머무를수 있습니다.
      if (!answer) return false
    })

    const userData = ref()

    // `this` 없이도 onBeforeRouteUpdate 옵션을 지정할수 있습니다.
    onBeforeRouteUpdate(async (to, from) => {
      //URL상의 쿼리나 해시가 변경되어 사용자 ID가 변경되었을때만 가져오게 한다.
      if (to.params.id !== from.params.id) {
        userData.value = await fetchUser(to.params.id)
      }
    })
  },
}
```

Composition API guards can also be used in any component rendered by `<router-view>`, they don't have to be used directly on the route component like in-component guards.

## `useLink`

Vue Router는 RouterLink의 내부 동작을 Composition API 함수로 노출합니다. [`v-slot` API](../../api/#router-link-s-v-slot) 와 동일한 속성에 접근 할수 있습니다.

```js
import { RouterLink, useLink } from 'vue-router'
import { computed } from 'vue'

export default {
  name: 'AppLink',

  props: {
    // add @ts-ignore if using TypeScript
    ...RouterLink.props,
    inactiveClass: String,
  },

  setup(props) {
    const { route, href, isActive, isExactActive, navigate } = useLink(props)

    const isExternalLink = computed(
      () => typeof props.to === 'string' && props.to.startsWith('http')
    )

    return { isExternalLink, href, navigate, isActive }
  },
}
```
