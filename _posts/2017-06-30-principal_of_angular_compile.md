---
layout: post
title: "Angular의 동작원리 - 컴파일"
date: 2017-06-30
categories: frontend
tags:
- angular
- compile
- aot
- jit
comment: yes
---

Angular의 동작원리 - 컴파일

앵귤러로 작성한 코드는 결국 브라우저에서 실행할 수 있는 자바스크립트이다.

## 부트스트랩과 컴파일

enabledProdMode() - 개발 환경에서만 실행하는 불필요한 로직 동작 방지

```
platformBrowserDynamic().bootstrapModule(AppModule);
```

앵귤러 애플리케이션 실행될 플랫폼을 브라우저로 지정하는 목적과 함께 애플리케이션 부트스트랩을 위한 PlatformRef 타입 객체 반환; 브라우저에서 하나의 웹페이지상에서 생성되는 객체로서
PlatformRef의 역할은 실행될 플랫폼에 맞게 앵귤러 모듈을 부트스트랩하는 것.

bootstrapModule 메서드가 바로 그것, 여기에 인자는 앵귤러 애플리케이션의 루트 모듈 (appModule)이다. 이후 모듈은 순차적으로 부트스트랩 된다.

## JIT 컴파일
부트스트랩의 첫 과정은 컴파일; 타입스크립트 컴파일이 아님, 타입스크립트는 앵귤러 CLI 빌드하는 과정에서 이미 자바스크립트로 변환 됨
: 컴포넌트의 템플릿을 자바스크립트 코드로 변환하는 것
: JIT 컴파일 결과물은 브라우저에서 평가되며 메모리상에서만 존재

개발자도구 > source탭을 통해서 들어가보면 ng://로 시작하는 목록아래 컴파일된 결과물을 볼 수 있다.

앵귤러 컴파일러가 vm 친화적인 자바스크립트 코드로 변환

### JIT 컴파일의 문제

1. 최초 애플리케이션 실행 속도가 느리다
2. 번들링된 결과물의 크기

앵귤러는 브라우저에서 컴파일하기 위해 번들링된 소스에 최초 컴파일에만 필요한 소스를 모두 포함한다; @angular/compiler => 최초 JIT 컴파일 시 제외하면 앱 로직과는 관련이 없음

## AOT 컴파일

JIT 컴파일 단점 극복 방법 두 가지(AOT, 서버 컴파일링:Angular Universal); 이 중 AOT는 번들링하기 전에 템플릿을 컴파일하는 방법.


AOT 컴파일은 애플리케이션이 브라우저에서 실행되기 전에 앵귤러의 소스를 자바스크립트로 변환하기 때문에 플랫폼 의존적인 동적코드를 사용하면 안된다.
: 현재 페이지 URL을 window.location.herf나 document.URL로 가져온 뒤 이를 기반으로하는 로직이 있다면 사용할 수 없음.

```typescript
platformBroser().bootstrapModuleFactory(AppModuleNgFactory);
```

AOT 컴파일 환경에서는 컴파일 된 모듈패키지 객체를 인자로 받아서 부트스트랩한다.

> @ngtools/webpack을 통한 AOT 컴파일은 메모리상에 ngfactory 파일들이 생성되어 알아서 동작하므로 main 코드의 변환이 필요 없음.


—
Referencing

: 조우진 (2017), Angular 첫 걸음, 한빛미디어, 서울
