# 명명 된 뷰

때로는 여러 뷰를 중첩하지 않고 동시에 표시해야합니다 (예 : `sidebar`  뷰및 `main` 뷰 가 있는 레이아웃 만들기). 이것은 명명 된 뷰가 유용한 곳입니다. 뷰에 하나의 콘센트를 사용하는 대신 여러 개를 갖고 각각에 이름을 지정할 수 있습니다. 이름을 주지 않은 `router-view` 는 `default` 이 기본값으로 지정됩니다.

```html
<router-view class="view left-sidebar" name="LeftSidebar"></router-view>
<router-view class="view main-content"></router-view>
<router-view class="view right-sidebar" name="RightSidebar"></router-view>
```

뷰는 구성 요소를 사용하여 렌더링되므로 여러 뷰에는 동일한 경로에 대해 여러 구성 요소가 필요합니다. `components` ( **s 사용** ) 옵션을 사용해야합니다.

```js
const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    {
      path: '/',
      components: {
        default: Home,
        // short for LeftSidebar: LeftSidebar
        LeftSidebar,
        // they match the `name` attribute on `<router-view>`
        RightSidebar,
      },
    },
  ],
})
```

이 예제의 작동 데모는 [여기](https://codesandbox.io/s/named-views-vue-router-4-examples-rd20l) 에서 찾을 수 있습니다.

## 중첩 된 명명 된 뷰

중첩 된 뷰가있는 명명 된 뷰를 사용하여 복잡한 레이아웃을 만들 수 있습니다. 그렇게 할 때 중첩 된 `router-view` 에 이름을 지정해야합니다. 예를 들면 설정 패널이 있습니다.

```
/settings/emails                                       /settings/profile
+-----------------------------------+                  +------------------------------+
| UserSettings                      |                  | UserSettings                 |
| +-----+-------------------------+ |                  | +-----+--------------------+ |
| | Nav | UserEmailsSubscriptions | |  +------------>  | | Nav | UserProfile        | |
| |     +-------------------------+ |                  | |     +--------------------+ |
| |     |                         | |                  | |     | UserProfilePreview | |
| +-----+-------------------------+ |                  | +-----+--------------------+ |
+-----------------------------------+                  +------------------------------+
```

- `Nav` 는 일반적인 컴포넌트입니다.
- `UserSettings` 는 부모 뷰 컴포넌트입니다.
- `UserEmailsSubscriptions` , `UserProfile` , `UserProfilePreview` 는 중첩 된 뷰 컴포넌트입니다.

**참고** : *HTML / CSS가 이러한 레이아웃을 표현하고 사용 된 구성 요소에 초점을 맞추기 위해 어떻게 생겼는지 잊어 버리겠습니다.*

`UserSettings`의  `<template>` 섹션은 다음과 같습니다.

```html
<!-- UserSettings.vue -->
<div>
  <h1>User Settings</h1>
  <NavBar />
  <router-view />
  <router-view name="helper" />
</div>
```

그런 다음이 경로 구성으로 위의 레이아웃을 얻을 수 있습니다.

```js
{
  path: '/settings',
  // 상단에 명명 된 뷰가있을 수도 있습니다.
  component: UserSettings,
  children: [{
    path: 'emails',
    component: UserEmailsSubscriptions
  }, {
    path: 'profile',
    components: {
      default: UserProfile,
      helper: UserProfilePreview
    }
  }]
}
```

이 예제의 작동 데모는 [여기](https://codesandbox.io/s/nested-named-views-vue-router-4-examples-re9yl?&initialpath=%2Fsettings%2Femails) 에서 찾을 수 있습니다.
