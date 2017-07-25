---
layout: post
title: "Angular2에서 Router 모듈을 통해 다른 컴포넌트로 데이터를 보내는 법"
date: 2017-01-31
categories: frontend
tags: [angular]
comment: yes
---

## Angular2에서 Router 모듈을 통해 다른 컴포넌트로 데이터를 보내는 법

``` typescript
this.router.navigate(['/general-seg-register', this.selectedSite, this.selectedDataSource, { id: this.selectedDataSourceId }]);
```

Router 인스턴스를 통해서 navigate 메서드를 호출 할때 미리 지정한 URL외에 데이터를 함께 보낼 수 있다.


app.routing.module.ts에서 Routes 인스턴스 내

``` typescript
{path: 'big-seg-register/:site-name/:datasource-name', component: BigSegRegisterComponent},
```

해당 컴포넌트의 key값을 미리 URL에 지정해두면 준비 끝.

이후 데이터를 받는 component에서 ActivatedRoute를 DI 시키고

``` typescript
this.activatedRoute.params.subscribe(params => {
  this.siteName = params['site-name'];
});
```

미리 지정한 키값으로 params에 접근하면 받을 수 있다

해당 컴포넌트의 url은 아래와 같은 형태로 표시 된다.

```
general-seg-register/aSiteName/aDataSourceName/
```

—

그 외 querystring을 통해서 보내고자할 때는

navigate 메서드에서 딕셔너리 형태로 포함시켜 보내면 된다.

``` typescript
this.router.navigate(['/general-seg-register', { id: this.selectedDataSourceId }]);
```

이후 받는 컴포넌트에서는

``` typescript
this.activatedRoute.params.subscribe(params => {
  this.siteId = params['id'];
});
```

처럼 받아서 처리할 수 있고 URL은 다음과 같이 표시된다

```
/general-seg-register;id=aSiteId
```

—

첫번째 방식은 URL을 통해서 페이지를 식별하고자하는 경우에 좋을 것 같고
두번째 방식은 그 외 데이터를 넘길 때 사용할 때 좋을 것 같다.

—

** URL 상에서 데이터를 노출하길 원하지 않는 경우?

=> global한 Service 클래스를 만들어서 인스턴스를 설정하고 데이터를 받는 쪽은 subscribe를 통해서 값을 바꿀 수 있도록 처리하는 방법을 이용한다.


---
Referencing

: http://stackoverflow.com/questions/36835123/how-do-i-pass-data-in-angular-2-components-while-using-routing
