# 데이터 가져오기

때로는 경로가 활성화 될 때 서버에서 데이터를 가져와야합니다. 예를 들어 사용자 프로필을 렌더링하기 전에 서버에서 사용자 데이터를 가져와야합니다. 두 가지 방법으로 이를 달성 할 수 있습니다.

- **탐색 후 가져 오기** : 먼저 탐색을 수행하고 들어오는 구성 요소의 수명주기 후크에서 데이터를 가져옵니다. 데이터를 가져 오는 동안 로딩 상태를 표시합니다.

- **내비게이션 전 가져 오기** : 경로에서 내비게이션 전 데이터를 가져와 가드에 진입하고 데이터를 가져온 후 내비게이션을 수행합니다.

기술적으로는 둘 다 유효한 선택이며 궁극적으로 목표로하는 사용자 경험에 따라 다릅니다.

## 탐색 후 가져 오기

이 접근 방식을 사용하면 들어오는 구성 요소를 즉시 탐색 및 렌더링하고 구성 요소의 `created` 후크에서 데이터를 가져옵니다. 네트워크를 통해 데이터를 가져 오는 동안 로딩 상태를 표시 할 수있는 기회를 제공하며, 각 뷰에 대해 로딩을 다르게 처리 할 수도 있습니다.

`Post` 컴포넌트가 `$route.params.id` 기반으로 한 게시물에  데이터를 가져와야  한다고 가정 해 보겠습니다.

```html
<template>
  <div class="post">
    <div v-if="loading" class="loading">Loading...</div>

    <div v-if="error" class="error">{{ error }}</div>

    <div v-if="post" class="content">
      <h2>{{ post.title }}</h2>
      <p>{{ post.body }}</p>
    </div>
  </div>
</template>
```

```js
export default {
  data() {
    return {
      loading: false,
      post: null,
      error: null,
    }
  },
  created() {
    // 라우트 파라메터의 변경을 watch하여 변경시 다시 가져온다
    this.$watch(
      () => this.$route.params,
      () => {
        this.fetchData()
      },
      // 뷰가 만들어질때 즉시 데이터를 가져오고 파라메터 변경을 추적한다.
      { immediate: true }
    )
  },
  methods: {
    fetchData() {
      this.error = this.post = null
      this.loading = true
      // replace `getPost` with your data fetching util / API wrapper
      getPost(this.$route.params.id, (err, post) => {
        this.loading = false
        if (err) {
          this.error = err.toString()
        } else {
          this.post = post
        }
      })
    },
  },
}
```

## 탐색 전 가져 오기

이 접근 방식을 사용하면 실제로 새 경로로 이동하기 전에 데이터를 가져옵니다. 들어오는 구성 요소 `beforeRouteEnter` 가드에서 데이터 가져 오기를 수행 할 수 있으며,  가져 오기가 완료 되면  `next`를 호출하면 됩니다.

```js
export default {
  data() {
    return {
      post: null,
      error: null,
    }
  },
  beforeRouteEnter(to, from, next) {
    getPost(to.params.id, (err, post) => {
      next(vm => vm.setData(err, post))
    })
  },
  // when route changes and this component is already rendered,
  // the logic will be slightly different.
  async beforeRouteUpdate(to, from) {
    this.post = null
    try {
      this.post = await getPost(to.params.id)
    } catch (error) {
      this.error = error.toString()
    }
  },
}
```

새로 보여질 뷰에서 데이터를 가져 오는 동안 이전 뷰가 유지됩니다. 따라서 데이터를 가져 오는 동안 진행상태나 적절한 표시기를 보여주는게 좋습니다.  데이터 가져 오기에 실패하면 적절한  전역 경고 메시지도 표시해야합니다.

<!-- ### Using Composition API -->

<!-- TODO: -->
