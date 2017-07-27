---
layout: post
title: "angular2 Routes Lazy Loading 적용 시 문제상황"
date: 2017-02-01
categories: frontend
tags: [angular, route, lazyloading]
comment: yes
---

angular2 app routes lazy loading 시도 중 몇가지 트러블슈팅


lazy loading의 적용은 어렵지 않은데 적용 후 빌드를 해보면
자꾸 `Cannot find module './datasource/editor/datasource-editor.module’.` 라고 나온다.

좀 찾아보니 subfolder 안에 있는 모듈에 대해서 문제가 좀 있는 모양인데
규모가 커지면 서브폴더 하나 안두는 프로젝트가 어디있으려나 ㅡㅡ

알아보는 중

- `./datasource/editor/datasource-editor.module` (x)
- `/datasource/editor/datasource-editor.module` (x)
- `datasource/editor/datasource-editor.module` (x)
- `app/datasource/editor/datasource-editor.module` (x)
- `/app/datasource/editor/datasource-editor.module` (x)

경로의 문제가 아닌거 같다

좀 찾아보니
webpack 설정에

```
module.exports = {
    module: {
        loaders: [
            {
                test: /\.ts$/,
                loaders: [
                    'awesome-typescript-loader',
                    'angular2-template-loader',
                    'angular2-router-loader'
                ]
            }
        ]
    }
}
```

를 추가하라길래 옳거니하고 넣어봤더니 또 안된다. ㅜㅜ


[https://github.com/brandonroberts/angular-router-loader](https://github.com/brandonroberts/angular-router-loader)
여기에는

```
NOTE: When specifying a relative path to lazy loaded module, one of the following two conditions must hold:

The routes are defined in the same module file where it is imported with RouterModule.forRoot or RouterModule.forChild
The routes are defined in a separate routing file, and that routing file is a sibling of module file.
```

라는데 무슨말인지 잘모르겠다

---


오오 드디어 해결

알고봤더니 프로젝트의 webpack 설정파일에

```
    module: {
      noParse: [
        /es6-shim/,
        /reflect-metadata/,
        /zone\.js(\/|\\)dist(\/|\\)zone-microtask/
      ],
      rules: [
        {
          test: /\.ts$/,
          use: ['awesome-typescript-loader', 'angular2-router-loader', 'angular2-template-loader'],
          exclude: [/node_modules\/(?!(ng2-.+))/]
        },
      ]
    },
```

이렇게 설정이 되어있었는데 angular2-router-loader는 추가가 안되어있었다.
추가하니까 app/blahblah 경로가 잘 먹힌다!!

근데 loaders와 rules 세팅의 차이는 뭔지…
웹팩 공부를 해야할거같다..


빌드하니 lazy loading 관련한 로그들이 같이 뜨기 시작한다.

```
[angular2-router-loader]: --DEBUG--
[angular2-router-loader]: File: /Users/closer27/Devs/ds_dev_team/poc-front-end/src/public/main-web/app/app.routing.module.ts [angular2-router-loader]: Original: loadChildren: 'app/datasource/editor/datasource-editor.module#DatasourceEditorModule'
[angular2-router-loader]: Replacement: loadChildren: () => new Promise(function (resolve) { (require as any).ensure([], function (require: any) { resolve(require('app/datasource/editor/datasource-editor.module')['DatasourceEditorModule']); 1);})
[angular2-router-loader]: --DEBUG--
[angular2-router-loader]: File: /Users/closer27/Devs/ds_dev_team/poc-front-end/src/public/main-web/app/app.routing.module.ts [angular2-router-loader]: Original: loadChildren: './shared/error-page/error-page.module#ErrorPageComponentModule'
[angular2-router-loader]: Replacement: loadChildren: () => new Promise(function (resolve) { (require as any).ensure([], function (require: any) { resolve(require('./shared/error-page/error-page. module')['ErrorPageComponentModule']); 1);})
chunk {0} main-web/app/vendor.2fc8b9873842bee63def.bundle.js (vendor) 6.47 MB {2} [initial] [rendered]
chunk {1} main-web/app/main.f41f3lcd5888dbc22bla.bundleAs (main) 1.04 MB {0} [initial] [rendered]
chunk {2} main-web/app/polyfills.698e192b12bd8ed64061.bundle.js (polyfills) 519 kB [entry] [rendered]

```


---

근데 적용을하고나니 문제가 루트로 접속해서 들어갔는데 엉뚱하게 lazy loading을 적용한 컴포넌트 페이지로 연결이 된다. 이건 또 무슨 상황이래

음 뭔가 export const AppRoutingModule 대신 다시 원래대로 class로 바꼈더니 잘된다.


---

그럼 이제 lazy loading을 통해서 모듈을 나중에 불러오는거니까 app.module에서 모듈 임포트를 지워도 되겠지? 싶어서 지웠더니

```
error_handler.js:51 EXCEPTION: Uncaught (in promise): Error: Loading chunk 3 failed.
Error: Loading chunk 3 failed.
    at HTMLScriptElement.onScriptComplete (http://localhost:3100/main-web/app/polyfills.698e192b12bd8ed64061.bundle.js:92:33)
    at HTMLScriptElement.wrapFn (webpack:///./~/zone.js/dist/zone.js?:698:29)
    at ZoneDelegate.invokeTask (webpack:///./~/zone.js/dist/zone.js?:265:35)
    at Object.onInvokeTask (webpack:///./~/@angular/core/src/zone/ng_zone.js?:262:37)
    at ZoneDelegate.invokeTask (webpack:///./~/zone.js/dist/zone.js?:264:40)
    at Zone.runTask (webpack:///./~/zone.js/dist/zone.js?:154:47)
    at HTMLScriptElement.ZoneTask.invoke (webpack:///./~/zone.js/dist/zone.js?:335:33)
```

저런 식으로 에러가 나온다. 아마 모듈 임포트가 안되어있으면 컴파일을 안하니까 청크가 생성이 안된다는 건가?.. 의미를 잘모르겠다.
아.. 각 컴포넌트 모듈에 RouterModule.forChild(routes)를 임포트 안시켜서 그런가보다

---

만약 한 모듈 안에 두 컴포넌트가 있고 이를 각각을 라우팅해서 사용하고 있었다면 어떻게 설정해줘야할까?

---

적용은 잘된거같은데 이게 제대로 된건지 사실 잘모르겠다.
공식 문서에 보면 lazy loading의 원리는

```
Lazy loading speeds up our application load time by splitting it into multiple bundles, and loading them on demand. The router is designed to make lazy loading simple and easy. Instead of providing the children property, you can provide the loadChildren property, as follows:
```

결국은 번들을 여러개로 쪼개서 각각을 요청이 있을 때마다 로딩을 한다는건데. 네트워크쪽 디버깅을 해보면 번들파일들이 쪼개진거 같아보이진 않는다.

---

``` typescript
{
    path: 'target-editor',
    loadChildren: 'app/target/editor/target-editor.module#TargetEditorModule'
}
```
이렇게 써줬을 때 안되고

``` typescript
{
    path: 'target-editor',
    loadChildren: 'app/target/editor/index#TargetEditorModule'
}
```
이렇게 index를 참조해주니까 chunk가 생성 된다.

---

network 디버그에서 chunk는 생성 되는데 호출이 안되는 경우
```
Uncaught (in promise): Error: Loading chunk 4 failed.
Error: Loading chunk 4 failed.
    at HTMLScriptElement.onScriptComplete (http://localhost:3100/main-web/app/polyfills.698e192b12bd8ed64061.bundle.js:92:33)
    at HTMLScriptElement.wrapFn (webpack:///./~/zone.js/dist/zone.js?:698:29)
    at ZoneDelegate.invokeTask (webpack:///./~/zone.js/dist/zone.js?:265:35)
    at Object.onInvokeTask (webpack:///./~/@angular/core/src/zone/ng_zone.js?:262:37)
    at ZoneDelegate.invokeTask (webpack:///./~/zone.js/dist/zone.js?:264:40)
    at Zone.runTask (webpack:///./~/zone.js/dist/zone.js?:154:47)
    at HTMLScriptElement.ZoneTask.invoke (webpack:///./~/zone.js/dist/zone.js?:335:33)
```

webpack.common.js에 설정에
```
use: ['awesome-typescript-loader', 'angular2-router-loader', 'angular2-template-loader'],
```
이런 식으로 router-loader가 끝에 없고 중간에 오는지 확인해보자.

순서가 상관 없는 줄 알았는데
awesome-typescript-loader가 끝나고
angular2-template-loader가 실행 되고
그다음에 angular2-router-loader가 실행되는 것이라 순서가 중요하다.
