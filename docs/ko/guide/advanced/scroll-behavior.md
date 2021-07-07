# 스크롤 동작

클라이언트 측 라우팅을 사용할 때 새 경로로 네비게이션 할때  맨 위로 스크롤 하거나 기존 페이지 리로딩처럼 스크롤 위치를 히스트리 이력으로 저장해두었다가 스크롤 위치를 복구 할수 있습니다.  Vue Router를 사용하면 이러한 작업을 더 잘 수행 할 수 합니다. 또한  경로 네비게이션에서 스크롤 동작을 완전히 커스터마이징 할 수 있습니다.

**참고 :이 기능은 브라우저가 `history.pushState`를  지원하는 경우에만 작동합니다.**

라우터 인스턴스를 만들 때 `scrollBehavior` 함수를 설정 할 수 있습니다.

```js
const router = createRouter({
  history: createWebHashHistory(),
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    // return desired position
  }
})
```

`scrollBehavior` 함수는 다른 [네비게이션 가드](./navigation-guards.md)처럼 함수 인자로  `to` 과 `from`를 넘겨 받습니다. 세 번째 함수 인자로  `savedPosition`를 넘겨 받을수 있는데 이 인자는  브라우자 앞으로가기/뒤로가기 버튼을 눌러 히스토리 이력에서 `popstate`를 받아 왔을때만 넘겨 받습니다.

이 함수는 [`ScrollToOptions`](https://developer.mozilla.org/en-US/docs/Web/API/ScrollToOptions) 위치 객체를 반환 할 수 있습니다.

```js
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    // 언제나 페이지 제일 위로 이동하기
    return { top: 0 }
  },
})
```

`el` 통해 CSS 셀렉터나  DOM 앨리먼트를  전달할 수도 있습니다. 그럴 경우  `top` 과 `left` 은 해당 요소에 대한 상대적 오프셋으로 처리됩니다.

```js
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    // 언제나 #main으로 부터 10px 만큼 위로 스크롤 시킨다
    return {
      // could also be
      // el: document.getElementById('main'),
      el: '#main',
      top: -10,
    }
  },
})
```

잘못된 값이나 빈 개체가 반환되면 스크롤이 발생하지 않습니다.

`savedPosition` 을 반환하면 뒤로가기 / 앞으로가기 버튼으로 이동할떄  네이티브와 유사하게 동작합니다.

```js
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    } else {
      return { top: 0 }
    }
  },
})
```

"링크로 스크롤하기" 동작을 흉내낼때

```js
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    if (to.hash) {
      return {
        el: to.hash,
      }
    }
  },
})
```

브라우저가 [스크롤 동작](https://developer.mozilla.org/en-US/docs/Web/API/ScrollToOptions/behavior)을 지원하는 경우 부드럽게 만들 수 있습니다.

```js
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    if (to.hash) {
      return {
        el: to.hash,
        behavior: 'smooth',
      }
    }
  }
})
```

## 스크롤 지연

때로는 페이지를 스크롤하기 전에 잠시 기다려야합니다. 예를 들어 트랜지션을 처리 할 때 스크롤하기 전에 트랜지션이 완료 될 때까지 기다립니다. 이를 위해 원하는 위치 설명자를 반환하는 Promise를 반환 할 수 있습니다. 다음은 스크롤하기 전에 500ms를 기다리는 예입니다.

```js
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve({ left: 0, top: 0 })
      }, 500)
    })
  },
})
```

페이지 레벨을 구현하는 컴포넌트의 이벤트를 받아서 각 페이지 컴포넌트의 전환와 스크롤 동작을 잘 동작하게 할수 있습니다. 하지만 다양한 사용예와 복잡성 때문에 단순 구현만 제공하고, 그 외에는직접  구현을 해서 사용할수 있게 하였습니다.
