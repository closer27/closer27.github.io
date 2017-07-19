---
layout: post
title: "returned expression type Map<any> is not assignable to type Observable<boolean> 에러가 날 때"
date: 2017-07-19
categories: frontend
tags:
- angular
- rxjs
- map
comment: yes
---

returned expression type Map<any> is not assignable to type Observable<boolean> 에러가 날 때

```
  public getCurrentUser(): Observable<User> {
    return HttpHelper.get(this.http, USER_INFO_API)
      .map((response) => {
        if (response['code'] !== StatusCode.OK) {
          return null;
        }
        return response['result'] as User;
      });
  }
```

Observable<User>를 반환하는 http 통신 메서드를 만들고 결과값을 map 메서드를 통해 null 또는, 인스턴스로서 매핑해서 반환하도록 변경했는데

`returned expression type Map<any> is not assignable to type Observable<User> blahblah` 라는 에러가 자꾸 뜨는 것이었다.

Observable로 감싸주지 않아서 그런가싶어서 Observable.of()로 감쌌더니 또 이번엔 Observable<Observable<User>> 형식 어쩌구하는거보니 그건 아닌 모양이다.

계속 검색해본 결과로는 rxjs 버전이 오르면서 5.1.0 버전부터였는지는 잘 모르겠지만

> Observable.map takes only one callback function as parameter. You should put the error callback in subscribe() or use catch

즉 Observable.map은 파라미터로 하나의 콜백함수만 취하기 때문에 error callback을 subscribe나 catch를 통해서 넣어줘야한다는 것이었다.

그래서

```
    return HttpHelper.get(this.http, USER_INFO_API)
      .map((response) => {
        if (response['code'] !== StatusCode.OK) {
          return null;
        }
        return response['result'] as User;
      })
      .catch((error: any) => {
        return Observable.of(null);
      });
```

.catch()를 써서 로직을 추가해줬더니 해결됐다.

-
Referencing

: StackOverflow. (2017). “Observable.map - Supplied parameter does not match any signature of call target”, [online] Available at: [https://stackoverflow.com/questions/42759006/observable-map-supplied-parameter-does-not-match-any-signature-of-call-target](https://stackoverflow.com/questions/42759006/observable-map-supplied-parameter-does-not-match-any-signature-of-call-target) [Accessed July 19, 2017]. 
