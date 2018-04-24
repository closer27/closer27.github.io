---
layout: post
title: "angular2 custom reuse strategy 사용"
date: 2017-02-22
categories: frontend
tags: [angular, angular2, CustomReuseStrategy]
comment: yes
---

앵귤러에서는 라우팅을 통해 페이지가 바뀔 때 컴포넌트가 폐기되고 새로 초기화가 되면서 새로 로딩이 되게 된다. 이 경우 뒤로가기나 앞으로가기를 했을 때 기존의 브라우저 상태가 저장이 안되는데 이를 지원하게 하는 것이 reuse strategy인 것.

이를 구현하려면 몇가지 절차가 필요하다.

### 1. RouteReuseStrategy를 구현하는 custom resuse strategy 클래스를 정의한다.

```
import {RouteReuseStrategy, ActivatedRouteSnapshot, DetachedRouteHandle} from "@angular/router";

export class CustomReuseStrategy implements RouteReuseStrategy {

    handlers: {[key: string]: DetachedRouteHandle} = {};

    shouldDetach(route: ActivatedRouteSnapshot): boolean {
        console.debug('CustomReuseStrategy:shouldDetach', route);
        return true;
    }

    store(route: ActivatedRouteSnapshot, handle: DetachedRouteHandle): void {
        console.debug('CustomReuseStrategy:store', route, handle);
        this.handlers[route.routeConfig.path] = handle;
    }

    shouldAttach(route: ActivatedRouteSnapshot): boolean {
        console.debug('CustomReuseStrategy:shouldAttach', route);
        return !!route.routeConfig && !!this.handlers[route.routeConfig.path];
    }

    retrieve(route: ActivatedRouteSnapshot): DetachedRouteHandle {
        console.debug('CustomReuseStrategy:retrieve', route);
        if (!route.routeConfig) return null;
        return this.handlers[route.routeConfig.path];
    }

    shouldReuseRoute(future: ActivatedRouteSnapshot, curr: ActivatedRouteSnapshot): boolean {
        console.debug('CustomReuseStrategy:shouldReuseRoute', future, curr);
        return future.routeConfig === curr.routeConfig;
    }
}
```

### 2. reuse strategy를 적용하고자하는 모듈의 provide에 선언해준다.

```
…
import { RouteReuseStrategy } from '@angular/router';
…

@NgModule({
  imports: [ … ],
  providers: [
    {provide: RouteReuseStrategy, useClass: CustomReuseStrategy}],
  declarations: [ … ],
  exports: [ … ]
})

export class SegListModule {
}
```

두가지만 해주면 끝!
그러나 아직 완벽한 지원이 아닌 상태인건지 몇가지 문제가 있다.


1. Cannot reattach ActivatedRouteSnapshot created from a different route
2. lazy loading과 호환이 안되는 문제
	- 입력된 값이 저장이 안된다. 즉 정상적으로 동작을 안한다는 것.

—
Referencing

: SOFTWAREarchitekt.at. (2017). "Sticky Routes In Angular 2.3+ With RouteReuseStrategy", [online] Available at: [https://www.softwarearchitekt.at/post/2016/12/02/sticky-routes-in-angular-2-3-with-routereusestrategy.aspx](https://www.softwarearchitekt.at/post/2016/12/02/sticky-routes-in-angular-2-3-with-routereusestrategy.aspx)
: StackOverflow. (2017). "Error: Cannot reattach ActivatedRouteSnapshot created from a different route", [online] Available at:
[http://stackoverflow.com/questions/41584664/error-cannot-reattach-activatedroutesnapshot-created-from-a-different-route](http://stackoverflow.com/questions/41584664/error-cannot-reattach-activatedroutesnapshot-created-from-a-different-route)
[Accessed February 22, 2017].
: example 코드: [https://plnkr.co/edit/okc6GXlvaTXwNTUQEm3i?p=preview](https://plnkr.co/edit/okc6GXlvaTXwNTUQEm3i?p=preview)
