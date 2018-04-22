---
layout: post
title: "angular2 - 브라우저 뒤로가기 시 url 변경"
date: 2018-04-15
categories: frontend
tags: [angular, angular2, PopStateEvent, router]
comment: yes
---

angular로 작성된 웹에서 브라우저의 뒤로가기 버튼을 눌러 이전 화면으로 돌아왔을 때 해당 컴포넌트에서 특정 로직을 동작시키고 싶었다.
리서치를 좀 해본 결과 PopStateEvent를 받는 이벤트리스너를 추가하여 강제로 라우팅을 시키는 방법을 사용해 파라미터를 추가하여 뒤로가기 버튼을 통해 돌아온 컴포넌트에서 분기처리하면 될 것 같았다.

```typescript
// 현재 컴포넌트
ngOnInit() {
  this.locationSubscription = <Subscription> this.location.subscribe((value: PopStateEvent) => {
    this.router.navigate(['list', {backButtonPressed: true}]);
  });
}

ngOnDestroy() {
  this.locationSubscription.unsubscribe();  // 구독 취소
}
```

```typescript
// 이전 컴포넌트
constructor(private activatedRoute: ActivatedRoute) {
  this.activatedRoute.params.subscribe((params) => {
    if (params['backButtonPressed']) {
      // 백버튼을 통해서 들어온 경우
    }
  }
}
```

`locationSubscription`를 멤버변수로 선언한 이유는 컴포넌트가 폐기될 때 구독을 취소하기 위함이다.
그러나 실제로 실행해보면 강제로 라우팅처리한 로직이 먼지 실행된 다음, 원래 돌아가야하는 경로로 다시 라우팅이 되어버리는 결과가 나타난다.

`setTimeout` 트릭으로 이 문제를 해결할 수 있다. 그리고 `PopStateEvent`의 `value`에는 이동하려는 url에 접근할 수 있어 특정 url인 경우에만 처리할 수 있게 분기처리할 수 있다.

이를 해결한 로직은 다음과 같다.

```typescript
this.locationSubscription = <Subscription> this.location.subscribe((value: PopStateEvent) => {
  if (value['url'] === '/list') {
    setTimeout(() => {
      this.router.navigate(['list', {backButtonPressed: true}]);
    });
  }
});
```

--
Referencing
: [https://stackoverflow.com/questions/38891002/angular-2-replace-history-instead-of-pushing?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa](https://stackoverflow.com/questions/38891002/angular-2-replace-history-instead-of-pushing?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
