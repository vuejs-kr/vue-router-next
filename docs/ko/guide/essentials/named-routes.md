# 명명 된 경로


`path` 와 함께 모든 `name` 을 제공 할 수 있습니다. 이것은 다음과 같은 장점이 있습니다.

- 하드 코딩 된 URL 없음
- `params` 자동 인코딩 / 디코딩
- URL에 오타가 발생하지 않도록 방지
- 경로 순위 우회 (ex a 를 표시)

```js
const routes = [
  {
    path: '/user/:username',
    name: 'user',
    component: User
  }
]
```

명명된 경로에 링크하기 위해 `router-link` 컴포넌트에 `to` prop에 객체를 줄수 있습니다.

```html
<router-link :to="{ name: 'user', params: { username: 'erina' }}">
  User
</router-link>
```

`router.push()` 와 함께 프로그래밍 방식으로 사용되는 것과 똑같은 객체입니다.

```js
router.push({ name: 'user', params: { username: 'erina' } })
```

두 경우 모두 라우터는 `/user/erina` 경로로 이동합니다.

[여기에](https://github.com/vuejs/vue-router/blob/dev/examples/named-routes/app.js) 전체 예가 있습니다.
