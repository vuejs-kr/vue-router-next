# 다양한 히스토리 모드

<vueschoollink href="https://vueschool.io/lessons/history-mode" title="Learn about the differences between Hash Mode and HTML5 Mode"></vueschoollink>

`history` 옵션을 사용하면 다른 히스토리 모드 중에서 선택할 수 있습니다.

## 해시 모드

해시 히스토리 모드는 `createWebHashHistory()` 를 이용해 만듭니다.

```js
import { createRouter, createWebHashHistory } from 'vue-router'

const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    //...
  ],
})
```

내부적으로 전달되는 실제 URL 앞에 해시 문자 ( `#`) 이 URL 섹션은 서버로 전송되지 않기 때문에 서버 수준에서 특별한 처리가 필요하지 않습니다. **그러나 이렇게 하면 SEO에 나쁜 영향을 미칩니다** . 검색엔진최적화가  걱정된다면 HTML5 히스토리 모드를 사용하세요.

## HTML5 Mode

HTML5 모드는 `createWebHistory()`를 이용해 만듭니다. 이 모드가 추천되는 모드입니다.

```js
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    //...
  ],
})
```

히스토리 모드를 사용할 때 URL은 `https://example.com/user/id` 와 같이 "정상"으로 보입니다. 멋지죠!

하지만 여기에 문제가 있습니다. 우리 앱은 적절한 서버 구성이없는 단일 페이지 클라이언트 측 앱이므로 사용자가 브라우저에서 `https://example.com/user/id` 을 입력하여 접속하면 404페이지를 보게 됩니다.

걱정하지 마세요. 문제를 해결하려면 서버에 간단한 포괄 대체 경로를 추가하기 만하면됩니다. URL이 정적 리소스와 일치하지 않는 경우 `index.html` 페이지를 제공하면 됩니다. 멋지조!

## 서버 구성 예제

**참고** : 다음 예제에서는 루트 폴더에서 앱을 제공한다고 가정합니다. 하위 폴더에 배포하는 경우 [Vue CLI의 `publicPath`](https://cli.vuejs.org/config/#publicpath) 옵션과 라우터의 관련 [`base` 속성을](../../api/#createwebhistory) 사용해야합니다. 또한 루트 폴더 대신 하위 폴더를 사용하려면 아래 예제를 조정해야합니다 (예 : `RewriteBase /` 를 `RewriteBase /name-of-your-subfolder/` 대체).

### Apache

```apacheconf
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

`mod_rewrite` 대신 [`FallbackResource`](https://httpd.apache.org/docs/2.2/mod/mod_dir.html#fallbackresource) 를 사용할 수도 있습니다.

### nginx

```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

### Native Node.js

```js
const http = require('http')
const fs = require('fs')
const httpPort = 80

http
  .createServer((req, res) => {
    fs.readFile('index.html', 'utf-8', (err, content) => {
      if (err) {
        console.log('We cannot open "index.html" file.')
      }

      res.writeHead(200, {
        'Content-Type': 'text/html; charset=utf-8',
      })

      res.end(content)
    })
  })
  .listen(httpPort, () => {
    console.log('Server listening on: http://localhost:%s', httpPort)
  })
```

### Express with Node.js

Node.js / Express의 경우 [connect-history-api-fallback 미들웨어](https://github.com/bripkens/connect-history-api-fallback) 사용을 고려하십시오.

### Internet Information Services (IIS)

1. [IIS UrlRewrite](https://www.iis.net/downloads/microsoft/url-rewrite) 설치
2. 다음을 사용하여 사이트의 루트 디렉터리에 `web.config`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="Handle History Mode and custom 404/500" stopProcessing="true">
          <match url="(.*)" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Rewrite" url="/" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

### Caddy v2

```
try_files {path} /
```

### Caddy v1

```
rewrite {
    regexp .*
    to {path} /
}
```

### Firebase hosting

`firebase.json`에  추가하십시오.

```json
{
  "hosting": {
    "public": "dist",
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
```

### Netlify

배포 된 파일에 포함 된 `_redirects` 파일을 만듭니다.

```
/* /index.html 200
```

vue-cli, nuxt 및 vite 프로젝트에서이 파일은 일반적으로 `static` 또는 `public` 이라는 폴더 아래에 있습니다.

You can more about the syntax on [Netlify documentation](https://docs.netlify.com/routing/redirects/rewrites-proxies/#history-pushstate-and-single-page-apps). You can also [create a `netlify.toml`](https://docs.netlify.com/configure-builds/file-based-configuration/) to combine *redirections* with other Netlify features.

## 경고

이에 대한주의 사항이 있습니다. 찾을 수없는 모든 경로가 이제 `index.html` 파일을 제공하므로 서버에서 더 이상 404 오류를보고하지 않습니다. 이 문제를 해결하려면 404 페이지를 표시하도록 Vue 앱 내에서 포괄 경로를 구현해야합니다.

```js
const router = createRouter({
  history: createWebHistory(),
  routes: [{ path: '/:pathMatch(.*)', component: NotFoundComponent }],
})
```

또는 Node.js 서버를 사용하는 경우 서버 측의 라우터를 사용하여 수신 URL을 일치시키고 일치하는 경로가 없으면 404로 응답하여 대체를 구현할 수 있습니다. 자세한 내용은 [Vue 서버 측 렌더링 문서](https://v3.vuejs.org/guide/ssr/introduction.html#what-is-server-side-rendering-ssr) 를 확인하세요.
