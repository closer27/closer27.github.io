---
layout: post
title: "angular AOT 컴파일시 몇가지 문제상황과 해결 방법"
date: 2017-06-02
categories: frontend
tags: [angular, AOT, JIT, troubleshooting, webpack]
comment: yes
---

# Trying to compile the angular project with AOT and trouble shooting
---

진행 중인 프로젝트를 AOT 방식으로 컴파일하여 배포하려고 시도해왔지만 번번히 여러 이유로 실패했는데
이 곳에 오는 누군가에게 도움이 되길바라며 trouble shooting 이슈들을 간단하게 정리했다.

당시 나의 개발 환경

- angular 4.2.0-beta.1
- angular-cli를 쓰지 않는 webpack 2.2
- @ngtools/webpack 1.4.0

## 1. ngc의 sass 미지원

[공식 홈페이지](https://angular.io/guide/aot-compiler)에서 소개하는 ngc를 통한 AOT 컴파일은 sass 파일이 제대로 지원이 안되면서 자연스럽게
webpack의 AotPlugin을 통해서 시도하게 되었다.

Referencing
: [https://github.com/angular/angular/issues/11815](https://github.com/angular/angular/issues/11815)

## 2. 라이브러리 업데이트

ngtools/webpack
awesome-typescript-loader
업데이트

```
/Users/closer27/Devs/ds_dev_team/dmp-dashboard/client-config/webpack.aot.js:74
            new ForkCheckerPlugin(),
            ^
TypeError: ForkCheckerPlugin is not a constructor
...
```
기존 webpack.aot.json을 통해서 npm run build:aot 실행해보니, 위와 같은 에러가 나온다
찾아보니 awesome-typescript-loader가 3.0.0 버전 이후로 ForkCheckerPlugin이 없어졌다고 함.

라이브러리를 업데이트하고 나서 다시 시도하니 에러가 사라졌다.

Referencing
: [https://github.com/s-panferov/awesome-typescript-loader/issues/274](https://github.com/s-panferov/awesome-typescript-loader/issues/274)

## 3. SourceFileConstructor 에러

```
> dmp-dashboard@2.0.0 build:aot /Users/closer27/Devs/ds_dev_team/dmp-dashboard
> webpack --config client-config/webpack.aot.js

/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/typescript/lib/typescript.js:14876
            var sourceFile = new SourceFileConstructor(263 /* SourceFile */, /*pos*/ 0, /* end */ sourceText.length);

TypeError: Cannot read property 'length' of undefined
    at createSourceFile (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/typescript/lib/typescript.js:14876:109)
    at parseSourceFileWorker (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/typescript/lib/typescript.js:14808:26)
    at Object.parseSourceFile (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/typescript/lib/typescript.js:14757:26)
    at Object.createSourceFile (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/typescript/lib/typescript.js:14612:29)
    at WebpackCompilerHost.getSourceFile (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/@ngtools/webpack/src/compiler_host.js:210:27)
    at findSourceFile (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/typescript/lib/typescript.js:64751:29)
    at processSourceFile (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/typescript/lib/typescript.js:64682:27)
    at processRootFile (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/typescript/lib/typescript.js:64570:13)
    at /Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/typescript/lib/typescript.js:63919:60
    at Object.forEach (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/typescript/lib/typescript.js:1398:30)
```
에러가 나오는데 이건 source 위치가 잘못돼서 뜨는거라고 함
tsconfig영역의 경로가 잘못된건 아닌지 확인한다.

Referencing
: [https://stackoverflow.com/questions/41158198/typescript-errorcannot-read-property-length-of-undefined-while-running-ng-ser](https://stackoverflow.com/questions/41158198/typescript-errorcannot-read-property-length-of-undefined-while-running-ng-ser)


## 4. @ngtools/webpack 통해서 aot 빌드

링크의 댓글을 보면 ngtools/webpack 통해서 빌드하면 ngfactory 클래스는 파일로 생성되는 것이 아니라 메모리에 생성된다고 함
따라서 main.ts 파일의 내용에 ngfactory 클래스를 인자로 넘기는 메서드를 써서 따로 변경할 필요 없이 JIT 컴파일 방식과 똑같이 사용해도 된다.

Referencing
:  [https://stackoverflow.com/questions/39111198/webpack-typescript-and-angular2-with-ahead-of-time-aot-compilation](https://stackoverflow.com/questions/39111198/webpack-typescript-and-angular2-with-ahead-of-time-aot-compilation)


## 5. forEach 에러

```
ERROR in Could not resolve "src/public/main-web/app/app.module" from "src/public/main-web/app/app.module".

ERROR in ./src/public/main-web/app/shared/error-page/error-page.component.ts
Module build failed: TypeError: Cannot read property 'forEach' of undefined
    at Promise.resolve.then.then.then (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/@ngtools/webpack/src/loader.js:367:40)
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:169:7)
 @ ./aot/src/public/main-web/app/app.module.ngfactory.ts 139:0-107
 @ ./src/public/main-web/main.browser.aot.ts

ERROR in ./src/public/main-web/app/shared/extra-page/go-to-site-selection.component.ts
Module build failed: TypeError: Cannot read property 'forEach' of undefined
    at Promise.resolve.then.then.then (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/@ngtools/webpack/src/loader.js:367:40)
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:169:7)
 @ ./aot/src/public/main-web/app/datasource/datasource-list/datasource-list.component.ngfactory.ts 13:0-121
 @ ./aot/src/public/main-web/app/app.module.ngfactory.ts
 @ ./src/public/main-web/main.browser.aot.ts
```

컴포넌트에서 쓰지않는 빈 styleUrls 항목은 지워야한다

Referencing
: [https://github.com/angular/angular-cli/issues/5639](https://github.com/angular/angular-cli/issues/5639)


## 6. 경로 에러

```
ERROR in Could not resolve "src/public/main-web/app/app.module" from "src/public/main-web/app/app.module".
```

계속 ngfactory를 못찾는다고 떠서 tsconfig와 webpack에서의 entry 경로를 변경 함.
결국 빌드 성공한 설정은 다음과 같다

tsconfig.aot.json
```
{
  "compilerOptions": {
    "target": "es5",
    "module": "es2015",
    "moduleResolution": "node",
    "sourceMap": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "lib": ["es2015", "dom"],
    "noImplicitAny": false,
    "suppressImplicitAnyIndexErrors": true
  },

  "files": [
    "src/public/main-web/app/app.module.ts",
    "src/public/main-web/main.browser.aot.ts"
  ],

  "angularCompilerOptions": {
    "genDir": ".",
    "entryModule": "src/public/main-web/app/app.module#AppModule"
  }
}
```

webpack.aot.json
```
const webpack = require('webpack');
const AotPlugin = require('@ngtools/webpack').AotPlugin;
var path = require('path');

module.exports = {
  entry: {
    polyfills: './src/public/main-web/polyfills.browser.ts',
    main: './src/public/main-web/main.browser.aot.ts'
  },
  output: {
    path: './dist',
    filename: '[name].bundle.js',
  },
  resolve: {
    extensions: ['.ts', '.js', '.html'],
  },
  plugins: [
    new AotPlugin({
      tsConfigPath: './tsconfig.aot.json',
    }),
    //new CopyWebpackPlugin([
    //	{from: './index.html'}
    //]),
    new webpack.LoaderOptionsPlugin({
      minimize: true,
      debug: false
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      output: {
        comments: false
      },
      sourceMap: true
    })
  ],
  module: {
    loaders: [
      {test: /\.scss$/, loaders: ['raw-loader', 'sass-loader']},
      {test: /\.css$/, loader: 'raw-loader'},
      {test: /\.html$/, loader: 'raw-loader'},
      {test: /\.ts$/, loader: '@ngtools/webpack'}
    ]
  },
  devServer: {
    historyApiFallback: true
  }
};
```

main.aot.ts
```
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppModule } from './app/app.module';

platformBrowserDynamic().bootstrapModule(AppModule);
```


## 7. 빌드는 되는데 watch가 안되네

```
/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/webpack-dev-server/bin/webpack-dev-server.js:386
		throw e;
		^

Error: `output.path` needs to be an absolute path or `/`.
    at Object.setFs (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/webpack-dev-middleware/lib/Shared.js:88:11)
    at Shared (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/webpack-dev-middleware/lib/Shared.js:214:8)
    at module.exports (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/webpack-dev-middleware/middleware.js:22:15)
    at new Server (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/webpack-dev-server/lib/Server.js:56:20)
    at startDevServer (/Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/webpack-dev-server/bin/webpack-dev-server.js:379:12)
    at /Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/webpack-dev-server/bin/webpack-dev-server.js:325:3
    at /Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/webpack-dev-server/node_modules/portfinder/lib/portfinder.js:160:14
    at /Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/webpack-dev-server/node_modules/async/lib/async.js:52:16
    at /Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/webpack-dev-server/node_modules/async/lib/async.js:269:32
    at /Users/closer27/Devs/ds_dev_team/dmp-dashboard/node_modules/webpack-dev-server/node_modules/async/lib/async.js:44:16
```

=> webpack의 output 경로를 절대경로로 지정해줌; 해결

```
ERROR in ./src/public/main-web/app/app.module.ngfactory.ts
Module build failed: Error: ENOENT: no such file or directory, open '/Users/closer27/Devs/ds_dev_team/dmp-dashboard/src/public/main-web/app/app.module.ngfactory.ts'
 @ ./src/public/main-web/main.browser.aot.ts 2:0-64
 @ multi (webpack)-dev-server/client?http://localhost:8080 ./src/public/main-web/main.browser.aot.ts
```
