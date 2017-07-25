---
layout: post
title: "Angular2 프로젝트 구성"
date: 2017-01-31
categories: frontend
tags: [angular]
comment: yes
---

## Angular2 프로젝트 구성

- package.json: npm 명령어를 통해 외부 모듈에 대한 의존성을 관리
- tsconfig.json: 타입스크립트를 자바스크립트로 변환할때 필요한 설정
- typings.json: 타입스크립트가 외부 라이브러리(node, jasmine, core-js)를 인식할 수 있게 해줌
- systems.config.js: 모듈 로더에 전달하는 System.JS 설정 정보

## index.html

```
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Weboojadle</title>
  <base href="/">

  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>
  <app-root>Loading...</app-root>
</body>
</html>
```

index.html에서 임포트하는 css는 앱의 전반적인 스타일을 정의할 수 있다.
다음은 라이브러리(폴리필; 구버전의 브라우저가 최신 스크립트 기능에 대응하도록 보완하는 라이브러리) 임포트
- core-js: ES6 지원을 위한 폴리필
- zone.js: 변화 감지를 위한 라이브러리
- reflect-metadata: ES7 장식자를 추가하기 위한 폴리필

## app.module.ts
애플리케이션 차원의 모듈 파일. 전역으로 사용하는 모듈이나 최상위 컴포넌트를 등록.
bootstrap 속성에는 앱을 실행할 때 가장 먼저 호출하게 할 애플리케이션 컴포넌트를 등록.

## main.ts
여기서는 app.modules.ts 파일을 입포트해서 platform-browser-dynamic을 등록한다.
platformBrowserDynamic은 앵귤러의 모든 API에 대한 진입점이다.
